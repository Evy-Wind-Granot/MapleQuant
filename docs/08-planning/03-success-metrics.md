# Success Metrics

| | |
|---|---|
| **Document ID** | DOC-038 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [Validation Standards](../07-quality/02-backtesting-validation-standards.md) · [Monitoring catalog](../06-operations/02-monitoring-and-alerting.md) · [Cost policy](../06-operations/05-cost-management.md) · [NFR targets](../03-requirements/03-non-functional-requirements.md) |

## 1. Principle

Early on, returns are statistically meaningless (small sample, fat tails) — but
**process quality is fully measurable from day one**. So the metric program
measures *discipline and implementation fidelity* first, and treats returns as
a long-sample question. Success = the system does what its research said it
would do, safely, at $0, with no surprises.

## 2. Research Gates (measured at G1 — pass/fail)

| Metric | Target | Source |
|---|---|---|
| Aggregate OOS Sharpe | ≥ 0.7 | [Validation Standards §5](../07-quality/02-backtesting-validation-standards.md) |
| OOS max drawdown | ≤ 20% | §5 |
| Positive walk-forward windows | ≥ 60% | §5 |
| Net CAGR (after all modeled costs) | > 0 | §5 |
| Parameter stability (±20%) | No accept/reject flips | §6 |
| Holdout degradation | Within headroom rule | §5 |
| Replay equivalence (live=BT) | 100% on golden tests | FR-ST-06 |

## 3. Paper Gates (measured at G2 — pass/fail)

| Metric | Target |
|---|---|
| Clean consecutive trading days | **60** |
| Scheduled-run success | ≥ 98% (NFR-04) |
| Missed-run detection | ≤ 15 min (NFR-05) |
| Realized slippage vs. model | ≤ 10 bps average |
| Unresolved reconciliation breaks | **0** |
| Drills (kill/revocation/DR) | All passed, within time budgets |
| Core coverage | ≥ 85% |

## 4. Live Operational KPIs (P3+, continuous)

| KPI | Target | Cadence |
|---|---|---|
| SEV1 incidents | **0** | per quarter |
| SEV2 incidents | ≤ 1 per quarter | per quarter |
| Scheduled-run success | ≥ 98% | monthly roll-up |
| Reconciliation breaks unresolved > 1 day | 0 | weekly |
| Fixed cost | **$0.00/month** (NFR-24) | monthly |
| Operator time | ≤ 30 min/week steady-state (NFR-07) | monthly estimate |
| Backup + restore tests | 100% on schedule, restore verified | quarterly |
| Alert quality (actionable, deduplicated) | Zero ignored-alert fatigue; every alert acknowledged in log | weekly |

## 5. Live Investment Tracking (P3+, monitored — **not judged** for 12 months)

Tracked against the **frozen G1 report** (the "backtest said" baseline):

| Track | Question it answers |
|---|---|
| Realized slippage vs. model | Is execution fidelity holding? (Gate: ≤ 10 bps avg) |
| Return vs. OOS expectation band | Is live within the distribution the research promised? |
| Drawdown vs. backtest path | Are we inside the Monte Carlo drawdown envelope? |
| Turnover & cost realization | Are churn bands and FX assumptions real? |
| Breaker trips | Zero expected; each trip = investigate, not shrug |

**Explicitly not a KPI in year one:** absolute return vs. SPY. With ~12 monthly
observations, that comparison is noise wearing a suit. The 12-month review (at
P4) is the first legitimate point to look at realized performance as evidence.

## 6. Venture KPIs

| KPI | Target |
|---|---|
| Stage gates hit on target dates (or slipped honestly) | 100% — no gate ever lowered |
| Documentation currency | All controlled docs reviewed within their cadence |
| Experiment log health | Every conclusion cites run IDs |
| Open questions burn-down | OQ-01…05 resolved before their needed-by dates |

## 7. What Failure Looks Like (pre-committed stop conditions)

Deciding stop rules *now*, in calm conditions, is the whole point:

| Signal | Pre-committed response |
|---|---|
| Any SEV1 | Full stop, RB-003, postmortem, re-arm only via deliberate review |
| 2+ SEV1s in a quarter | Live trading suspended; root-cause program before any re-arm; consider permanent stand-down |
| Live drifts outside the OOS band for a full quarter | Demote to paper; re-run validation before resuming |
| Drawdown breaker trips twice in any rolling 6 months | Demote to paper + review the G1 report honestly (regime change vs. implementation failure) |
| Ops time > 2× budget for a month | Simplify or pause — complexity debt is risk debt |
| Any non-zero invoice | SEV2 process (OPS-13) |
| G1 fails twice on the same strategy family | Family rejected; capital stays defensive; research moves on. **Not trading is a valid output** (Validation §9) |

## 8. Reporting Artifacts

| Artifact | Contains | Audience |
|---|---|---|
| Daily digest | §4/§5 daily items | Operator |
| Weekly review notes | alerts, KPIs, breaks, costs | Operator (logged) |
| Quarterly report | all KPIs vs. targets, risk-register review, doc review, drill records | Operator + future P4 evidence |
| Stage-gate report | gate criteria evidence (RTM links) | Recorded in document control |
