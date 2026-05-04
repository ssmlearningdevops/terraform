# Terraform Interview Q&A

## 1. What is Terraform and why is it used?
Terraform is an open-source Infrastructure as Code (IaC) tool used to automate infrastructure provisioning.

Example: Deploy AWS EC2 and VPC resources with one command.

## 2. What is Infrastructure as Code (IaC)?
IaC means managing infrastructure through code instead of manual setup.

Example: `.tf` files define servers, networks, and databases.

## 3. What is a provider in Terraform?
A provider is a plugin that allows Terraform to interact with cloud platforms.

Example: `provider "aws" { region = "us-east-1" }`

## 4. What is a resource in Terraform?
A resource is the actual infrastructure object managed by Terraform.

Example: `resource "aws_instance" "web" { ami = "ami-123" }`

## 5. What is a Terraform state file?
The `terraform.tfstate` file tracks resources and maps them to your configuration.

## 6. What is `terraform init`?
It initializes the project, downloads providers, and configures the backend.

## 7. What is `terraform plan`?
It shows the execution plan before applying changes.

## 8. What is `terraform apply`?
It creates or updates resources based on the configuration.

## 9. What is `terraform destroy`?
It deletes all resources managed by the current Terraform state.

## 10. What are variables in Terraform?
Variables are inputs used for flexibility and reusability.

Example: `variable "instance_type" { default = "t2.micro" }`

## 11. What are outputs in Terraform?
Outputs are values exported after deployment.

Example: `output "web_ip" { value = aws_instance.web.public_ip }`

## 12. What is a module in Terraform?
A module is a reusable package of Terraform code.

Example: A VPC module reused across environments.

## 13. Difference between `count` and `for_each`?
- `count`: Numeric replication.
- `for_each`: Iterates over a map or set.

Example: `count = 3` vs `for_each = var.subnets`

## 14. What is a remote backend in Terraform?
A remote backend stores state in cloud storage such as S3, GCS, or Azure Storage.

## 15. Why is the state file important?
It ensures Terraform knows what exists. Losing it can cause drift and recreation issues.

## 16. How do you manage secrets in Terraform?
Use Vault, cloud secret managers, or environment variables.

## 17. What is state locking?
State locking prevents concurrent modifications.

Example: DynamoDB locking with an S3 backend.

## 18. How do you troubleshoot Terraform errors?
Check logs, run `terraform plan`, validate configs, and verify provider versions.

## 19. What are common issues in Terraform?
Provider mismatches, state drift, missing variables, and permission errors.

## 20. What are Terraform best practices?
Use modules, remote state with locking, separate environments, and secure secret handling.

## 21. How do you structure Terraform code for large projects?
Use modules, split files (`main.tf`, `variables.tf`, `outputs.tf`), and maintain environment folders.

## 22. What is the use of modules, and how do you create reusable modules?
Modules organize and reuse infrastructure code.

Create a module folder with:
- `main.tf`
- `variables.tf`
- `outputs.tf`

## 23. Difference between `terraform init`, `terraform plan`, and `terraform apply`?
- `init`: Set up the project.
- `plan`: Preview changes.
- `apply`: Execute changes.

## 24. What happens if the state file is lost or corrupted?
You typically need to recover from backup or re-import resources using `terraform import`.

## 25. How do you manage remote state (S3, GCS, Azure Storage)?
Configure backend settings in the `terraform` block.

Example: `terraform { backend "s3" { ... } }`

## 26. What happens if two engineers run `terraform apply` at the same time?
State locking prevents conflicts. One process waits until the lock is released.

## 27. How do you handle team collaboration in Terraform?
Use remote state, state locking, version control, and code reviews.

## 28. How do you manage multiple environments (dev, qa, prod)?
Use separate state files or workspaces.

Example: `terraform workspace new dev`

## 29. Separate state files vs workspaces: which do you prefer?
Separate state files provide stronger isolation. Workspaces are useful for lightweight environment separation.

## 30. How do you manage environment-specific variables?
Use dedicated `*.tfvars` files per environment.

## 31. How do you design infrastructure for high availability?
Use multi-AZ architecture, load balancers, auto-scaling groups, and redundant databases.

## 32. You need to deploy a highly available web application. How do you design it with Terraform?
Create a VPC with subnets across multiple AZs, plus ALB, ASG, and Multi-AZ RDS.

## 33. Your infrastructure is accidentally deleted. How do you recover?
Re-run `terraform apply` if state is intact and configurations are correct.

## 34. Terraform is recreating resources unnecessarily. How do you fix it?
Check for drift, align configs with real infrastructure, and use lifecycle rules such as `prevent_destroy` where appropriate.

## 35. How do you integrate Terraform with CI/CD pipelines?
Run Terraform in pipeline stages with approvals (for example, GitHub Actions or Jenkins).

## 36. What are `depends_on`, `lifecycle`, and `count/for_each`?
- `depends_on`: Explicit dependency management.
- `lifecycle`: Control resource behavior.
- `count`/`for_each`: Resource replication patterns.

## 37. How do you import existing resources into Terraform?
Use `terraform import`.

Example: `terraform import aws_instance.web i-123456`

## 38. What is drift and how do you detect it?
Drift is infrastructure changed outside Terraform. Detect it with `terraform plan`.

## 39. How do you optimize Terraform execution time?
Use modular design, reduce state size, and use targeted operations carefully when needed.

Example: `terraform apply -target=<resource>`

## 40. How do you design VPC/VNet using Terraform?
Define subnets, route tables, gateways, and security groups inside reusable modules.

## 41. How do you ensure security best practices in infrastructure provisioning?
Apply least-privilege IAM, encryption at rest/in transit, secure secret handling, and network segmentation.

## 42. How do you control cost using Terraform?
Use right-sized instance types, auto-scaling, and cost-allocation tags for tracking and optimization.
