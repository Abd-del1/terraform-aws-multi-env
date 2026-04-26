# 🚀 Terraform Workspaces + Modules (AWS Infrastructure)

## 📌 Overview

This project demonstrates a **production-style Infrastructure as Code (IaC)** implementation using **Terraform Workspaces and Modular Design** to provision AWS infrastructure across multiple environments (**dev** and **prod**) from a **single codebase**.

It also includes **remote state management using S3 and DynamoDB**, enabling safe and collaborative infrastructure deployments.

---

## 🎯 Key Objectives

- Multi-environment infrastructure using Terraform Workspaces  
- Modular and reusable Terraform code  
- Automated AWS resource provisioning  
- Remote backend with state locking  
- Real-world DevOps best practices  

---

## 🧱 Architecture

### 🔹 Resource Distribution

| Resource           | dev | prod |
|------------------|-----|------|
| EC2 Instances     | 2   | 3    |
| S3 Buckets        | 1   | 2    |
| DynamoDB Tables   | 1   | 2    |

---

### 🔹 Naming Convention

All resources are prefixed with workspace:

- `dev-terra-server-1`
- `prod-terra-server-2`

---

## 🏗️ Architecture Diagram

```mermaid
graph TD
    A[Developer] --> B[Terraform CLI]
    B --> C[Workspaces dev/prod]

    C --> D[EC2 Module]
    C --> E[S3 Module]
    C --> F[DynamoDB Module]

    D --> D1[EC2 Instances]
    E --> E1[S3 Buckets]
    F --> F1[DynamoDB Tables]

    B --> G[S3 Backend]
    G --> H[Terraform State File]
    G --> I[DynamoDB Lock Table]
📁 Project Structure
.
├── main.tf
├── variables.tf
├── outputs.tf
├── providers.tf
├── terraform.tf
├── terra-automate-key.pub
└── modules/
    ├── ec2/
    ├── s3/
    └── dynamodb/
⚙️ How It Works
locals {
  env_config = {
    dev  = { instance_count = 2, bucket_count = 1, table_count = 1 }
    prod = { instance_count = 3, bucket_count = 2, table_count = 2 }
  }

  current = lookup(local.env_config, terraform.workspace, local.env_config["dev"])
}

👉 A single configuration map controls all environments.

🧩 Modules
☁️ EC2 Module
Creates EC2 instances
Configures security groups
Adds SSH key
🪣 S3 Module
Creates S3 buckets
Blocks public access
🗄️ DynamoDB Module
Creates tables
Uses PAY_PER_REQUEST
🔐 Security
EC2 Security Group
SSH (22)
HTTP (80)
All outbound allowed
🖥️ EC2 Details
AMI: ami-0d76b909de1a0595d
Instance Type: t3.micro
Volume: 10GB gp3
🛠️ Prerequisites
Terraform >= 1.5.0
AWS CLI configured
AWS account
SSH key pair
⚙️ Remote Backend Setup (S3 + DynamoDB)
Step 1: Create S3 Bucket
aws s3api create-bucket \
  --bucket abdullah-terraform-state-bucket \
  --region us-west-2 \
  --create-bucket-configuration LocationConstraint=us-west-2

Enable versioning:

aws s3api put-bucket-versioning \
  --bucket abdullah-terraform-state-bucket \
  --versioning-configuration Status=Enabled
Step 2: Create DynamoDB Table
aws dynamodb create-table \
  --table-name terraform-locks \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region us-west-2
Step 3: Configure Backend

Add to terraform.tf:

terraform {
  backend "s3" {
    bucket         = "abdullah-terraform-state-bucket"
    key            = "terraform/state.tfstate"
    region         = "us-west-2"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
Step 4: Initialize Backend
terraform init

👉 This migrates local state to S3.

▶️ Usage
Initialize
terraform init
Create Dev Environment
terraform workspace new dev
terraform plan
terraform apply
Create Prod Environment
terraform workspace new prod
terraform plan
terraform apply
Switch Workspace
terraform workspace list
terraform workspace select dev
Outputs
terraform output
🧨 Destroy
terraform destroy

Destroy all:

terraform workspace select dev && terraform destroy -auto-approve
terraform workspace select prod && terraform destroy -auto-approve
🔄 Add New Environment
staging = { instance_count = 2, bucket_count = 1, table_count = 1 }
💡 Best Practices
Modular design
Workspace isolation
Remote state management
Secure configurations
Scalable architecture
⚠️ Limitations
No CI/CD pipeline
No monitoring integration
Basic networking setup
🚀 Future Enhancements
Add CI/CD (GitHub Actions / Jenkins)
Add monitoring (Prometheus + Grafana)
Add VPC module
Add auto-scaling
Add logging (ELK stack)
💼 Resume Description

Designed and implemented multi-environment AWS infrastructure using Terraform Workspaces and modular architecture with remote state management (S3 + DynamoDB), enabling scalable and consistent provisioning.

👨‍💻 Author

Abdullah
DevOps Engineer
