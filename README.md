# Terraform Notes

## Lock File

- `terraform.lock.hcl` ensures Terraform uses consistent provider versions across runs for reliability and security.

## Init Command Variants

- `terraform init -migrate-state`
  - Moves existing Terraform state to a new backend while preserving data.
- `terraform init -reconfigure`
  - Resets backend settings and ignores saved backend configuration.
- `terraform init -upgrade`
  - Upgrades provider versions and refreshes `.terraform.lock.hcl`.

## Core Commands

- `terraform plan`
- `terraform plan -replace="aws_instance.example"`
  - Recreates a specific resource.
  - Preferred over deprecated `taint` workflows because it is explicit and previewable in `plan`.
- `terraform apply -auto-approve`
- `terraform destroy -auto-approve`

## Resource Lifecycle

- `create_before_destroy` (boolean): create replacement first, then destroy old resource (useful for near zero-downtime updates).
- `prevent_destroy` (boolean): protects a resource from accidental destruction.
- `ignore_changes` (list of attributes): ignores changes to selected attributes so they do not trigger updates.

Example:

```hcl
resource "aws_autoscaling_group" "example" {
  name                 = "example-asg"
  desired_capacity     = 2
  max_size             = 5
  min_size             = 1
  launch_configuration = aws_launch_configuration.example.name

  lifecycle {
    ignore_changes        = [desired_capacity]
    create_before_destroy = true
    prevent_destroy       = true
  }
}
```

## Using `count`

`count` lets you create multiple copies of a resource from a number.

```hcl
resource "aws_instance" "example" {
  count         = 3
  ami           = "ami-123456"
  instance_type = "t2.micro"
}
```

## Conditional Resource Creation

You can use conditional expressions with `count`.

```hcl
resource "aws_instance" "example" {
  count         = var.create_instance ? 1 : 0
  ami           = "ami-123456"
  instance_type = "t2.micro"
}
```

- If `var.create_instance` is `true`, Terraform creates 1 instance.
- If `false`, Terraform creates 0 instances.

## Using `for_each`

`for_each` creates one resource per item in a map or set.

```hcl
resource "aws_s3_bucket" "env_buckets" {
  for_each = {
    dev     = "my-dev-bucket"
    staging = "my-staging-bucket"
    prod    = "my-prod-bucket"
  }

  bucket = each.value

  tags = {
    Environment = each.key
  }
}
```

Terraform creates:

- `my-dev-bucket` with `Environment = dev`
- `my-staging-bucket` with `Environment = staging`
- `my-prod-bucket` with `Environment = prod`

Reference individual resources:

- `aws_s3_bucket.env_buckets["dev"].id`
- `aws_s3_bucket.env_buckets["prod"].id`

## Terraform Version Migration Checklist

1. Upgrade Terraform and providers with `terraform init -upgrade`.
2. Run `tflint` to catch syntax and style issues.
3. Run security scanners (`tfsec`, `checkov`, or `terrascan`).
4. Fix flagged issues.
5. Run `terraform plan` to validate migration impact.

## Backend Migration: S3 + DynamoDB to S3 Native Locking (v1.10+)

1. Back up your state.
2. Update backend configuration:
   - Remove `dynamodb_table = "my-lock-table"`
   - Add `use_lockfile = true`
3. Enable S3 bucket versioning.
4. Re-initialize with `terraform init -migrate-state`.
5. Test locking behavior.
6. Clean up DynamoDB lock table.
