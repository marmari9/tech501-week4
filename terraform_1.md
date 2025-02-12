#  Ansible vs Terraform

##  Overview  
| Tool       | Purpose                                  | Type          |
|------------|----------------------------------------|--------------|
| **Ansible**  | Configuration management and automation | Mutable Infrastructure |
| **Terraform** | Infrastructure provisioning and orchestration | Immutable Infrastructure |

---

##  Key Differences  

| Feature             | Ansible    | Terraform   |
|---------------------|------------|-------------|
| **Primary Use**     | Configures and manages existing servers | Provisions and manages infrastructure |
| **Infrastructure Type** | Mutable (changes in place) | Immutable (rebuilds resources) |
| **State Management** | Does not maintain state | Uses a state file to track resources |
| **Execution Method** | Agentless, uses SSH or WinRM | Uses APIs to manage cloud resources |
| **Idempotency** | Ensures repeated runs yield the same result | Uses declarative approach to enforce the desired state |
| **Language** | YAML (Playbooks) | HCL (HashiCorp Configuration Language) |
| **Cloud Integration** | Works with AWS, Azure, GCP, on-prem | Primarily used for cloud provisioning |

---

##  When to Use What?  
**Use Ansible if:**  
- You need to automate server configurations, application deployments, and updates.  
- Your infrastructure already exists, and you want to manage it dynamically.  

**Use Terraform if:**  
- You need to provision, modify, and manage cloud infrastructure.  
- You want an immutable and version-controlled approach to infrastructure management.  

---

- Can They Work Together?   
Yes! Terraform can **provision infrastructure**, and Ansible can **configure** it.  
For example:
1. Use Terraform to create AWS EC2 instances.  
2. Use Ansible to configure software on those instances.  

## Terraform Configuration Files

Terraform uses several configuration files to manage infrastructure as code:

### main.tf
This file is the core of Terraform configuration. It defines the resources want to create, such as EC2 instances, and specifies their properties.

### variable.tf
This file is used to define input variables, allowing to customize aspects of your infrastructure without changing the main configuration.

### output.tf
This file specifies the outputs of Terraform configuration, such as IP addresses or URLs, making it easy to find key information.

* Importance
Using Terraform ensures a uniform, standardized production environment for EC2 instances, enhancing consistency and reducing errors.

- A .gitignore file specifies intentionally untracked files that Git should ignore. For a Terraform project, it might include:

- .terraform/: Directory containing Terraform plugins and modules.
- .tfstate: Terraform state files, which contain the current state of the infrastructure.
- .tfstate.backup: Backup files of the Terraform state.
- .tfvars: Files containing sensitive variable values.


### Steps to deploy an **AWS EC2 instance** using Terraform with a dedicated security group.

---

1- ** Terraform Configuration**

- **`main.tf` - EC2 Instance Configuration**
```bash
provider "aws" {
  region = "eu-west-1"
}

resource "aws_instance" "app_instance" {
  # AMI ID for Ubuntu (Update if needed)
  ami = "ami-0c1c30571d2dae5c9"

  # Instance type
  instance_type = "t3.micro"

  # Attach the correct key pair
  key_name = "tech501-maram-key-2"

  # Assign a public IP
  associate_public_ip_address = true

  # Attach the security group
  vpc_security_group_ids = [aws_security_group.tech501_maram_sg.id]

  # Tags
  tags = {
    Name = "tech501-maram-terraform-app"
  }
}


```

- security-group.tf - Security Group Configuration

```bash
resource "aws_security_group" "tech501_maram_sg" {
  name        = "tech501-maram-tf-allow-port-22-3000-80"
  description = "Allow SSH, HTTP, and app port 3000"

  # Allow SSH from localhost (your IP)
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["81.159.157.178/32"]  # Replace with your actual public IP
  }

  # Allow port 3000 from anywhere
  ingress {
    from_port   = 3000
    to_port     = 3000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Allow HTTP (port 80) from anywhere
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Allow all outbound traffic
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "tech501-maram-tf-allow-port-22-3000-80"
  }
}

```
2️- **Deployment Steps**:

1- Initialize Terraform: Run the following command to initialize Terraform:
```bash
terraform init
```
- Note: This command sets up Terraform and downloads necessary provider plugins.

2- Plan and Apply the Infrastructure: Check what Terraform will create:
```bash
terraform plan
```

3- Deploy the resources (Destructive): 
```bash
terraform apply 
```
⚠️ Warning: terraform apply makes real changes to AWS.

- Successful Deployment Screenshot: The EC2 instance was created successfully with the required security groups
![alt text](NSG.png)

- Do Not Share These Files on a public repo
    - .terraform/ folder → Contains provider plugins, do not share in public repositories.
    - terraform.tfstate → Stores the actual state of infrastructure, should be kept private.
    - terraform.tfstate.backup → Backup of state, also confidential.

4️- Destroying Infrastructure (Destructive Action): To remove all resources created by Terraform:
```bash
terraform destroy
```
⚠️ Warning: This will permanently delete your EC2 instance and security group.

