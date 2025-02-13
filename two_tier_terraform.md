# **Two-Tier Architecture on Azure Using Terraform**

## **Overview**
This Terraform setup provisions a **two-tier architecture** on **Azure**, including:
- **App Tier:** An Ubuntu-based Virtual Machine (VM) using an image.
- **Database Tier:** A separate VM running MongoDB.
- **Networking:** A Virtual Network (VNet) with **Public and Private Subnets**.
- **Security:** Network Security Groups (NSGs) with appropriate firewall rules.

---

## **Infrastructure Overview**
| Component                 | Description |
|---------------------------|-------------|
| **Resource Group**        | `tech501` (Azure container for resources) |
| **Virtual Network (VNet)** | `tech501-maram-2-subnet-vnet-2` (Network for the environment) |
| **Subnets**               | Public: `10.0.2.0/24`, Private: `10.0.3.0/24` |
| **App VM**                | `tech501-maram-new-app-vm` (Node.js App) |
| **DB VM**                 | `tech501-maram-new-db-vm` (MongoDB) |
| **Public IP**             | Assigned to App VM for external access |
| **NSGs (Firewall Rules)** | App: Allow `22`, `80`, `3000`; DB: Allow `22`, `27017` from Public Subnet |

---

## **Terraform Configuration**

### **1️- Provider Configuration (`main.tf`)**
```hcl
provider "azurerm" {
  features {}
  subscription_id = "<YOUR_SUBSCRIPTION_ID>"
}
```

### **2️- Networking (`main.tf`)**
```hcl
resource "azurerm_virtual_network" "tech501_vnet" {
  name                = "tech501-maram-2-subnet-vnet-2"
  location            = "UK South"
  resource_group_name = "tech501"
  address_space       = ["10.0.0.0/16"]
}

resource "azurerm_subnet" "public_subnet" {
  name                 = "public-subnet"
  resource_group_name  = azurerm_virtual_network.tech501_vnet.resource_group_name
  virtual_network_name = azurerm_virtual_network.tech501_vnet.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_subnet" "private_subnet" {
  name                 = "private-subnet"
  resource_group_name  = azurerm_virtual_network.tech501_vnet.resource_group_name
  virtual_network_name = azurerm_virtual_network.tech501_vnet.name
  address_prefixes     = ["10.0.3.0/24"]
}
```

### **3️- Security Groups (`security_groups.tf`)**
```hcl
# App VM Network Security Group
resource "azurerm_network_security_group" "app_nsg" {
  name                = "app-vm-nsg"
  location            = "UK South"
  resource_group_name = "tech501"

  security_rule {
    name                       = "Allow-SSH"
    priority                   = 1001
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  security_rule {
    name                       = "Allow-HTTP"
    priority                   = 1002
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "80"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  security_rule {
    name                       = "Allow-NodeJS"
    priority                   = 1003
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "3000"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}

# DB VM Network Security Group
resource "azurerm_network_security_group" "db_nsg" {
  name                = "db-vm-nsg"
  location            = "UK South"
  resource_group_name = "tech501"

  security_rule {
    name                       = "Allow-SSH"
    priority                   = 1001
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  security_rule {
    name                       = "Allow-MongoDB"
    priority                   = 1002
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "27017"
    source_address_prefix      = "10.0.2.0/24" # Only allow MongoDB from public subnet
    destination_address_prefix = "*"
  }

  security_rule {
    name                       = "Deny-All"
    priority                   = 1003
    direction                  = "Inbound"
    access                     = "Deny"
    protocol                   = "*"
    source_port_range          = "*"
    destination_port_range     = "*"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}

# Attach App NSG to Public Subnet
resource "azurerm_subnet_network_security_group_association" "app_nsg_assoc" {
  subnet_id                 = azurerm_subnet.public_subnet.id
  network_security_group_id = azurerm_network_security_group.app_nsg.id
}

# Attach DB NSG to Private Subnet
resource "azurerm_subnet_network_security_group_association" "db_nsg_assoc" {
  subnet_id                 = azurerm_subnet.private_subnet.id
  network_security_group_id = azurerm_network_security_group.db_nsg.id
}

```

### **4️- App VM (`app_vm.tf`)**
```hcl
resource "azurerm_linux_virtual_machine" "app_vm" {
  name                  = "tech501-maram-new-app-vm"
  resource_group_name   = "tech501"
  location              = "UK South"
  size                  = "Standard_B1s"
  admin_username        = "adminuser"
  network_interface_ids = [azurerm_network_interface.app_vm_nic.id]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "StandardSSD_LRS"
  }

  source_image_id = "/subscriptions/<SUBSCRIPTION_ID>/resourceGroups/TECH501/providers/Microsoft.Compute/images/tech501-maram-sparta-app-ready-to-run-img"

    # Ensure the DB VM is created first
  depends_on = [azurerm_linux_virtual_machine.db_vm]

  custom_data = base64encode(<<EOF
#!/bin/bash
cd /repo/app
export DB_HOST=mongodb://10.0.3.4:27017/posts
pm2 start app.js
EOF
  )
}
```

### **5️- Database VM (`db_vm.tf`)**
```hcl
resource "azurerm_linux_virtual_machine" "db_vm" {
  name                  = "tech501-maram-new-db-vm"
  resource_group_name   = "tech501"
  location              = "UK South"
  size                  = "Standard_B1s"
  admin_username        = "adminuser"
  network_interface_ids = [azurerm_network_interface.db_vm_nic.id]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "StandardSSD_LRS"
  }

  source_image_id = "/subscriptions/<SUBSCRIPTION_ID>/resourceGroups/TECH501/providers/Microsoft.Compute/images/tech501-maram-sparta-db-ready-to-run-img"
}
```

---

## ** Deployment Steps**
1️- **Initialize Terraform:**
```sh
terraform init
```

2️- **Plan the Infrastructure:**
```sh
terraform plan
```

3️- **Apply the Configuration:**
```sh
terraform apply
```
(Type `yes` when prompted)


4- **Check if the App is Running:**
```sh
ssh -i ~/.ssh/tech501-maram-key-az adminuser@APP_VM_PUBLIC_IP
pm2 list
```

5- **Visit the App in Browser:
- The app webpage was running successfuly.

![](<app page.png>)
---
6- **the posts**:
![](<posts page.png>)

