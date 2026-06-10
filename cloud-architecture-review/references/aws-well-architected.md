# AWS Well-Architected Framework — Review Lens

Six pillars. For each, the key questions to ask of the architecture and the most common gaps.

## 1. Operational Excellence

Key questions:
- How are changes deployed? Is there CI/CD, or manual console changes?
- Is infrastructure defined as code (CloudFormation/Terraform/CDK)?
- Are there runbooks/playbooks for failure scenarios?
- How is operational health measured (dashboards, KPIs)?

Common gaps: no IaC; no deployment pipeline; no rollback strategy; console-managed production; no game days or failure testing.

## 2. Security

Key questions:
- Identity: least-privilege IAM? Roles instead of long-lived access keys? MFA on human access? Identity Center/federation for workforce?
- Detection: CloudTrail (org-wide, all regions), GuardDuty, Security Hub, Config enabled?
- Infrastructure protection: private subnets for workloads, security groups scoped tightly, no 0.0.0.0/0 on management ports, WAF in front of public endpoints?
- Data protection: encryption at rest (KMS) and in transit (TLS) everywhere; S3 Block Public Access; secrets in Secrets Manager/Parameter Store, never in code or env files?
- Multi-account strategy: separate accounts for prod/non-prod/security via Organizations? SCPs as guardrails?

Common gaps: workloads in public subnets; databases publicly reachable; single account for everything; secrets in plaintext; no GuardDuty/CloudTrail; over-broad IAM (`*` actions/resources); no WAF on internet-facing apps.

## 3. Reliability

Key questions:
- Multi-AZ for all stateful and stateless tiers? (Single-AZ anything is a finding.)
- Defined RTO/RPO? Backups (AWS Backup) tested via restore?
- Auto Scaling / self-healing for compute? Health checks on load balancers?
- Service quotas considered? Throttling/retry/backoff in service-to-service calls?
- DR strategy matching business need: backup-restore, pilot light, warm standby, multi-region active-active?

Common gaps: single-AZ RDS; no backups or untested backups; single NAT gateway as SPOF; no autoscaling; synchronous tight coupling with no queue/buffer; no DR plan; DNS/failover not configured.

## 4. Performance Efficiency

Key questions:
- Right service for the job (e.g., is a relational DB being used as a queue or document store)?
- Right-sizing evidence: Compute Optimizer, instance family currency (Graviton where applicable)?
- Caching at appropriate layers: CloudFront for static/edge, ElastiCache/DAX for data?
- Async/event-driven where latency tolerance allows (SQS, EventBridge, Step Functions)?

Common gaps: no CDN for global users; no caching tier in read-heavy paths; oversized or legacy instance generations; chatty synchronous microservice calls; serverless not considered for spiky workloads.

## 5. Cost Optimization

Key questions:
- Tagging strategy for cost allocation? Budgets and alerts configured?
- Commitment discounts (Savings Plans/RIs) for steady-state; Spot for fault-tolerant work?
- Lifecycle policies on S3 (intelligent tiering, expiry)? Old snapshots/EBS volumes cleaned up?
- Data transfer cost considered (cross-AZ, cross-region, NAT gateway processing, egress)?

Common gaps: everything on-demand; no tagging; idle non-prod environments running 24/7; NAT gateway data processing for high-volume traffic that could use VPC endpoints; gp2 instead of gp3; no S3 lifecycle rules.

## 6. Sustainability

Key questions:
- Region selection considering carbon intensity where latency/compliance allows?
- Utilization maximized (right-sizing, serverless, autoscaling to zero where possible)?
- Data lifecycle management to avoid storing unneeded data?

Common gaps: heavily underutilized fleets; no archival/expiry of cold data. Treat as Low severity unless the user states sustainability goals.

## Useful AWS-specific signals in diagrams

- Public subnet containing app/database tiers → Security, Critical/High.
- No VPC endpoints with heavy S3/DynamoDB traffic through NAT → Cost, Medium.
- Single region with no DR statement → Reliability, severity depends on workload criticality.
- IGW directly attached to instances rather than via ALB/NLB → Security/Reliability.
- Absence of CloudWatch/CloudTrail/X-Ray in the diagram → flag as verification question.
