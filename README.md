# 🚀 Terraform Workspaces + Modules (AWS Infrastructure)

## 📌 Overview

This project demonstrates a **production-style Infrastructure as Code (IaC)** setup using **Terraform Workspaces and Modular Architecture** to provision AWS infrastructure across multiple environments (**dev** and **prod**) from a single codebase.

It also includes **remote state management using S3 and DynamoDB**, enabling safe and collaborative deployments.

---

## 🎯 Key Features

- Multi-environment support using Terraform Workspaces  
- Modular Terraform design (EC2, S3, DynamoDB)  
- Dynamic resource provisioning per environment  
- Remote backend using S3 + DynamoDB  
- Scalable and reusable infrastructure  

---

## 🧱 Architecture

| Resource          | dev | prod |
|------------------|-----|------|
| EC2 Instances    | 2   | 3    |
| S3 Buckets       | 1   | 2    |
| DynamoDB Tables  | 1   | 2    |

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
    G --> H[Terraform State]
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

👉 A single configuration map controls infrastructure for each environment.

🧩 Modules
EC2 Module
Creates EC2 instances
Configures security groups
Adds SSH key
S3 Module
Creates S3 buckets
Blocks public access
DynamoDB Module
Creates tables
Uses PAY_PER_REQUEST
🔐 Security
SSH (Port 22)
HTTP (Port 80)
All outbound traffic allowed
🖥️ EC2 Details
AMI: ami-0d76b909de1a0595d
Instance Type: t3.micro
Volume: 10GB (gp3)
🔑 SSH Key

terra-automate-key.pub is the public key used to access EC2 instances.

resource "aws_key_pair" "terra_key" {
  key_name   = "terra-automate-key"
  public_key = file("terra-automate-key.pub")
}

Connect using:

ssh -i terra-automate-key.pem ec2-user@<public-ip>
🛠️ Prerequisites
Terraform >= 1.5.0
AWS CLI configured
AWS account
SSH key pair
☁️ Remote Backend Setup (S3 + DynamoDB)
Create S3 Bucket
aws s3api create-bucket \
  --bucket abdullah-terraform-state-bucket \
  --region us-west-2 \
  --create-bucket-configuration LocationConstraint=us-west-2

Enable versioning:

aws s3api put-bucket-versioning \
  --bucket abdullah-terraform-state-bucket \
  --versioning-configuration Status=Enabled
Create DynamoDB Table
aws dynamodb create-table \
  --table-name terraform-locks \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region us-west-2
Configure Backend

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
Initialize Backend
terraform init
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
🧨 Destroy Infrastructure
terraform destroy

Destroy all:

terraform workspace select dev && terraform destroy -auto-approve
terraform workspace select prod && terraform destroy -auto-approve
🔄 Add New Environment
staging = { instance_count = 2, bucket_count = 1, table_count = 1 }
💡 Best Practices
Modular Terraform design
Workspace-based isolation
Remote state management
Secure configuration
Scalable infrastructure
⚠️ Limitations
No CI/CD integration
No monitoring setup
Basic networking
🚀 Future Improvements
Add CI/CD (Jenkins / GitHub Actions)
Add monitoring (Prometheus + Grafana)
Add VPC module
Add auto-scaling
💼 Resume Description

Designed and implemented multi-environment AWS infrastructure using Terraform Workspaces and modular architecture with remote state management (S3 + DynamoDB).

👨‍💻 Author

Abdullah
DevOps Engineer
