# Backtesting Design

| | |
|---|---|
| **Document ID** | DOC-021 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [ADR-003](adr/ADR-003-single-codepath-event-driven-core.md) · [FR-BT group](../03-requirements/02-functional-requirements.md) · [Validation Standards](../07-quality/02-backtesting-validation-standards.md) |

## 1. The One Rule

**A backtest is the live engine with different drivers** ([ADR-003](adr/ADR-003-single-codepath-event-driven-core.md)):

| Live component | Backtest counterpart |
|---|---|
| Broker adapter (REST) | `SimBroker` adapter (fills, fees, slippage model) |
| Data ingestor (fetch) | `HistoricalDataWindow` (snapshot replay) |
| Wall-clock scheduling | Event clock driven by bar timestamps |
| Risk manager / accounting / audit / notifier | **Identical code** |

The replay-equivalence golden test (FR-ST-06) proves this continuously: running
the live pipeline over historical data must equal the backtest, decision for
decision.

## 2. SimBroker (execution simulation)

| Aspect | Model |
|---|---|
| Fill timing | Orders placed at session T execute at next available open (T+1 open) — no same-bar fills (anti-lookahead) |
| Fill price | Open ± configurable slippage (default 5 bps adverse); liquidity-aware spread for thin ETFs |
| Partial fills | Not modeled in v1 (universe is highly liquid ETFs); documented assumption |
| Commissions | $0 only when broker confirms (A-4); parameterized so broker changes are one config line |
| FX friction | Conversion fee parameter (Wealthsimple historically ~1.5% — verify, CT-04) applied on CAD↔USD flows; modeling favors USD-sub-account holding |
| Cash | Interest on CAD/USD cash at T-bill proxy (conservative; documented) |
| Corporate actions | Applied from the actions table (splits, dividends reinvest per strategy rules) |

## 3. Bias Controls (structural, not procedural)

| Bias | Structural control |
|---|---|
| Lookahead | `HistoricalDataWindow` only serves bars ≤ decision timestamp; strategies can't request the future (FR-BT-03); truncation property test |
| Survivorship | Point-in-time universe (FR-BT-04); delistings present in history |
| Overfitting | Walk-forward + sensitivity gates (§4) and [Validation Standards](../07-quality/02-backtesting-validation-standards.md) — G1 refuses fragile configs |
| Data snooping | Experiment log counts every test run; conclusions cite the full log, not the best run |
| Cost optimism | Conservative defaults (5 bps slippage, FX friction on) + sensitivity on costs |

## 4. Evaluation Runners

1. **Single window:** full metrics suite (FR-BT-05) + per-trade log + rationale
   dump.
2. **Walk-forward (FR-BT-06):** rolling windows (default: 10 y train / 1 y test,
   stepped yearly; ≥ 12 test windows). Acceptance is dominated by **aggregate
   out-of-sample** results, never in-sample.
3. **Sensitivity (FR-BT-07):** each parameter ±20% on the grid; reports flag
   any parameter whose perturbation flips the accept/reject outcome.
4. **Monte Carlo (FR-BT-09, optional):** block-bootstrap of OOS returns →
   drawdown distribution feeding the risk limits sanity check.

## 5. Experiment Tracking (FR-BT-08)

Every run is an artifact directory: `config.yaml` (hash), `code version`,
`data snapshot ID`, parameters, metrics JSON, plots (static), and a generated
human report. Runs are comparable via a small `mq report diff` command. The
experiment log is append-only and referenced by conclusions — a G1 claim must
cite run IDs.

## 6. Acceptance Gates

Quantitative thresholds (OOS Sharpe, drawdown, stability) are **defined in the
[Backtesting Validation Standards](../07-quality/02-backtesting-validation-standards.md)**
— the single source of truth for what "validated" means at G1. This document
defines *how* results are produced; that document defines *when they count*.
