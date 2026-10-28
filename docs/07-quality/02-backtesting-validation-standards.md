# Backtesting Validation Standards

| | |
|---|---|
| **Document ID** | DOC-034 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [G1 gate](../08-planning/01-roadmap.md) · [Backtesting Design](../04-architecture/06-backtesting-design.md) · [Strategy v1](../04-architecture/04-strategy-engine-design.md) · [Success Metrics](../08-planning/03-success-metrics.md) |

## 1. Authority & Purpose

This document defines **what "validated" means** — the quantitative and
procedural bar a strategy must clear before it may trade real capital (G1
gate). The engine (Backtesting Design) produces results; **this document
decides when results count**. A strategy that fails here does not trade — no
exceptions, no "it's close enough".

## 2. What Counts as a Result

1. Produced by the **core engine** (ADR-003), not a notebook. Notebook findings
   are hypotheses; core runs are results.
2. Fully reproducible: **config hash + data snapshot ID + code version**
   recorded; re-running the same triple yields identical metrics (FR-BT-08).
3. Logged in the **append-only experiment log** with a run ID — including
   failures and dead ends. Conclusions must cite run IDs.
4. Run on the **frozen universe** with point-in-time membership (FR-BT-04).

## 3. Data Windows & Universe Rules

| Window | Definition | Rules |
|---|---|---|
| Full history | 2016-10 → 2026-10 (≥ 10 y, includes 2018 Q4, 2020 COVID crash, 2022 bear, 2024–2025 regime) | Data quality checks pass on every bar used |
| **Research window** | 2016-10 → 2024-10 | All iteration, tuning, and walk-forward happens here |
| **Holdout** | 2024-10 → 2026-10 | Run **once, last, after sign-off of §4–§6**; parameters and universe are frozen before the run; holdout results are reported verbatim |

**Universe freeze:** the ETF universe (incl. defensive asset) is defined,
documented (OQ-06), and committed **before** the holdout run. No symbol
edits after seeing holdout results — a universe change restarts the holdout
clock.

## 4. Walk-Forward Protocol (FR-BT-06)

- Rolling windows: **8-year train → 1-year test**, stepped annually across the
  research window (≥ 8 out-of-sample windows; ≥ 10 preferred).
- The aggregate **out-of-sample** record is the decision object. In-sample
  numbers are reported for reference only and may never justify acceptance.
- Each window's parameters are selected **only from data prior to that
  window's test period** (no leakage through time travel in tuning).

## 5. Acceptance Thresholds (G1)

| Criterion | Threshold (aggregate OOS) |
|---|---|
| Sharpe ratio | **≥ 0.7** |
| Max drawdown | **≤ 20%** |
| Positive windows | **≥ 60%** of walk-forward test windows |
| Return after full costs | CAGR > 0 (net of slippage, FX friction, and any real commissions) |
| Sample size | ≥ 40 rebalance events in the OOS aggregate (significance floor) |
| Holdout | No threshold degradation > ⅓ of headroom vs. OOS aggregate (e.g., Sharpe ≥ 0.7 must not fall below ~0.47) |

**All must pass.** Near-misses are recorded honestly and treated as failures
(the "just one more window" impulse is how overfitting ships).

## 6. Stability & Sensitivity Requirements

| Test | Requirement |
|---|---|
| Parameter perturbation (FR-BT-07) | Each parameter ± 20%: accept/reject outcome **must not flip**; parameters that flip are flagged fragile and either re-parameterized or the strategy is rejected |
| Cost stress | 2× slippage (10 bps) must still pass §5 thresholds; FX friction ON in all runs |
| Cost sensitivity report | Included in the validation report: results at 1× / 2× costs |

## 7. Multiple-Testing Discipline

1. The experiment log counts **every** strategy variant tried — the honest
   denominator for any "it worked" claim.
2. Best-of-N selection over many variants must discount its headline metrics
   (deflated-Sharpe logic, applied qualitatively: N variants tried → expect
   the winner's OOS edge to shrink in live).
3. Thresholds may not be re-tuned to fit a result. Threshold changes are
   prospective, documented decisions (§9).

## 8. Review & Sign-Off (G1)

1. Validation report compiled: all §2–§7 artifacts + run IDs.
2. **48-hour cooldown**, then self-review against this checklist line-by-line
   (solo-development substitute for peer review).
3. G1 decision recorded in the document-control log: Pass / Fail / Iterate.
4. A passing G1 freezes: strategy parameters, universe, cost model — the
   frozen triple is what G2 paper-trades. **G2 tests the frozen artifact, not
   a moving target.**

## 9. Failure Handling

| Outcome | Action |
|---|---|
| Any §5 threshold missed | **Fail.** Iterate (new hypothesis → full §2–§8 cycle with a fresh holdout discipline) or reject the strategy family |
| Holdout degrades beyond headroom rule | Reject current parameters; the holdout is now contaminated — research restarts with a new holdout period when one accrues |
| Fragile parameters | Re-parameterize from first principles or simplify the strategy |
| Repeated failures | Accept a market where the strategy has no edge; capital stays in the defensive asset. **Not trading is a valid output of the process** |

## 10. Threshold Governance

These thresholds are a policy baseline, not physics. They may change only
via: written proposal → ADR-style decision record → updated in this document
and [Success Metrics](../08-planning/03-success-metrics.md) → applies
**prospectively only**. Tightening is allowed anytime; loosening requires the
full record and applies to future strategies, never a current candidate.
