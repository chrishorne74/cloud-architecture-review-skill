# Landing Zone & FinOps Maturity Overlay

Apply this overlay when the input is platform-scale rather than workload-scale: multi-account/multi-subscription diagrams, landing zones, "cloud foundation" designs, or anything showing org structure. Workload pillars still apply, but the questions below are what distinguish a good landing zone.

## Organization structure

- AWS: Organizations with OUs separating security, infrastructure, workloads, sandbox; prod/non-prod split. Azure: management group hierarchy (platform / landing zones / sandbox / decommissioned). GCP: organization → folders → projects.
- Dedicated accounts/subscriptions/projects for: security tooling, log archive, shared network, and per-workload-environment. One big shared account/subscription for many workloads is a High finding at this scale.
- Account/subscription **vending**: automated, IaC-driven provisioning with baseline applied (AWS Control Tower / Account Factory, Azure subscription vending, GCP project factory). Manual creation is an Operational Excellence finding.

## Guardrails (policy-as-code)

- Preventative: SCPs / Azure Policy deny + deployIfNotExists / GCP organization policy constraints. Minimum expected set: deny disabling audit logging, deny public storage, restrict regions, deny IAM user key creation (AWS) / require managed identities (Azure) / restrict service account key creation (GCP).
- Detective: Config/Defender for Cloud/SCC posture management feeding a central security account.
- Guardrails managed as code in a pipeline, not hand-edited in the console.

## Identity

- Single workforce identity source federated in (Identity Center/Entra/Cloud Identity); no per-account local users.
- **Break-glass**: documented emergency access accounts, excluded from Conditional Access lockout, monitored for use. Absence is a High finding — most landing-zone diagrams omit it.
- Privileged access is just-in-time (PIM / Identity Center permission sets with short sessions), not standing admin.

## Network platform

- Hub-spoke or equivalent (Transit Gateway, vWAN, Shared VPC) with centralized egress/inspection where the security model requires it.
- IPAM: non-overlapping CIDR plan with room to grow — overlapping ranges block future peering/hybrid and are expensive to fix later.
- Hybrid connectivity (DX/ExpressRoute/Interconnect) redundant if on the critical path.
- DNS strategy: central private zones, conditional forwarding to on-prem.

## Central observability & security operations

- Log archive account/subscription: org-wide audit trails (CloudTrail/Activity Log/Audit Logs), immutable retention.
- Security findings aggregated to one place with an owner and a triage process (not just enabled-and-ignored).

## FinOps maturity

Rate the design Crawl / Walk / Run and report the level:

| Level | Signals |
|---|---|
| **Crawl** | Tagging/labeling standard exists and is enforced by policy (untagged = noncompliant or denied); budgets + alerts per account/subscription; billing data flowing to someone who looks at it |
| **Walk** | Showback/chargeback by tag/account to workload owners; centralized commitment management (Savings Plans/RIs/CUDs bought at org level); anomaly detection enabled; scheduled off-hours shutdown for non-prod |
| **Run** | Unit economics (cost per transaction/customer) tracked; rightsizing recommendations actioned on a cadence with measured outcome; cost as a gate in architecture review and CI (infracost or similar) |

A landing zone with no tagging enforcement and no budget alerts hasn't reached Crawl — flag it as a Cost finding (High at enterprise scale), since retrofitting allocation onto untagged estates is notoriously expensive.

## Reporting

Add a **Platform Maturity** subsection under Findings when this overlay applies, and include the FinOps level in the Summary. Don't penalize a workload-level diagram for omitting landing-zone concerns — apply this overlay only when the input claims platform scope.
