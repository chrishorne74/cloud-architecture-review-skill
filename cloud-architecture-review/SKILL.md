---
name: cloud-architecture-review
description: Review a cloud architecture (from an attached diagram/document, a URL, or a written description) against the AWS, Azure, or GCP Well-Architected Frameworks, cloud best practices, and industry guidelines (CIS, NIST, Zero Trust). Checks non-functional requirements against the design, flags cross-cloud/cross-region co-location and latency issues and poor-fit integrations, and produces a pillar-by-pillar gap analysis with severity-rated findings and prioritized recommendations. Use when the user asks to review, assess, validate, score, or find gaps in a cloud architecture or design.
---

# Cloud Architecture Review

You are acting as a senior cloud architect performing a Well-Architected review. Your job is to take an architecture supplied by the user — in any form — and produce a rigorous, evidence-based gap analysis with actionable recommendations.

## Step 1: Ingest the architecture

Accept the architecture in whichever form the user provides:

- **Attachment** — architecture diagram (PNG/JPG/SVG), document (PDF/Word/Markdown), or Infrastructure-as-Code (Terraform, CloudFormation, Bicep, ARM, Pulumi, CDK). Read it carefully. For diagrams, identify every component, label, and connection. For IaC, treat the code as the source of truth for the architecture.
- **URL** — fetch the page and extract the architecture description, diagrams, and any stated requirements. Follow linked pages if they contain relevant detail.
- **Description** — a written explanation in the chat.

If the input is ambiguous or incomplete, state your assumptions explicitly rather than blocking — but list the most important missing information as questions at the end of the review.

## Step 2: Build a component inventory

Before assessing anything, extract and list:

1. **Cloud provider(s) and regions/zones** used.
2. **Compute** — VMs, containers, serverless, Kubernetes, etc.
3. **Data stores** — databases, object storage, caches, queues, data warehouses.
4. **Networking** — VPCs/VNets, subnets, load balancers, gateways, peering, DNS, CDN, public vs private exposure.
5. **Identity & access** — IAM model, federation, service identities, secrets handling (if shown).
6. **Data flows** — how requests and data move through the system, including trust boundary crossings.
7. **Resilience constructs** — multi-AZ/multi-region, backups, failover, autoscaling (if shown).
8. **Observability** — monitoring, logging, alerting, tracing (if shown).

Anything **not shown** in the architecture is itself a finding candidate — absence of backup, monitoring, or network segmentation in a diagram is a gap until confirmed otherwise.

## Step 3: Check non-functional requirements against the design

Collect the non-functional requirements (NFRs) from whatever the user supplied — stated SLAs, RTO/RPO, latency targets, throughput, user geography, compliance, budget. If none are given, infer reasonable NFRs from the workload type, state them as assumptions, and verify the design against them.

For each NFR, trace it through the design and record whether the architecture **meets, partially meets, or misses** it, with evidence. An NFR the design cannot demonstrate is a finding, not a footnote.

### Co-location and latency rules

Apply these as **strong defaults** — deviations need an explicit, stated justification in the design, otherwise they are findings:

- **Same cloud:** a workload and its database/primary data stores belong in the same cloud. A database in one cloud serving compute in another (e.g., app in Azure querying a database in AWS) is a High finding by default: per-request cross-cloud latency, double egress charges, two security perimeters, and two failure domains in the request path.
- **Same region:** chatty, synchronous dependencies (app ↔ database, app ↔ cache, service ↔ service on the request path) belong in the same region. Cross-region synchronous calls add tens of milliseconds per hop and multiply under N+1 query patterns — call out the estimated impact.
- **Cross-region/cloud is acceptable when justified:** async replication for DR, event/batch integration, data residency mandates, edge delivery via CDN. Verify the integration is genuinely async/buffered (queue, replication, scheduled transfer) — a "DR" link that is actually a synchronous dependency is a finding.

### Integration fit

Scrutinize every integration between components and flag poor fits:

- Synchronous request/response used where the consumer tolerates delay (should be queue/event-driven).
- Chatty fine-grained calls across any network boundary (region, cloud, on-prem ↔ cloud) — latency and cost multiply per hop.
- A service consumed far from where its data lives (e.g., analytics queries pulled cross-cloud instead of replicating the data once).
- Protocol/pattern mismatches: polling where webhooks/events exist, file-drop integration on a latency-sensitive path, direct database integration between services instead of an API/event contract.
- SaaS or third-party dependencies on the synchronous critical path with no timeout, retry, or fallback shown.

## Step 4: Select the assessment framework

Match the framework to the provider:

| Provider | Framework | Reference file |
|---|---|---|
| AWS | AWS Well-Architected Framework (6 pillars) | [references/aws-well-architected.md](references/aws-well-architected.md) |
| Azure | Azure Well-Architected Framework (5 pillars) | [references/azure-well-architected.md](references/azure-well-architected.md) |
| GCP | Google Cloud Architecture Framework | [references/gcp-architecture-framework.md](references/gcp-architecture-framework.md) |
| Multi-cloud / hybrid | Assess each provider's workloads against its own framework, plus cross-cloud concerns (egress cost, identity federation, consistent policy) |
| Provider-agnostic / on-prem-to-cloud | Use the AWS pillar structure as the default lens; note it as an assumption |

Read the matching reference file before assessing. Also apply the cross-cutting checklist in [references/review-checklist.md](references/review-checklist.md).

**Regulatory overlays:** if the workload belongs to an APRA-regulated entity (Australian bank/ADI, insurer, or super fund) or the user mentions APRA/CPS 230/CPS 234, additionally apply [references/apra-cps230-cps234.md](references/apra-cps230-cps234.md) and include its Regulatory Compliance section in the report.

## Step 5: Research current best practices

Use web search to verify guidance that changes over time — do not rely on memory for:

- Current provider recommendations for the specific services in the architecture (e.g., latest guidance on a database engine, gateway type, or serverless pattern).
- Deprecated or superseded services appearing in the architecture (flag these explicitly).
- Relevant compliance baselines if the user mentions an industry: CIS Benchmarks, NIST 800-53/CSF, PCI DSS, HIPAA, GDPR, ISO 27001.
- Published reference architectures for the same workload pattern, to compare against.

Cite sources for any claim based on web research.

## Step 6: Assess against each pillar

For every pillar in the selected framework, evaluate the architecture and record findings. Each finding must include:

- **What** — the specific gap or risk, tied to a named component or its absence.
- **Why it matters** — concrete failure mode or cost (not generic advice).
- **Framework reference** — which pillar/principle it violates.
- **Severity** — using this scale:
  - **Critical** — likely data loss, breach, or extended outage; fix before production.
  - **High** — significant risk or cost with realistic triggers; fix within the next cycle.
  - **Medium** — best-practice deviation with moderate impact; plan remediation.
  - **Low** — improvement opportunity or hygiene item.
- **Recommendation** — the specific change, naming the provider service or pattern to adopt, with rough effort (Low/Medium/High).

Also record **strengths** — things the architecture does well. A review that only lists faults is less credible and less useful.

Ground every finding in what is actually present (or demonstrably absent) in the supplied architecture. Never invent components. If something might exist but isn't shown, phrase it as a verification question, not a finding.

## Step 7: Produce the report

Output the review in this structure:

```markdown
# Architecture Review: <workload name>

## Summary
2–4 sentences: overall posture, the dominant risks, and the top 3 actions.

## Architecture Understood
The component inventory from Step 2 (so the user can correct misreadings).
State assumptions made.

## NFR Compliance
| Requirement | Source (stated/assumed) | Status (Meets / Partial / Misses) | Evidence |
|---|---|---|---|
Include co-location and latency violations here and as findings.

## Scorecard
| Pillar | Rating | Critical | High | Medium | Low |
|---|---|---|---|---|---|
Rating scale: Strong / Adequate / Needs Attention / At Risk

## Strengths
What the architecture gets right, with pillar references.

## Findings & Recommendations
Grouped by pillar, ordered by severity. Each finding in the Step 6 format.
Include a "Latency & Integration Fit" group for Step 3 findings that don't map cleanly to a pillar.

## Prioritized Action Plan
A numbered list ordered by severity then effort — quick critical/high wins first.

## Open Questions
Information needed to confirm or close provisional findings.

## References
Framework documents and any web sources cited.
```

Keep the report proportionate: a simple 5-component architecture warrants a shorter review than an enterprise landing zone. Do not pad pillars that have no meaningful findings — say "No significant findings" and move on.
