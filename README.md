# Cloud Architecture Review — GitHub Copilot Agent Skill

A GitHub Copilot [agent skill](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) that reviews cloud architectures against the **AWS, Azure, and GCP Well-Architected Frameworks**, cloud best practices, and industry guidelines (CIS, NIST, Zero Trust, PCI/HIPAA/GDPR where relevant).

Give it an architecture as a **diagram attachment**, **document**, **Infrastructure-as-Code**, **URL**, or plain **description**, and it produces:

- A component inventory (so you can correct any misreading)
- **NFR compliance check** — stated or inferred non-functional requirements traced through the design
- **Co-location & latency analysis** — flags cross-cloud app↔database splits, cross-region synchronous calls, and poor-fit integrations
- A pillar-by-pillar scorecard
- Severity-rated findings (Critical/High/Medium/Low) with specific, effort-rated recommendations
- A prioritized action plan and open questions

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
    └── review-checklist.md
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
- **Evidence over vibes**: every finding ties to a named component or a demonstrable absence; unknowns become open questions, not assumed findings.
- **Current guidance**: the skill instructs the agent to verify time-sensitive recommendations and deprecated services via web search rather than relying on model memory.
- **Strengths included**: reviews report what the architecture gets right, not just faults.

## License

MIT
