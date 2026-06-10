# Azure Well-Architected Framework — Review Lens

Five pillars. For each, the key questions to ask of the architecture and the most common gaps. Also check alignment with Azure Landing Zone / Cloud Adoption Framework concepts for enterprise architectures.

## 1. Reliability

Key questions:
- Availability Zones used for all zonal services (VMs, AKS, App Service premium, SQL)? Zone-redundant SKUs selected?
- Defined SLO/RTO/RPO? Composite SLA of chained services calculated?
- Azure Backup / geo-redundant storage (GRS/GZRS) where data loss is unacceptable?
- Health probes on Load Balancer/Application Gateway/Front Door? Autoscale rules configured?
- DR: zone-redundant deployment + tested backups is the standard posture; paired-region/Site Recovery/active-active only where stated tolerances require surviving region loss.

Common gaps: single-instance VMs with no availability set/zone; LRS storage for critical data; no tested failover; App Service on a single instance; AKS with a single node pool and no zone spread.

## 2. Security

Key questions:
- Identity: Entra ID as the control plane; managed identities instead of service principals with secrets; PIM for privileged roles; Conditional Access enforced?
- Network: hub-spoke or Virtual WAN topology; NSGs + Azure Firewall; private endpoints for PaaS (Storage, SQL, Key Vault) instead of public endpoints; WAF on Application Gateway/Front Door?
- Data: encryption at rest (default) + customer-managed keys where required; TLS enforced; Key Vault for secrets/certs with RBAC and purge protection?
- Posture: Microsoft Defender for Cloud enabled across subscriptions; Azure Policy guardrails; Sentinel or equivalent SIEM?

Common gaps: PaaS services exposed on public endpoints; secrets in app settings instead of Key Vault references; no private endpoints; flat network with no segmentation; service principals with client secrets where managed identities would work; no Azure Policy enforcement.

## 3. Cost Optimization

Key questions:
- Reservations / Savings Plans for steady compute; spot for interruptible work; Hybrid Benefit for Windows/SQL licenses?
- Right SKU and tier (e.g., Premium where Standard suffices)? Dev/test pricing for non-prod?
- Autoscale and scale-to-zero (Functions consumption, Container Apps) where workload is spiky?
- Cost Management budgets, alerts, and tag-based allocation in place?

Common gaps: no reservations on stable VMs; Hybrid Benefit unused; non-prod running 24/7; ExpressRoute/Front Door SKUs oversized; orphaned disks and public IPs; no budget alerts.

## 4. Operational Excellence

Key questions:
- IaC (Bicep/Terraform/ARM) with CI/CD via Azure DevOps or GitHub Actions?
- Safe deployment practices: staging slots, canary/blue-green, automated rollback?
- Azure Monitor + Log Analytics + Application Insights wired into all tiers? Actionable alerts mapped to runbooks?

Common gaps: portal-managed production; no centralized Log Analytics workspace; alerts that page nobody; no deployment slots for App Service; missing diagnostics settings on resources.

## 5. Performance Efficiency

Key questions:
- Right service and tier for the workload pattern (e.g., Cosmos DB vs SQL, Functions vs AKS)?
- Caching: Azure Cache for Redis, CDN/Front Door caching for static and edge content?
- Async patterns with Service Bus/Event Grid/Queue Storage where appropriate?
- Performance testing and autoscale validation done?

Common gaps: no caching tier; synchronous coupling between services; single-region deployment serving global users without Front Door; under-provisioned DTU/vCore SQL with no elastic pool consideration.

## Useful Azure-specific signals in diagrams

- Storage account / SQL / Key Vault with public network access and no private endpoint → Security, High.
- Single VM with no availability set/zone for a production workload → Reliability, High.
- No hub-spoke / firewall in an enterprise multi-workload diagram → Security, Medium-High.
- Connection strings/secrets shown flowing to app config → Security, High.
- No Front Door/Traffic Manager in a multi-region diagram → Reliability/Performance, Medium.
- Multi-region deployment with no stated requirement driving it → over-engineering, check justification.
