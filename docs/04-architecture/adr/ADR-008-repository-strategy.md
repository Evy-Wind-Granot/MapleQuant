# ADR-008: Repository Strategy — Public Engine Code, Private Business Docs

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-10-27 |
| **Deciders** | Founder |

## Context

Two content classes exist: (1) the engine code — secrets-free by design
(FR-CF-03, gitleaks-enforced), with free CI at stake; (2) business documents
(this set) containing strategy specifics, financial planning, and personal
operational details. A future open-source option (P4 model 2) depends on the
code being releasable without archaeology.

## Decision

Two repositories:

1. **`maplequant-engine` (public, from day one):** platform code + technical
   docs. No secrets ever (CI-enforced). Free GitHub Actions minutes; public
   accountability; portfolio value; forces the secret hygiene the security
   docs demand anyway.
2. **`maplequant-docs` (private):** this business/planning/regulatory document
   set, personal metrics, and anything strategy-competitive. Any future
   open-sourcing of docs is a **curated, deliberate copy** (OQ-07), never a
   visibility flip.

## Consequences

**Positive:** free CI; external scrutiny of code quality early; no risk of
leaking business docs by making a repo public later; clean history from day
one.

**Negative:** public code requires politeness (issues, basic README) — small
ongoing tax; a public repo reveals strategy family existence (accepted:
momentum-rotation is not proprietary alpha, and the edge was never secrecy —
see [Competitive Landscape §3](../../01-market-compliance/02-competitive-landscape.md)).

**Mitigations:** security-by-design makes code exposure safe (SEC-01…05);
trade-off reviewed at P4 alongside the commercialization decision.

## Alternatives Considered

| Alternative | Rejected because |
|---|---|
| One private repo | No free CI minutes for private repos at scale (and weaker hygiene pressure); visibility flip later is riskier than day-one public |
| Everything public | Business plans, personal financials, and strategy-competitive research don't belong in the open; irreversible |
| Everything private + paid CI | Breaks NFR-24 |
