# Terraform in Action: Advanced Concepts

**Date:** January 12, 2025

## 1. Variables
Parameterize configurations to make them dynamic.
*   **Define**: In `variables.tf`.
*   **Use**: `var.variable_name`.
*   **Pass**: CLI (`-var="region=us-west-2"`) or File (`-var-file="prod.tfvars"`).

```hcl
variable "instance_count" {
  type    = number
  default = 2
}
```

## 2. Modules
Reusable containers for multiple resources.
*   **Structure**: `modules/vpc/` (`main.tf`, `variables.tf`, `outputs.tf`).
*   **Usage**:
    ```hcl
    module "vpc" {
      source = "./modules/vpc"
      cidr   = "10.0.0.0/16"
    }
    ```

## 3. State Management & Remote Backends
**State file** (`terraform.tfstate`) maps real world resources to your configuration.
*   **Best Practice**: Store remotely (S3, GCS, Azure Blob) for collaboration.
*   **Locking**: Use DynamoDB (AWS) to prevent concurrent writes.
```hcl
terraform {
  backend "s3" {
    bucket = "my-tf-state"
    key    = "prod/terraform.tfstate"
    dynamodb_table = "terraform-locks"
  }
}
```

## 4. Workspaces
Manage multiple environments (dev, prod) with the **same** configuration.
*   **Commands**: `terraform workspace new/select/list`.
*   **Usage**: `instance_type = terraform.workspace == "prod" ? "t2.large" : "t2.micro"`

## 5. Data Sources
Query **existing** infrastructure (outside of Terraform's control).
```hcl
data "aws_ami" "ubuntu" { most_recent = true ... }
resource "example" "web" { ami = data.aws_ami.ubuntu.id }
```

## 6. Provisioners
Execute scripts on a resource after creation (e.g., `remote-exec`, `local-exec`).
*   *Note: Use mainly as a last resort; prefer configuration management tools (Ansible) or cloud-init.*

## 7. Best Practices
1.  **Version Control**: Git everything (except `.terraform` and secret vars).
2.  **Remote State**: Mandatory for teams.
3.  **Modules**: Don't repeat yourself (DRY).
4.  **Plan First**: Always review `terraform plan` output.
5.  **State Locking**: Prevent corruption.

## 8. Terraform in CI/CD (GitHub Actions)
Automate infrastructure changes.
*   **Pull Request**: Run `terraform plan`.
*   **Merge to Main**: Run `terraform apply`.
```yaml
- name: Terraform Apply
  if: github.ref == 'refs/heads/main'
  run: terraform apply -auto-approve
```
