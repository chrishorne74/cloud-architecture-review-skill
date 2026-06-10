# Design Principles

The house principles every architecture is evaluated against, in addition to the provider framework pillars. Each gets a row in the report's Principles Compliance table, and findings cite principle IDs (e.g., "violates P1"). A deviation is acceptable only with an explicit, stated justification in the design — undocumented deviations are findings at the severity shown.

| ID | Principle | Default severity when violated |
|---|---|---|
| **P1** | **Same cloud** — a workload and its databases/primary data stores run in the same cloud. | High |
| **P2** | **Same region** — synchronous request-path dependencies (app↔DB, app↔cache, service↔service) are co-located in one region. | High |
| **P3** | **Multi-AZ is the standard resiliency posture** — every production tier is zone-redundant; this satisfies both availability and DR when paired with tested in-region backups. | High (stateful) / Medium (stateless) |
| **P4** | **Cross-region/cross-cloud is non-standard** — permitted only with explicit justification (regulation, tolerances surviving region loss, data residency, edge delivery); unjustified use is over-engineering. | Medium (over-engineering) / High (if it puts sync dependencies across boundaries) |
| **P5** | **Async across boundaries** — anything crossing a region, cloud, or on-prem boundary is buffered (queue, replication, batch), never a synchronous request-path call. | High |
| **P6** | **Right-fit integration** — events/webhooks over polling, APIs/contracts over shared databases, managed services over self-managed equivalents, no third-party sync dependency on the critical path without timeout/fallback. | Medium |
| **P7** | **Evidence-based resiliency** — RTO/RPO claims trace to actual mechanisms (backup tested, failover rehearsed); a diagram arrow is not a capability. | High |
| **P8** | **Demonstrable security baseline** — data classified, encrypted at rest and in transit, least-privilege access, secrets vaulted, private by default with minimal public surface. | High |
| **P9** | **Operable by default** — IaC, pipeline-deployed, monitored with actionable alerts; no console-managed production. | Medium |
| **P10** | **Cost-aware data flow** — high-volume paths avoid avoidable transfer charges (NAT for S3-class traffic, missing CDN, chatty cross-AZ/region flows); estimates quantified per cost-gravity reference. | Medium |

## How to apply

1. After the pillar assessment (Step 6), walk P1–P10 against the inventory and record each as **Compliant / Deviation (justified) / Violation / Not applicable**.
2. A *justified* deviation needs the justification visible in the supplied design or stated by the user — reviewer charity doesn't count. Record the justification in the table.
3. Principle violations and pillar findings overlap by design — don't duplicate the finding text; the principles table cites the finding, the finding cites the principle ID.

## Report section

Insert after the Scorecard:

```markdown
## Principles Compliance
| ID | Principle | Status | Evidence / Justification |
|---|---|---|---|
```

One line per principle, including the compliant ones — the table's value is showing what was checked, not just what failed.
