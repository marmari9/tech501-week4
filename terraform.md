# Terraform Overview

## What is Terraform? What is it used for?
Terraform is an **Infrastructure as Code (IaC)** tool developed by HashiCorp that allows you to define and provision cloud infrastructure using a **declarative** configuration language called **HCL (HashiCorp Configuration Language)**. It is used to **automate the provisioning and management of infrastructure** across multiple cloud providers, including AWS, Azure, Google Cloud, and on-premises data centers.

## Why Use Terraform? The Benefits
1. **Multi-Cloud Support** – Works across AWS, Azure, Google Cloud, and on-premises.
2. **Declarative Approach** – You define the desired end state, and Terraform figures out the steps to achieve it.
3. **State Management** – Keeps track of infrastructure changes using a **state file**.
4. **Version Control** – Configuration files can be stored in Git for collaboration and tracking.
5. **Idempotency** – Running Terraform multiple times results in the same infrastructure state.
6. **Infrastructure as Code (IaC)** – Enables repeatability and automation.
7. **Dependency Management** – Handles dependencies between resources.
8. **Reusable Modules** – Encourages modular code for reusability.

## Alternatives to Terraform
1. **AWS CloudFormation** – AWS-native IaC tool.
2. **Pulumi** – Uses general-purpose programming languages (Python, Go, etc.) instead of HCL.
3. **Ansible** – More focused on configuration management but can provision infrastructure.
4. **Chef/Puppet** – Primarily configuration management tools.
5. **Google Cloud Deployment Manager** – GCP’s equivalent of CloudFormation.
6. **Azure Resource Manager (ARM) Templates** – Microsoft’s IaC tool.

## Who is Using Terraform in the Industry?
Terraform is widely used by:
- **Tech Giants** – Google, Microsoft, Netflix, LinkedIn, Uber.
- **Financial Services** – HSBC, Capital One.
- **Healthcare & Pharma** – Pfizer, Roche.
- **Retail & E-commerce** – Shopify, Walmart.
- **Startups & Enterprises** – Many organizations use it for cloud automation and infrastructure management.

## In IaC, What is Orchestration? How Does Terraform Act as an "Orchestrator"?
**Orchestration** in IaC refers to the **automated coordination and management** of multiple infrastructure components. Terraform acts as an **orchestrator** by:
- **Defining infrastructure in a structured manner** (HCL).
- **Managing dependencies** between resources (e.g., an EC2 instance needing a security group).
- **Ensuring consistency** across environments.
- **Applying changes efficiently** while minimizing downtime.

---

# Supplying AWS Credentials to Terraform

## Best Practices for Supplying AWS Credentials
Terraform needs AWS credentials to authenticate and deploy resources. The best practice is to use:
1. **IAM Roles (Best for EC2, CI/CD pipelines)** – Attach IAM roles to instances or CI/CD runners instead of using static credentials.
2. **Environment Variables (Good for local use)** – Set `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` in your shell.
3. **AWS Named Profiles** – Use `~/.aws/credentials` with `aws configure`.
4. **AWS SSO (Secure but requires authentication)** – Best for enterprises with central authentication.
5. **Terraform Cloud & Vault (For large-scale deployments)** – Store credentials securely.

## Terraform AWS Credentials Lookup Order (Precedence)
Terraform looks for AWS credentials in the following order:
1. **Environment Variables** – `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`.
2. **Shared Credentials File (`~/.aws/credentials`)** – Profiles set via `aws configure`.
3. **AWS Config File (`~/.aws/config`)** – If using named profiles.
4. **IAM Role (EC2 Instance Profile or ECS Task Role)** – If running inside AWS.
5. **Terraform Cloud or Vault Secrets** – If configured.
6. **Explicitly Defined in Terraform Code** (Not recommended).

## Best Practices for AWS Credentials in Terraform
- **NEVER hardcode credentials** in Terraform files (`.tf` files) or store them in version control.
- **Use IAM roles whenever possible** for security.
- **Use environment variables or named profiles** for local development.
- **Use a secrets manager** (e.g., AWS Secrets Manager, HashiCorp Vault) for sensitive data.
- **Use Terraform Cloud** if working in a team setting.

---

# Terraform for Different Environments (Production, Testing, etc.)

## Why Use Terraform for Multiple Environments?
1. **Consistency** – Ensures identical infrastructure across environments.
2. **Automation** – Reduces manual effort in setting up multiple environments.
3. **Modularity** – Reuse the same Terraform code for different environments with minimal changes.
4. **Isolation** – Prevents conflicts between production and testing environments.
5. **Scalability** – Easily scale up/down infrastructure based on environment needs.
6. **Security** – Enforces least privilege access policies per environment.
7. **Cost Control** – Helps optimize cloud spending by managing separate environments efficiently.

---


