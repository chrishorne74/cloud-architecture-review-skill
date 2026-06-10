# Data Transfer Cost Gravity

Use this table to turn qualitative "this may be expensive" findings into rough numbers. All figures are **indicative USD list prices** — they change and vary by region/tier, so for any finding where cost is the headline, verify current pricing (provider pricing pages or pricing tools) and cite the source. Express findings as cost per TB per month at the workload's stated or estimated volume.

| Path | Indicative rate | Per TB | Notes |
|---|---|---|---|
| AWS NAT Gateway processing | $0.045/GB | ~$45 | Plus hourly charge; applies to ALL traffic through it, including S3/DynamoDB — use free gateway VPC endpoints instead |
| AWS cross-AZ | $0.01/GB each direction | ~$20 round trip | Often invisible in designs; chatty service meshes multiply it |
| AWS inter-region | $0.02–$0.08/GB | $20–80 | Replication traffic counts |
| AWS internet egress | ~$0.09/GB (first tiers) | ~$90 | CloudFront egress is cheaper and origin→CloudFront is free — CDN is a cost control, not just performance |
| Azure cross-AZ | Free (charges removed) | $0 | Don't flag Azure cross-AZ traffic on cost grounds |
| Azure inter-region | $0.02–$0.08/GB | $20–80 | Varies by continent pairing |
| Azure internet egress | ~$0.087/GB | ~$87 | First ~100GB/mo free |
| GCP cross-zone | $0.01/GB | ~$10 | |
| GCP inter-region | $0.02–$0.08/GB | $20–80 | |
| GCP internet egress (premium tier) | ~$0.12/GB | ~$120 | Standard tier cheaper; free egress only applies when migrating off GCP entirely |
| **Cross-cloud** | Source cloud's internet egress rate | **~$90–120** | Paid on every request/replication cycle — this is the quantitative teeth behind the same-cloud co-location rule |

## How to apply

1. Identify high-volume flows in the inventory (replication, analytics extracts, service-to-service chatter, media delivery).
2. Estimate monthly volume from whatever the user stated (requests/sec × payload, DB size × replication frequency); state the estimate as an assumption.
3. Multiply and put the number in the finding: *"App in Azure querying the AWS RDS instance at ~5 TB/month ≈ $450/month in AWS egress alone, before latency cost"* lands far harder than *"cross-cloud traffic incurs egress charges."*
4. Classic traps to scan for: NAT gateway in front of S3-heavy workloads (VPC endpoint saves ~$45/TB), no CDN on high-egress public content, cross-AZ Kafka/database replication chatter on AWS, logging/observability pipelines shipping full-fidelity data cross-region.
