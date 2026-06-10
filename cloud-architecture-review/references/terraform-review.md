# Terraform Review Guidance

When the architecture input is Terraform, the code is the source of truth — it is machine-checkable evidence, not a sketch. Read all `.tf` files (and `.tfvars`, module sources where local). Findings from code get higher confidence than diagram findings: cite the file and resource address (e.g., `aws_db_instance.main` in `rds.tf`).

## Run scanners if available

Before manual review, check whether `checkov`, `tfsec`, or `trivy` is installed and run it against the configuration (e.g., `checkov -d . --compact`). Fold results into the findings: deduplicate against your own observations, keep the scanner's rule ID as evidence, and re-rate severity using this skill's scale — scanner severities are often inflated or context-blind. Also run `terraform validate` if the working directory is initialized. If no scanner is available, say so and proceed manually.

## What to extract from the code

Build the Step 2 inventory from resources, then check these Terraform-specific signals:

### Reliability
- AZ spread: `count`/`for_each` over multiple AZs vs a hardcoded single `availability_zone`; one subnet per tier is a single-AZ design.
- `multi_az = true` on RDS; `aws_db_instance` without it in prod is a High finding.
- `deletion_protection` on databases and load balancers; `lifecycle { prevent_destroy = true }` on stateful resources.
- `skip_final_snapshot = true` on production databases — flag it.
- Backup settings: `backup_retention_period = 0`, missing `aws_backup_plan` coverage.

### Security
- Security groups/firewalls: ingress `cidr_blocks = ["0.0.0.0/0"]`, especially ports 22/3389/database ports.
- Public exposure: `publicly_accessible = true`, `acl = "public-read"`, missing `aws_s3_bucket_public_access_block`, `associate_public_ip_address = true` on workloads.
- Encryption flags absent or false: `storage_encrypted`, `encrypted`, `sse_algorithm`/`server_side_encryption_configuration`, `kms_key_id` defaulting to provider-managed where the data classification wants CMK.
- IAM: inline policies with `Action = "*"` or `Resource = "*"`; `iam:PassRole` on `*`.
- **Secrets in code**: hardcoded passwords/keys in resource arguments, `variable` defaults, or `.tfvars` committed to the repo. Also flag secrets that transit Terraform **state** (e.g., `random_password` → DB password): state must then be treated as sensitive.
- Logging/audit resources present: flow logs, `aws_cloudtrail`, access logging blocks on buckets/LBs.

### State and pipeline hygiene
- Remote backend with encryption and locking (`backend "s3"` with `encrypt = true` + lock table, or equivalent). Local state for shared infrastructure is a finding.
- `required_version` and provider version constraints pinned; unpinned module `source` refs (no version/tag) are supply-chain findings.
- Default tags / labels configured (`default_tags`) — absence breaks cost allocation (Cost pillar).

### Structure
- Separate state/workspaces per environment; prod and non-prod in one state file is an Operational Excellence finding.
- Module boundaries sane: a 2,000-line `main.tf` monolith is a maintainability note (Low), not a pillar violation.

## What Terraform can't show

Don't infer absence of monitoring dashboards, runbooks, or org-level guardrails from a workload-level module — those may live in another repo. Route them to Open Questions instead of findings, and say which repo/layer you'd expect them in.
