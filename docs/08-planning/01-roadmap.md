# Roadmap

| | |
|---|---|
| **Document ID** | DOC-036 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [Business Plan](../00-executive/03-business-plan.md) · [RTM stage gates](../03-requirements/07-requirements-traceability-matrix.md) · [Validation Standards](../07-quality/02-backtesting-validation-standards.md) · [Testing §6](../07-quality/01-testing-strategy.md) |

> **Slip rule:** dates are targets; **gates are hard**. A missed gate slips the
> calendar — it never lowers the bar. Nothing ships live without an official
> broker API (RL-5): if Wealthsimple access hasn't landed by G3 time, **live
> waits**; paper keeps running.

## Phase Overview

| Phase | Window | Objective | Exit gate |
|---|---|---|---|
| **P0 — Foundation** | 2026-10 → 2026-11 | Docs baseline, repo, data layer, CLI skeleton | **G0** |
| **P1 — Research** | 2026-11 → 2026-12 | Backtest engine + strategy v1 validated | **G1** |
| **P2 — Validation** | 2027-01 → 2027-04 | 60 trading days of clean paper trading | **G2** |
| **P3 — Live (capped)** | 2027-04 → 2027-07 | CAD 2,000 live, 90 days supervised | **G3 entry → supervision exit** |
| **P4 — Decision** | 2027-Q4 | Commercialize / open-source / hold | **G4** |

## P0 — Foundation (target exit: 2026-11)

**Deliverables**

1. This documentation set baselined (G0 evidence).
2. Public engine repo + private docs repo standing ([ADR-008](../04-architecture/adr/ADR-008-repository-strategy.md)); CI gates green on a skeleton.
3. Data layer: ingestion, validation, storage, calendar, FX (FR-DA-01…06) with contract tests.
4. Config framework: strict schema validation (FR-CF-01), secrets plumbing (SEC-01/03), `mq` CLI skeleton (FR-CF-04).

**G0 exit criteria:** all G0 rows in the [RTM](../03-requirements/07-requirements-traceability-matrix.md) verified; gitleaks + CI gates enforced; data backfilled with clean provenance for the full research window (2016-10 →).

## P1 — Research (target exit: 2026-12)

**Deliverables**

1. Event-driven core + SimBroker ([ADR-003](../04-architecture/adr/ADR-003-single-codepath-event-driven-core.md); FR-BT-01…08) with replay-equivalence golden test in CI.
2. MQ-MOM-1 implemented (FR-ST-01…06).
3. Walk-forward, sensitivity, and experiment-tracking runners.
4. **Validation report** per [Backtesting Validation Standards](../07-quality/02-backtesting-validation-standards.md).

**G1 exit criteria:** every §5 acceptance threshold passed on aggregate OOS; holdout run recorded against the frozen universe; 48-hour self-review completed; frozen triple (parameters + universe + cost model) committed. **A fail at G1 is a legitimate outcome** — capital simply stays in research/cash.

## P2 — Validation (target exit: 2027-04)

**Deliverables**

1. Alpaca paper adapter certified ([Adapter spec §5](../04-architecture/03-broker-adapter-specification.md)).
2. Risk manager, reconciliation, accounting, notifier, kill switch — all Musts verified in paper (RTM v0.4 rows).
3. Host + monitoring + heartbeat + backups running ([Deployment](../06-operations/01-deployment-and-environments.md), [Monitoring](../06-operations/02-monitoring-and-alerting.md)).
4. Wealthsimple Trade API application submitted (CT-01) and status verified (OQ-01).

**G2 exit criteria** ([Testing §6](../07-quality/01-testing-strategy.md)): 60 consecutive clean trading days · ≥ 98% scheduled runs · slippage ≤ 10 bps beyond model · zero unresolved reconciliation breaks · kill-switch, revocation, and DR drills passed · coverage ≥ 85% core.

## P3 — Live, Capped (2027-04 → 2027-07)

**Entry (G3):** live adapter certified on sandbox endpoints; trade-only scoped key issued (SEC-02); broker ToS reviewed and archived (CT-02); capital cap configured; **deliberate `mq arm live`** (FR-EX-08).

**Operation:** CAD 2,000 cap; daily supervision for the first 90 days (digest review + weekly ops review); monthly full metrics review vs. the frozen backtest expectation.

**Supervision exit:** 90 days with no SEV1, no unresolved SEV2, reconciliation clean, and live behavior within the drift bands of the G1 report → eligibility to propose the first capital step-up (2k → 5k → 10k, each step requiring ≥ 2 clean quarters — [Business Plan §7.2](../00-executive/03-business-plan.md)).

## P4 — Decision (2027-Q4)

**Decision pack contents:** refreshed [Market Analysis](../01-market-compliance/01-market-analysis.md) · 12 months of live/paper evidence · licensing review ([Data Req §7](../03-requirements/06-data-requirements.md)) · name/trademark search result (OQ-04) · counsel scoping for models 2–4 ([Regulatory §4](../01-market-compliance/03-regulatory-compliance.md)) · candid recommendation: **hold is a valid outcome**.

## Operating Cadence (all phases)

| Rhythm | Activity |
|---|---|
| Daily | Digest scan (≤ 30 s) |
| Weekly | Ops review (≤ 30 min): alerts, metrics, reconciliation, costs, next week |
| Monthly | Cost review ([Cost §3](../06-operations/05-cost-management.md)) · dependency updates (SEC-20) |
| Quarterly | DR drill · runbook review (OPS-17) · risk register review · doc freshness pass |
| At each gate | RTM update · doc versions bumped · stage-gate decision recorded |

## Scope Discipline

The roadmap adds nothing mid-phase. New strategy ideas, broker integrations,
and features go to the backlog (and [Open Questions](../09-appendix/03-open-questions.md))
and wait for a gate boundary. **Boredom during P2 paper is the plan working.**
