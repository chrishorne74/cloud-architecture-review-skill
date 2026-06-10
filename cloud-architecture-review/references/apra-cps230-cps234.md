# APRA CPS 230 & CPS 234 — Review Lens

Apply this lens when the workload belongs to an APRA-regulated entity: Australian banks/ADIs, insurers, or superannuation (RSE) licensees — or when the user mentions APRA, CPS 230, or CPS 234. These are enforceable prudential standards, so map findings to specific requirements and rate severity accordingly (a breach candidate is High or Critical, not Medium).

Status: CPS 234 in force since 1 July 2019. CPS 230 in force since 1 July 2025; transition for pre-existing service provider arrangements ends 1 July 2026, so treat CPS 230 as fully applicable to everything reviewed. Verify current amendments via web search (targeted amendments were issued April 2026).

## CPS 234 — Information Security

Architecture-checkable requirements:

### Information asset classification
- [ ] Information assets are identified and classified by **criticality and sensitivity**. A design with no data classification (or one tier for everything) cannot demonstrate controls "commensurate with criticality" — finding.
- [ ] Controls scale with classification: stronger encryption/key management (CMK), tighter access, and isolation for the most sensitive assets.

### Controls implementation
- [ ] Encryption at rest and in transit on all classified data paths and stores.
- [ ] Least-privilege access with MFA on privileged/human access; privileged access management for production.
- [ ] Network segmentation isolating sensitive workloads; no sensitive data store publicly reachable.
- [ ] Secrets in a managed vault, never in code/config.

### Third-party and cloud (shared responsibility)
- [ ] Information assets managed by third parties (the cloud provider, SaaS, integrators) are identified, and the entity has **assessed the provider's information security capability** — for cloud this means consuming provider assurance (SOC 2, ISO 27001, IRAP for Australian government-adjacent workloads) and evidencing the customer side of the shared responsibility model.
- [ ] The design doesn't assume the provider covers controls that are customer responsibility (guest OS patching, IAM, data classification, network config).

### Detection, response, and notification
- [ ] Logging and monitoring sufficient to **detect and respond to incidents in a timely manner**: centralized security logs, alerting, retention adequate for investigation.
- [ ] Architecture supports the **72-hour APRA notification** for material security incidents — if the design can't detect a breach, it can't notify; flag missing detection as a CPS 234 finding, not just hygiene.
- [ ] Material control weaknesses must be notifiable within 10 business days — posture tooling (e.g., Security Hub / Defender for Cloud / SCC) supports discovering them.

### Testing and assurance
- [ ] Controls are **systematically tested**; nothing in the architecture prevents penetration testing or control validation in production-like environments.
- [ ] Test environments don't hold unprotected production data (classification follows the data).

## CPS 230 — Operational Risk Management

Architecture-checkable requirements:

### Critical operations and tolerance levels
- [ ] Critical operations supported by this workload are identified, with board-approved **tolerance levels**: maximum period of disruption (maps to RTO), maximum data loss (maps to RPO), and minimum service level during disruption.
- [ ] Trace each tolerance through the design: does the HA/DR architecture **demonstrably** meet the stated maximum outage and data loss? A multi-AZ design with async cross-region replication cannot honestly claim RPO=0 across regions — check claims against mechanisms.
- [ ] If no tolerance levels are stated, list them as required inputs (Open Questions) and assess against inferred values.

### Business continuity
- [ ] BCP/DR exists and is **tested with severe but plausible scenarios** — region loss, provider service failure, ransomware/data corruption (backups must be isolated and immutable to survive the last one).
- [ ] Failover is executable within tolerance: automated or well-rehearsed, not a diagram-only arrow.

### Service provider management
- [ ] The cloud provider (and any material SaaS in the critical path) is a **material service provider**: belongs on the register, with due diligence, and contracts providing access/audit rights and APRA's right to information.
- [ ] **Exit/transition strategy** exists for material providers — portability of data and workloads is an architectural property: proprietary lock-in with no documented exit path is a CPS 230 finding for critical operations.
- [ ] **Concentration risk**: critical operation entirely dependent on one provider/region with no mitigation is worth flagging; fourth-party risk (the SaaS provider's own cloud) noted where visible.
- [ ] **Offshoring/data location**: data or operations hosted outside Australia for critical operations requires APRA notification and risk assessment — check chosen regions and support arrangements; flag offshore data residency as a verification question if not addressed.

### Technology health
- [ ] CPS 230 explicitly requires monitoring the **age and health of information assets** — end-of-life OSes, unsupported runtimes, deprecated services in the architecture are CPS 230 findings (not just hygiene), severity rising with criticality.
- [ ] IT capability supports current and projected business requirements — capacity/scaling design for critical operations.

## Reporting

When this lens applies, add a **Regulatory Compliance (APRA)** section to the report between NFR Compliance and the Scorecard:

| Requirement | Standard | Status (Meets / Partial / Misses / Unknown) | Evidence / Gap |
|---|---|---|---|

Use "Unknown" liberally — many CPS obligations are organizational, not architectural; only assess what the supplied architecture can evidence, and route the rest to Open Questions.
