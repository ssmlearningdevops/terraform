# Terraform Interview Scenarios

## 1. Why doesn't Terraform detect manual changes immediately?
**Answer:** Terraform does not monitor infrastructure in real time. It refreshes state during `terraform plan`, `terraform apply`, and `terraform refresh`.

If drift is not detected, either:
- The last refresh happened before the manual change, or
- `ignore_changes` is configured.

**Fix:** Run `terraform refresh` and reconcile any out-of-band changes.

## 2. What happens if a teammate's apply crashes and you see "state lock acquired"?
**Answer:** Terraform uses state locking (for example, DynamoDB with S3) to prevent concurrent state writes.

**Fix:**
1. Confirm the original process is no longer running.
2. Run `terraform force-unlock <LOCK_ID>`.

Never force-unlock while apply is still active, or state corruption can occur.

## 3. Can Terraform import a resource that was manually deleted?
**Answer:** No. Once a resource is deleted, it cannot be imported. Terraform state may still show it until refresh runs.

**Fix flow:**
1. Run `terraform refresh`.
2. State marks the resource as destroyed.
3. Plan shows a create action.

Then either allow recreation or remove the stale entry with `terraform state rm` and restore manually if needed.

## 4. Why might reading outputs from another team's remote state fail?
**Answer:** Common causes are:
- IAM permission issues (for example, missing `s3:GetObject`), or
- Referencing an outdated or incorrect state path.

**Fix:** Validate IAM permissions and confirm the correct workspace prefix and state version.

## 5. Why would `terraform init` suddenly show many replacements with no code changes?
**Answer:** Usually provider version drift. Without version constraints, Terraform may pull a newer provider that marks some attributes as `ForceNew`.

**Fix:** Pin provider versions and commit `terraform.lock.hcl`.

## 6. How do you resolve a dependency cycle between cluster and service modules?
**Answer:** Break the cycle with a two-phase deployment or by using data sources.

**Approach:**
1. Create the cluster first.
2. Read required cluster data (for example, `aws_eks_cluster_auth`).
3. Plan and apply the service module.

## 7. Why does a plan take 25 minutes for 10,000 resources?
**Answer:** A monolithic state file is usually the bottleneck.

**Fix:** Split infrastructure into layers (Networking, Security, Compute, Database), each with a separate state file using Terragrunt, stacks, or workspaces.

This reduces plan time and blast radius.

## 8. Why might a sensitive variable still appear in plain text?
**Answer:** `sensitive = true` only redacts values in certain CLI contexts. If the value is exposed through an output that is not marked sensitive, it can still print.

**Fix:** Mark related outputs as sensitive and use a dedicated secret manager (for example, Vault or AWS Secrets Manager).

## 9. Why can't Terraform destroy an RDS or Cloud SQL instance with deletion protection enabled?
**Answer:** The cloud provider API blocks deletion while deletion protection is enabled.

**Fix:**
1. Set `deletion_protection = false`.
2. Run `terraform apply`.
3. Run `terraform destroy`.

## 10. Why does Terraform always show a diff for Kubernetes annotations?
**Answer:** Kubernetes controllers often mutate annotations server-side after apply.

**Fix:** Use `lifecycle.ignore_changes` for the affected annotation fields so Terraform ignores automatic updates.
