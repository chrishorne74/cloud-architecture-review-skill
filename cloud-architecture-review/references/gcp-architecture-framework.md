# Google Cloud Architecture Framework — Review Lens

Five pillars (plus cross-cutting perspectives). For each, the key questions to ask of the architecture and the most common gaps.

## 1. Operational Excellence

Key questions:
- IaC (Terraform is the GCP norm) with CI/CD (Cloud Build/Cloud Deploy or GitHub Actions)?
- Cloud Operations suite (Cloud Monitoring, Logging, Trace) integrated? SLOs defined with error budgets?
- Organized resource hierarchy: organization → folders → projects, with environments separated by project?

Common gaps: everything in one project; no IaC; no SLOs; logs not centralized or retained appropriately.

## 2. Security, Privacy, and Compliance

Key questions:
- Identity: Cloud Identity/Workspace federation; service accounts with least privilege; workload identity federation instead of exported service account keys (exported JSON keys are a classic Critical finding)?
- Network: Shared VPC or hub-spoke; firewall rules scoped; Private Google Access / Private Service Connect for APIs; no public IPs on VMs that don't need them; Cloud Armor on external load balancers?
- Data: CMEK where required; Secret Manager for secrets; VPC Service Controls for data exfiltration protection on sensitive data services (BigQuery, GCS)?
- Posture: Security Command Center enabled; organization policy constraints (e.g., disable external IPs, restrict service account key creation)?

Common gaps: downloaded service account keys; default VPC with default firewall rules; public GCS buckets; no org policies; no VPC-SC around sensitive data; over-privileged default service accounts (Editor role).

## 3. Reliability

Key questions:
- Regional vs zonal resources: regional MIGs, regional GKE clusters, multi-zone Cloud SQL (HA configuration)?
- Defined RTO/RPO; backups (Cloud SQL automated backups, GCS versioning) tested?
- Global vs regional load balancing matched to footprint; health checks configured?
- DR across regions where criticality demands it?

Common gaps: zonal GKE/SQL for production; single-region with no DR statement; no tested restores; Memorystore basic tier (no replication) for critical caching.

## 4. Cost Optimization

Key questions:
- Committed use discounts (CUDs) for steady compute; Spot VMs for batch; sustained use discounts understood?
- Right-sizing via Recommender; autoscaling MIGs/GKE; scale-to-zero with Cloud Run where suitable?
- BigQuery: on-demand vs capacity pricing matched to usage; partitioning/clustering to limit scan costs?
- GCS lifecycle policies and storage classes (Nearline/Coldline/Archive)?
- Budgets and alerts in place; labels for cost attribution?

Common gaps: no CUDs; BigQuery full-table scans from unpartitioned tables; Standard storage for cold data; idle resources in non-prod; no budget alerts.

## 5. Performance Optimization

Key questions:
- Right service for the pattern: Cloud Run vs GKE vs GCE; Spanner vs Cloud SQL vs Firestore vs Bigtable matched to data model and scale?
- Cloud CDN for static/cacheable content; Memorystore for hot data?
- Async with Pub/Sub where latency tolerance allows; Dataflow for streaming pipelines?

Common gaps: Cloud SQL where the access pattern wants Firestore/Bigtable (or vice versa); no CDN for global users; synchronous chains that should be Pub/Sub-decoupled.

## Useful GCP-specific signals in diagrams

- Service account key files or keys flowing to external systems → Security, Critical.
- Default VPC / no Shared VPC in multi-team enterprise context → Security, Medium-High.
- Zonal cluster or non-HA Cloud SQL labelled production → Reliability, High.
- BigQuery fed directly by app writes without Pub/Sub/Dataflow buffering → Performance/Reliability, Medium (verify pattern).
- External HTTP(S) LB without Cloud Armor → Security, Medium.
