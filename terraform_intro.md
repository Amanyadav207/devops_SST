# Introduction to Infrastructure Provisioning & Terraform

**Date:** January 9, 2025

## 1. What is IaC?
**Infrastructure as Code (IaC)** is managing infrastructure via code rather than manual processes.
*   **Benefits**: Automated, Version Controlled, Consistent, Repeatable.

## 2. What is Terraform?
Open-source, declarative IaC tool by HashiCorp.
*   **Key Features**: Multi-cloud (AWS, Azure, GCP), State Management, Plan/Apply workflow.

## 3. Workflow: The 5 Steps
`Write` → `Init` → `Plan` → `Apply` → `Destroy`

## 4. Components (HCL)
**HCL** (HashiCorp Configuration Language) is the syntax used.

| Component | Purpose | Example |
| :--- | :--- | :--- |
| **Provider** | Cloud plugin | `provider "aws" { region = "us-east-1" }` |
| **Resource** | Infra object | `resource "aws_s3_bucket" "b" { ... }` |
| **Variable** | Input param | `variable "instance_type" { default = "t2.micro" }` |
| **Output** | Return value | `output "ip" { value = aws_instance.web.public_ip }` |

## 5. Essential Commands
```bash
terraform init      # Download providers
terraform plan      # Preview changes
terraform apply     # Create/Update infrastructure
terraform destroy   # Delete infrastructure
terraform fmt       # Format code
terraform validate  # Check syntax
```

## 6. State Management
Terraform tracks the *real world* resources in a file called **`terraform.tfstate`**.
*   **Remote State**: Store state in S3/GCS for team collaboration (locking, consistency).
*   **Commands**: `terraform state list`, `terraform state show`.

## 7. Example: AWS Server
```hcl
provider "aws" { region = "us-east-1" }

resource "aws_instance" "web" {
  ami           = "ami-0c55b159..."
  instance_type = "t2.micro"
  tags = { Name = "WebServer" }
}
```
