# Terraform Interview Scenarios

## 1. Your CI/CD pipeline ran successfully, but a manual security group rule was added via the cloud console three hours ago. The next terraform plan shows no changes. Why didn't Terraform detect the drift immediately?
**Answer:** Terraform does not monitor infrastructure in real time. It refreshes state during `terraform plan`, `terraform apply`, and `terraform refresh`.

If drift is not detected, either:
- The last refresh happened before the manual change, or
- `ignore_changes` is configured.

**Fix:** Run `terraform refresh` and reconcile any out-of-band changes.

## 2. Your teammate in Pune ran terraform apply and their laptop lost power. Now, every subsequent run fails with an "Error: state lock acquired by another process". How do you resolve this without corrupting the infrastructure?
**Answer:** Terraform uses state locking (for example, DynamoDB with S3) to prevent concurrent state writes.

**Fix:**
1. Confirm the original process is no longer running.
2. Run `terraform force-unlock <LOCK_ID>`.

Never force-unlock while apply is still active, or state corruption can occur.

## 3. Someone in the Chennai team manually deleted a production database from the cloud console. terraform plan now says it will create a new database. But you don't want a new one; you want to import the old one (which is gone). What is the conceptual flaw in the engineer's assumption?
**Answer:** No. Once a resource is deleted, it cannot be imported. Terraform state may still show it until refresh runs.

**Fix flow:**
1. Run `terraform refresh`.
2. State marks the resource as destroyed.
3. Plan shows a create action.

Then either allow recreation or remove the stale entry with `terraform state rm` and restore manually if needed.

## 4. A networking team in Hyderabad exports a VPC ID via a remote state output. Your application team's Terraform fails because it cannot read that output. The network state file exists in S3. Why might this fail conceptually?
**Answer:** Common causes are:
- IAM permission issues (for example, missing `s3:GetObject`), or
- Referencing an outdated or incorrect state path.

**Fix:** Validate IAM permissions and confirm the correct workspace prefix and state version.

## 5. The Mumbai team wrote Terraform code six months ago. Today, a junior engineer runs `terraform init` on a new machine. The plan suddenly shows 50 resources to replace, not update. No code changed. What happened conceptually?
**Answer:** The junior engineer likely omitted or misconfigured the required_providers block. Without version constraints, terraform init pulls the latest major provider version. Cloud providers (AWS, Azure, GCP) often introduce schema changes in major versions (e.g., v4 → v5) where an attribute becomes ForceNew (causing replacement). 

**Fix:** The solution is to lock the provider version in `terraform.lock.hcl` and always commit it to Git. This scenario is why "it works on my machine" fails in a team setting.

## 6. You are deploying a Kubernetes cluster (EKS/AKS/GKE) and a service on that cluster in the same terraform apply. The plan fails with "Cycle: module.eks, module.service". You cannot split the code due to business policy. How do you break the cycle conceptually?
**Answer:** This is a classic chicken-and-egg problem. Terraform builds a dependency graph. The cluster needs the service to exist? No. The service needs the cluster's API endpoint. Break the cycle with a two-phase deployment or by using data sources.

**Approach:**
1. Create the cluster first.
2. Read required cluster data (for example, `aws_eks_cluster_auth`).
3. Plan and apply the service module.

## 7. Your Delhi NCR team manages 10,000 resources in a single state file. terraform plan now takes 25 minutes to run, breaking your SLAs. What is the architectural solution, not a code fix?
**Answer:** A monolithic state file is usually the bottleneck.

**Fix:** Split infrastructure into layers (Networking, Security, Compute, Database), each with a separate state file using Terragrunt, stacks, or workspaces.

This reduces plan time and blast radius.

## 8. A developer in Kolkata runs terraform plan to debug a module. The plan prints the entire module output, including a sensitive = true database password. But the password is shown in plain text. Why did Terraform fail to protect it?
**Answer:** `sensitive = true` only redacts values in certain CLI contexts. If the value is exposed through an output that is not marked sensitive, it can still print.

**Fix:** Mark related outputs as sensitive and use a dedicated secret manager (for example, Vault or AWS Secrets Manager).

## 9. You run terraform destroy to clean up a test environment in Pune. It fails with "Error: deletion protection enabled" for an RDS database or GCP Cloud SQL instance. The state file shows the resource exists. How do you force the destroy without manually clicking in the console?
**Answer:** The cloud provider API blocks deletion while deletion protection is enabled.

**Fix:**
1. Set `deletion_protection = false` in resource's configuration.
2. Run `terraform apply`.
3. Run `terraform destroy`.

## 10. Every time the Bengaluru team runs terraform apply on their Kubernetes cluster, it shows a diff for a single annotation (e.g., kubectl.kubernetes.io/last-applied-configuration), even though no one changed the code. Why is this not a bug, and how do you fix the noise?
**Answer:** Kubernetes controllers often mutate annotations server-side after apply.

**Fix:** Use `lifecycle.ignore_changes` for the affected annotation fields so Terraform ignores automatic updates.
