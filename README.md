# Cloud Architecture Review — GitHub Copilot Agent Skill

A GitHub Copilot [agent skill](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) that reviews cloud architectures against the **AWS, Azure, and GCP Well-Architected Frameworks**, cloud best practices, and industry guidelines (CIS, NIST, Zero Trust, PCI/HIPAA/GDPR where relevant). For APRA-regulated entities (Australian banks, insurers, super funds) it also checks the design against **CPS 230** (operational risk: tolerance levels, BCP, service provider/exit strategy, tech health) and **CPS 234** (information security: classification, controls, detection supporting 72-hour notification).

Give it an architecture as a **diagram attachment**, **document**, **Infrastructure-as-Code**, **URL**, or plain **description**, and it produces:

- A component inventory (so you can correct any misreading)
- A **principles compliance table** — ten house principles (P1–P10: same-cloud/same-region co-location, multi-AZ as standard, async across boundaries, right-fit integration, evidence-based resiliency, security baseline, operability, cost-aware data flow), each rated Compliant / Justified deviation / Violation
- **NFR compliance check** — stated or inferred non-functional requirements traced through the design
- **Co-location & latency analysis** — flags cross-cloud app↔database splits, cross-region synchronous calls, and poor-fit integrations
- A pillar-by-pillar scorecard
- Severity-rated findings (Critical/High/Medium/Low) with specific, effort-rated recommendations
- A prioritized action plan and open questions

Additional lenses applied automatically when relevant: **Terraform-aware review** (resource-level signals, runs checkov/tfsec if installed), **data-transfer cost estimates** (per-TB numbers behind egress/cross-boundary findings), and a **landing zone overlay** with FinOps maturity rating (Crawl/Walk/Run) for platform-scale designs.

## Install

### Per repository (shared with your team)

Copy the skill folder into the repo:

```
.github/skills/cloud-architecture-review/
├── SKILL.md
└── references/
    ├── aws-well-architected.md
    ├── azure-well-architected.md
    ├── gcp-architecture-framework.md
    ├── review-checklist.md
    ├── apra-cps230-cps234.md      # APRA regulatory overlay
    ├── terraform-review.md        # IaC extraction signals + checkov/tfsec
    ├── cost-gravity.md            # indicative data-transfer cost table
    ├── landing-zone-review.md     # platform-scale + FinOps maturity overlay
    └── example-review.md          # worked example for calibration
```

### Personal (all your projects)

Copy `cloud-architecture-review/` into `~/.copilot/skills/`.

Works with Copilot coding agent, Copilot code review, Copilot CLI, and agent mode in VS Code.

## Usage

In Copilot Chat (agent mode) or the Copilot CLI:

> Review this architecture against the Well-Architected Framework *(attach diagram or paste URL)*

> Find the gaps in the design described in docs/architecture.md — our RTO is 1 hour and users are in EU and US.

> Assess infra/main.tf against AWS best practices and our latency NFRs.

## Design choices baked into the skill

- **Strong co-location defaults**: workloads and their databases belong in the same cloud and region; cross-boundary synchronous dependencies are findings unless explicitly justified (DR, data residency, edge).
- **Resiliency = availability + DR, with multi-AZ as the standard for both**: single-AZ production is a finding; cross-region or cross-cloud DR is non-standard and needs explicit justification (regulation, RTO/RPO surviving region loss, residency) — unjustified multi-region is flagged as over-engineering.
- **Evidence over vibes**: every finding ties to a named component or a demonstrable absence; unknowns become open questions, not assumed findings.
- **Current guidance**: the skill instructs the agent to verify time-sensitive recommendations and deprecated services via web search rather than relying on model memory.
- **Strengths included**: reviews report what the architecture gets right, not just faults.

## License

MIT
