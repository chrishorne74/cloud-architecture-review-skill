# Cross-Cutting Review Checklist

Provider-agnostic checks to apply on every review, alongside the provider framework. These catch gaps that pillar-by-pillar review can miss.

## Trust boundaries & data flow

- [ ] Every arrow crossing a trust boundary (internet → DMZ → internal → data) is authenticated, authorized, and encrypted.
- [ ] Ingress paths are minimal and fronted by a managed edge (CDN/WAF/LB) — no direct-to-compute exposure.
- [ ] Egress is controlled (NAT/firewall/proxy), not unrestricted outbound.
- [ ] Admin/management plane access is separated from the data plane (bastion, VPN, ZTNA, IP allowlist + MFA).

## Co-location & latency (strong defaults)

- [ ] Workload and its databases/primary data stores are in the **same cloud**. Cross-cloud app↔database is a High finding unless explicitly justified.
- [ ] Synchronous request-path dependencies (app↔DB, app↔cache, service↔service) are in the **same region**.
- [ ] Every cross-region or cross-cloud link is async/buffered (replication, queue, batch) — or has a stated justification (DR, data residency, edge).
- [ ] Latency-sensitive paths have no avoidable network-boundary hops; estimate added latency per hop when flagging.
- [ ] Cross-boundary data transfer costs (egress, cross-AZ/region) considered for high-volume flows.

## Resiliency = Availability + DR (assess separately)

- [ ] **Availability**: every production tier is multi-AZ (the standard baseline) — compute, database, cache, LB, NAT. Single-AZ stateful = High, stateless = Medium.
- [ ] **DR**: a distinct answer exists for region loss, matched to RTO/RPO (backup-restore → pilot light → warm standby → active-active). Multi-AZ is never the DR answer.
- [ ] The design doesn't conflate the two, and doesn't gold-plate (multi-region with no requirement driving it).

## Single points of failure

Walk the diagram and ask "what happens if this box disappears?" for every component. Classic SPOFs:
- [ ] Single NAT gateway / firewall / load balancer instance.
- [ ] Single-zone database or cache.
- [ ] One message broker node; one DNS dependency.
- [ ] A shared service (auth, config) that everything synchronously depends on.

## Data lifecycle

- [ ] Classification: is sensitive data (PII, payment, health) identified and treated differently?
- [ ] Backups exist, are isolated (separate account/project/immutable), and restore has been tested.
- [ ] Retention and deletion policies exist (compliance + cost).
- [ ] Data residency requirements vs chosen regions.

## Industry guidelines to map findings against (when applicable)

| Context | Guideline |
|---|---|
| General hardening | CIS Benchmarks (per-provider foundations benchmark) |
| US federal / general control framework | NIST 800-53, NIST Cybersecurity Framework |
| Network/identity philosophy | Zero Trust (NIST 800-207) |
| Payment data | PCI DSS v4 |
| Health data (US) | HIPAA |
| EU personal data | GDPR (data residency, minimization, erasure) |
| Service org reporting | SOC 2 trust criteria |
| InfoSec management | ISO 27001 |
| Australian banks/insurers/super (APRA-regulated) | CPS 230 + CPS 234 — see [apra-cps230-cps234.md](apra-cps230-cps234.md) |

Only invoke these when the workload context makes them relevant; cite the specific control area, not just the framework name.

## Anti-patterns to scan for

- Lift-and-shift VMs running workloads that have an obvious managed equivalent (self-managed DB/queue/cache on VMs) — flag with the managed alternative.
- Hardcoded credentials, connection strings, or API keys anywhere in the flow.
- Environments not isolated (shared prod/non-prod network, account, or cluster).
- No tagging/labeling convention (breaks cost allocation, automation, and incident response).
- Architecture diagrams showing only the happy path — no failure, scaling, or DR annotations.
- Deprecated or end-of-life services/runtimes (verify with web search).

## Review quality bar

- Every finding names a component from the inventory or a demonstrable absence.
- Severity is justified by impact + likelihood, not vibes.
- Recommendations name specific services/patterns and rough effort.
- Unknowns become Open Questions, never assumed findings.
