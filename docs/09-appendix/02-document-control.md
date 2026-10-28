# Document Control

| | |
|---|---|
| **Document ID** | DOC-040 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |

## 1. Status Legend

| Status | Meaning |
|---|---|
| **Draft** | Written, not yet reviewed against reality |
| **In Force** | Baseline against which work is executed and reviewed |
| **Under Revision** | Actively being changed; note the open issue |
| **Superseded** | Replaced by another document version (kept for audit) |

All documents begin life as v0.1 Draft. This entire set is **v0.1 (Draft)** as
of 2026-10-27; it becomes **In Force** when the G0 review pass completes.

## 2. Versioning Policy

- `MAJOR.MINOR`: MINOR for clarifications/additions; MAJOR for scope or
  requirement changes (and gate decisions).
- Every controlled doc carries: Document ID, version, owner, date, related links.
- Changes to **Must requirements**, **red lines**, or **invariants** require a
  change note in §4 and a linked issue/ADR.
- Docs are reviewed: at each stage gate (mandatory), quarterly (hygiene), or
  on trigger events (incidents, ADRs).

## 3. Controlled Document Register

| ID | Document | v0.1 status | Review cadence |
|---|---|---|---|
| DOC-001 | [Executive Summary](../00-executive/01-executive-summary.md) | Draft | Each gate |
| DOC-002 | [Vision, Mission & Values](../00-executive/02-vision-mission-values.md) | Draft | G4 |
| DOC-003 | [Business Plan](../00-executive/03-business-plan.md) | Draft | Each gate |
| DOC-004 | [Market Analysis](../01-market-compliance/01-market-analysis.md) | Draft | G4 |
| DOC-005 | [Competitive Landscape](../01-market-compliance/02-competitive-landscape.md) | Draft | G4 |
| DOC-006 | [Regulatory & Compliance](../01-market-compliance/03-regulatory-compliance.md) | Draft | Quarterly + P4 |
| DOC-007 | [Product Overview](../02-product/01-product-overview.md) | Draft | Each gate |
| DOC-008 | [Product Requirements Document](../02-product/02-product-requirements-document.md) | Draft | Each gate |
| DOC-009 | [SRS](../03-requirements/01-software-requirements-specification.md) | Draft | Each gate |
| DOC-010 | [Functional Requirements](../03-requirements/02-functional-requirements.md) | Draft | Each gate |
| DOC-011 | [Non-Functional Requirements](../03-requirements/03-non-functional-requirements.md) | Draft | Each gate |
| DOC-012 | [Security Requirements](../03-requirements/04-security-requirements.md) | Draft | Quarterly + incidents |
| DOC-013 | [Operational Requirements](../03-requirements/05-operational-requirements.md) | Draft | Quarterly |
| DOC-014 | [Data Requirements](../03-requirements/06-data-requirements.md) | Draft | Quarterly + new sources |
| DOC-015 | [RTM](../03-requirements/07-requirements-traceability-matrix.md) | Living | Every release |
| DOC-016 | [System Architecture](../04-architecture/01-system-architecture.md) | Draft | On ADRs |
| DOC-017 | [Tech Stack](../04-architecture/02-tech-stack.md) | Draft | Quarterly |
| DOC-018 | [Broker Adapter Spec](../04-architecture/03-broker-adapter-specification.md) | Draft | On broker changes |
| DOC-019 | [Strategy Engine Design](../04-architecture/04-strategy-engine-design.md) | Draft | G1 |
| DOC-020 | [Risk & Portfolio Design](../04-architecture/05-risk-and-portfolio-design.md) | Draft | Quarterly |
| DOC-021 | [Backtesting Design](../04-architecture/06-backtesting-design.md) | Draft | G1 |
| DOC-022 | [ADR Index](../04-architecture/adr/README.md) | Living | On ADRs |
| DOC-023 | [Security Overview](../05-security/01-security-overview.md) | Draft | Quarterly |
| DOC-024 | [Threat Model](../05-security/02-threat-model.md) | Draft | On triggers (SDLC §5) |
| DOC-025 | [Secrets & Credential Management](../05-security/03-secrets-and-credential-management.md) | Draft | Quarterly + rotations |
| DOC-026 | [Secure SDLC](../05-security/04-secure-development-lifecycle.md) | Draft | Quarterly |
| DOC-027 | [Incident Response Plan](../05-security/05-incident-response-plan.md) | Draft | After incidents + drills |
| DOC-028 | [Deployment & Environments](../06-operations/01-deployment-and-environments.md) | Draft | On infra changes |
| DOC-029 | [Monitoring & Alerting](../06-operations/02-monitoring-and-alerting.md) | Draft | Quarterly |
| DOC-030 | [Backup & Disaster Recovery](../06-operations/03-backup-and-disaster-recovery.md) | Draft | Quarterly + drills |
| DOC-031 | [Runbooks](../06-operations/04-runbooks.md) | Living | Quarterly + postmortems |
| DOC-032 | [Cost Management Policy](../06-operations/05-cost-management.md) | Draft | Monthly (ledger) |
| DOC-033 | [Testing Strategy](../07-quality/01-testing-strategy.md) | Draft | Each gate |
| DOC-034 | [Backtesting Validation Standards](../07-quality/02-backtesting-validation-standards.md) | Draft | G1 + threshold changes |
| DOC-035 | [Release & Change Management](../07-quality/03-release-and-change-management.md) | Draft | Quarterly |
| DOC-036 | [Roadmap](../08-planning/01-roadmap.md) | Living | Monthly check-in |
| DOC-037 | [Risk Register](../08-planning/02-risk-register.md) | Living | Quarterly + incidents |
| DOC-038 | [Success Metrics](../08-planning/03-success-metrics.md) | Draft | Quarterly |
| DOC-039 | [Glossary](01-glossary.md) | Draft | As needed |
| DOC-040 | [Document Control](02-document-control.md) | Living | Continuous |
| DOC-041 | [Open Questions](03-open-questions.md) | Living | Continuous |

## 4. Change Log

| Date | Doc | Version | Change | Reason |
|---|---|---|---|---|
| 2026-10-27 | ALL | 0.1 | Initial documentation set created | G0 baseline draft |
