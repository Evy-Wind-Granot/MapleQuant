# Requirements Traceability Matrix (RTM)

| | |
|---|---|
| **Document ID** | DOC-015 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [Functional](02-functional-requirements.md) · [NFR](03-non-functional-requirements.md) · [Security](04-security-requirements.md) · [Operational](05-operational-requirements.md) |

## 1. How to read this matrix

- One row per **Must** requirement (individual) or Should/Could (grouped).
- **Verified by** = the artifact that proves implementation (test, gate, drill,
  or doc). A requirement is *Implemented* only when its verification artifact
  passes in CI/ops and is recorded here.
- This is a **living document**: updated at every release and stage gate.
  Status legend: ☐ Planned · ◐ In progress · ☑ Verified.

## 2. Functional Requirements (Must items)

| ID | Requirement (abbrev.) | Verified by | Gate | Status |
|---|---|---|---|---|
| FR-DA-01 | EOD OHLCV ingestion | IT (fixtures) | v0.1 | ☐ |
| FR-DA-02 | Ingest validation | UT + IT | v0.1 | ☐ |
| FR-DA-03 | Staleness/gap fail-closed | UT | v0.1 | ☐ |
| FR-DA-04 | Idempotent upserts | UT | v0.1 | ☐ |
| FR-DA-05 | BoC FX ingestion | IT | v0.1 | ☐ |
| FR-DA-06 | Exchange calendar logic | UT (holidays) | v0.1 | ☐ |
| FR-DA-09 | Backtest data snapshots | IT | v0.2 | ☐ |
| FR-ST-01 | Strategy plug-in interface | UT | v0.2 | ☐ |
| FR-ST-02 | Determinism (no hidden inputs) | UT (double replay) | v0.2 | ☐ |
| FR-ST-03 | MQ-MOM-1 reference strategy | BT-gate | **G1** | ☐ |
| FR-ST-04 | Signal rationale logging | UT | v0.2 | ☐ |
| FR-ST-06 | Replay equivalence live=BT | IT (CI golden) | **G1** | ☐ |
| FR-BT-01 | Same-core backtest | IT | v0.2 | ☐ |
| FR-BT-02 | Cost model | UT | v0.2 | ☐ |
| FR-BT-03 | No-lookahead structural | UT (property) | v0.2 | ☐ |
| FR-BT-04 | Point-in-time universe | UT | v0.2 | ☐ |
| FR-BT-05 | Metrics suite | UT (golden) | v0.2 | ☐ |
| FR-BT-06 | Walk-forward runner | IT | **G1** | ☐ |
| FR-BT-07 | Parameter sensitivity | IT | **G1** | ☐ |
| FR-BT-08 | Experiment artifacts | IT | v0.2 | ☐ |
| FR-EX-01 | Adapter-only order flow | IT (contract suite) | v0.4 | ☐ |
| FR-EX-02 | Idempotent orders | UT (fault injection) | v0.4 | ☐ |
| FR-EX-03 | Uncertain-state query-first | UT (fault injection) | v0.4 | ☐ |
| FR-EX-04 | Unknown-state worst case | UT | v0.4 | ☐ |
| FR-EX-05 | Hours gate + rate limits | UT + IT | v0.4 | ☐ |
| FR-EX-06 | Error taxonomy | UT | v0.4 | ☐ |
| FR-EX-08 | Live arm step | IT | **G3** | ☐ |
| FR-RK-01 | Pre-trade gate (all caps) | UT (property) | v0.4 | ☐ |
| FR-RK-02 | Daily loss breaker | UT | v0.4 | ☐ |
| FR-RK-03 | Drawdown breaker | UT + IT | v0.4 | ☐ |
| FR-RK-04 | Kill switch (3 paths) | IT + DR drill | **G2** | ☐ |
| FR-RK-05 | No auto-resume | UT | v0.4 | ☐ |
| FR-RK-06 | Code-enforced maxima | UT | v0.4 | ☐ |
| FR-RK-07 | Freshness gate | UT | v0.4 | ☐ |
| FR-RK-09 | Fail-closed on break/outage | IT | v0.4 | ☐ |
| FR-RC-01 | Pre-session reconciliation | IT | v0.4 | ☐ |
| FR-RC-02 | EOD reconciliation + hold | IT | v0.4 | ☐ |
| FR-RC-03 | Whitelisted auto-repair | UT | v0.4 | ☐ |
| FR-RC-04 | Reconciliation logging | UT | v0.4 | ☐ |
| FR-PF-01 | Book of record | UT | v0.4 | ☐ |
| FR-PF-02 | P&L computation | UT (golden) | v0.4 | ☐ |
| FR-PF-03 | ACB per lot | UT (fixtures) | v0.4 | ☐ |
| FR-PF-04 | Superficial-loss flags | UT | v0.4 | ☐ |
| FR-PF-05 | CAD base translation | UT | v0.4 | ☐ |
| FR-NT-01 | Daily digest | IT | v0.4 | ☐ |
| FR-NT-02 | Immediate alerts | IT | v0.4 | ☐ |
| FR-NT-03 | Authenticated Telegram cmds | IT | v0.4 | ☐ |
| FR-NT-04 | Alerts never block trading | UT | v0.4 | ☐ |
| FR-CF-01 | Strict config validation | UT | v0.1 | ☐ |
| FR-CF-02 | Mode selection | IT | v0.4 | ☐ |
| FR-CF-03 | Secrets outside config | UT + gitleaks | v0.1 | ☐ |
| FR-CF-04 | Admin CLI | IT (smoke) | v0.1 | ☐ |
| FR-CF-05 | Config hash stamping | UT | v0.2 | ☐ |
| FR-TX-01 | Tax export (golden) | IT | v0.5 | ☐ |
| FR-TX-02 | CRA mapping doc | DOC | v0.5 | ☐ |
| FR-AU-01 | Append-only audit log | UT | v0.2 | ☐ |
| FR-AU-02 | Order lineage | UT | v0.4 | ☐ |
| FR-AU-03 | Hash-chained entries | UT | v0.2 | ☐ |
| US-103/502 etc. | PRD acceptance criteria | [PRD §3](../02-product/02-product-requirements-document.md) | per story | ☐ |

*Should/Could items (FR-DA-07/08, FR-ST-05, FR-BT-09, FR-EX-07, FR-RK-08,
FR-PF-06, FR-NT-05, FR-CF-06, FR-AU-04) are tracked in the FR doc; they enter
this matrix when promoted to Must.*

## 3. Non-Functional Requirements

| ID | Verification artifact | Status |
|---|---|---|
| NFR-01/02/03 (latency) | MON dashboards + UT | ☐ |
| NFR-04/05 (reliability) | MON: session success rate ≥ 98%, heartbeat alerts | ☐ |
| NFR-06 (crash safety) | UT fault-injection suite | ☐ |
| NFR-07 (solo maintainability) | CI coverage gate ≥ 85% core; ops-time log | ☐ |
| NFR-08 (durability/RPO) | DR drill records | ☐ |
| NFR-09–11 (capacity) | Capacity review (OPS-21) | ☐ |
| NFR-12–14 (observability/audit) | IT + integrity check tool | ☐ |
| NFR-15–17 (portability/rollback) | RB-008 drill + deploy log | ☐ |
| NFR-18–20 (usability) | Runbook review (OPS-17) | ☐ |
| NFR-21–23 (localization) | UT fixtures (calendar, FX, ACB) | ☐ |
| NFR-24 ($0 cost) | Monthly cost review (OPS-13) | ☐ |

## 4. Security & Operational Requirements

| Group | Verification | Status |
|---|---|---|
| SEC-01–06 (credentials) | Host audit, credential register, gitleaks CI, rotation log | ☐ |
| SEC-07–10 (host/network) | Quarterly host audit checklist | ☐ |
| SEC-11–16 (app security) | UT/IT + bandit CI + design review | ☐ |
| SEC-17–20 (supply chain) | CI: hashed lockfile, pip-audit, PR justifications | ☐ |
| SEC-21–25 (detection/response) | Drills (revocation, kill switch) + MON | ☐ |
| OPS-01–05 (environments/deploy) | Host config audit + deploy log | ☐ |
| OPS-06–09 (monitoring) | MON + weekly ops log | ☐ |
| OPS-10–12 (backup/DR) | Backup logs + quarterly restore tests | ☐ |
| OPS-13–14 (cost) | Monthly review, free-tier inventory | ☐ |
| OPS-15–18 (change/incidents) | Repo settings, postmortem log, drill records | ☐ |
| OPS-19–22 (hygiene) | Quarterly checklist | ☐ |

## 5. Stage-Gate Cross-Check

| Gate | Key requirement dependencies |
|---|---|
| **G0** | Docs complete (this set); FR-CF-01/03/04; FR-DA-01…06 |
| **G1** | FR-ST-03/06; FR-BT-01…08; [Backtesting Validation Standards](../07-quality/02-backtesting-validation-standards.md) report |
| **G2** | All FR-EX, FR-RK, FR-RC Musts verified in paper; SEC-21/23 drills; OPS-06/08/10/11 running |
| **G3** | FR-EX-08; OPS-05; live credential register + revocation drill; capital cap configured |
| **G4** | OQ-07 decision pack: market analysis refresh, counsel review, licensing review |
