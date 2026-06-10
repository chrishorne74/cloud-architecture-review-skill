# Worked Example Review

Follow this example for tone, severity calibration, and report shape. Note what it does: every finding names a component, severities are justified by impact + likelihood, costs and latencies are estimated with stated assumptions, unknowns become questions, and strengths are genuine, not filler.

---

## Input supplied by the user

> Review this design. Retail web app, ~50k users mostly in Australia, checkout must feel instant. ALB in ap-southeast-2 → ECS Fargate (2 services: web, orders) in public subnets → orders service calls our product database, which is Azure SQL in Australia East (the data team owns it and standardized on Azure). Sessions in ElastiCache Redis, single node. Images in S3 served straight from the bucket. Deployed with Terraform via GitHub Actions.

## Output

# Architecture Review: Retail Web App (AWS ap-southeast-2 + Azure SQL)

## Summary

The compute and deployment foundations are sound (Fargate, IaC, CI/CD), but the architecture has two structural problems: the orders service depends synchronously on a database in a different cloud, and both the cache and (apparently) the database have no redundancy. Top three actions: move the product data into AWS or put a read replica/cache in front of it, take workloads out of public subnets, and make the session store and database highly available.

## Architecture Understood

- **Edge/compute**: ALB → ECS Fargate (web, orders) in ap-southeast-2, **public subnets**
- **Data**: Azure SQL (Australia East, cross-cloud), ElastiCache Redis **single node** (sessions), S3 (product images, direct serving)
- **Pipeline**: Terraform via GitHub Actions

Assumptions: "instant checkout" read as p95 < 500ms end-to-end; product DB is on the synchronous checkout path; no DR requirement stated, assumed RTO ≤ 4h / RPO ≤ 1h for a retail revenue path. Monitoring, WAF, and backups were not mentioned — treated as absent pending confirmation.

## NFR Compliance

| Requirement | Source | Status | Evidence |
|---|---|---|---|
| Checkout p95 < 500ms | Stated ("instant") | **Misses** | Cross-cloud DB call adds ~5–15ms per query AWS↔Azure within Sydney; an N+1 pattern at 10 queries/checkout adds 50–150ms before app time |
| Users in Australia served locally | Stated | Meets | ap-southeast-2 + Australia East are both Sydney |
| RTO ≤ 4h / RPO ≤ 1h | Assumed | **Unknown→At risk** | No backup, failover, or multi-AZ evidence for Redis or Azure SQL tier |

## Scorecard

| Pillar | Rating | Critical | High | Medium | Low |
|---|---|---|---|---|---|
| Security | At Risk | 0 | 2 | 1 | 0 |
| Reliability | At Risk | 0 | 2 | 1 | 0 |
| Performance | Needs Attention | 0 | 1 | 1 | 0 |
| Cost | Needs Attention | 0 | 0 | 2 | 0 |
| Operational Excellence | Adequate | 0 | 0 | 1 | 0 |
| Sustainability | Adequate | 0 | 0 | 0 | 0 |

## Strengths

- Fargate removes host patching burden (Security, Ops Excellence) and scales to demand (Cost).
- Terraform + GitHub Actions: reproducible infrastructure with an audit trail (Ops Excellence).
- Region choice matches the user base (Performance).

## Findings & Recommendations

### Latency & Integration Fit

**[High] Cross-cloud synchronous database dependency** — `orders` (AWS) → Azure SQL on the checkout path. Violates the same-cloud co-location default: per-query latency, Azure egress charges on every read (~$87/TB), two security perimeters, and an outage in either cloud breaks checkout. The data team's Azure preference is an organizational reason, not an architectural one — it doesn't need to put Azure on the request path. **Recommend** (pick one): replicate product data to an AWS read store (DynamoDB/Aurora) fed async from Azure SQL (effort: Medium); or accept Azure as system of record but cache aggressively in the existing Redis with sensible TTLs (effort: Low, partial fix). Re-rate to Critical if checkout writes also cross clouds — see Open Questions.

### Security

**[High] Workloads in public subnets** — both ECS services are directly addressable. Compromise of a task exposes it to the internet without the ALB's protections. **Recommend**: private subnets + NAT (or VPC endpoints), ALB stays public (effort: Low — subnet/routing change in Terraform).

**[High] No WAF in front of a public retail app** — card-adjacent retail traffic with no L7 protection. **Recommend**: AWS WAF on the ALB with managed rule sets (effort: Low).

**[Medium] Cross-cloud data path security unverified** — how does `orders` authenticate to Azure SQL, and is the secret in Secrets Manager? Connection over public internet vs private link changes the rating — verification question, provisionally Medium.

### Reliability

**[High] Single-node Redis holding sessions** — node loss logs out every user and, if carts live in session, empties carts mid-checkout. **Recommend**: ElastiCache replication group, multi-AZ with automatic failover (effort: Low).

**[High] No backup/HA evidence for the product database** — Azure SQL tier and geo-redundancy unknown; for a revenue path this is At Risk until shown otherwise. **Recommend**: confirm zone-redundant tier + PITR; test a restore (effort: Low to verify).

**[Medium] No health-based autoscaling or failure-mode design stated** — single-AZ task placement would make this High; Fargate behind ALB with multi-AZ subnets largely mitigates if configured. Verify subnet spread in Terraform.

### Performance

**[High]** — covered by the cross-cloud finding above (dominant latency contributor).

**[Medium] S3 images served directly** — no CDN: higher latency for non-Sydney users, no caching, S3 egress at ~$90/TB vs CloudFront's lower rates and free origin fetch. **Recommend**: CloudFront in front of the bucket with OAC (effort: Low).

### Cost

**[Medium] Azure egress on every product read** — at an assumed 2 TB/month read volume, ~$175/month avoidable spend; grows linearly with traffic. Fixed by the co-location recommendation.

**[Medium] No tagging/budget evidence** — confirm `default_tags` and a budget alert exist (effort: Low).

### Operational Excellence

**[Medium] No monitoring/alerting mentioned** — no CloudWatch alarms, dashboards, or tracing stated for a revenue path. **Recommend**: ALB 5xx/latency alarms, ECS service alarms, and X-Ray or equivalent on checkout (effort: Medium).

## Prioritized Action Plan

1. Private subnets for ECS tasks (High, Low effort)
2. WAF on ALB (High, Low effort)
3. Redis replication group with multi-AZ (High, Low effort)
4. Verify Azure SQL tier, backups, and connection path (High, Low effort to verify)
5. Decide product-data strategy: replicate into AWS or cache-first (High, Medium effort)
6. CloudFront for images (Medium, Low effort)
7. Monitoring baseline on checkout path (Medium, Medium effort)

## Open Questions

1. Do checkout **writes** go to Azure SQL, or only reads? (Determines whether the cross-cloud finding is High or Critical.)
2. How does `orders` connect to Azure SQL — public endpoint or private link, and where does the credential live?
3. What are the actual RTO/RPO targets, and what Azure SQL service tier is in use?
4. Are ECS tasks spread across ≥2 AZs in the Terraform?

## References

- AWS Well-Architected Framework — Reliability & Security pillars
- Same-cloud/same-region co-location defaults (this skill, Step 3)
- Indicative egress rates: cost-gravity reference (verify current pricing)

---

*Calibration notes: nothing here was rated Critical because no confirmed data-loss/breach condition was shown — single-node Redis is High, not Critical, because sessions are rebuildable. The cross-cloud finding leads the report because it's structural; subnet and WAF fixes outrank it in the action plan only because they're near-zero effort.*
