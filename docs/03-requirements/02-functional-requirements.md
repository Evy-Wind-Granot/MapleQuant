# Functional Requirements

| | |
|---|---|
| **Document ID** | DOC-010 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [SRS](01-software-requirements-specification.md) · [NFR](03-non-functional-requirements.md) · [RTM](07-requirements-traceability-matrix.md) |

## 1. Conventions

- **Priority:** M = Must (MVP) · S = Should · C = Could · W = Won't (this release).
- **Verification legend:** UT = unit test · IT = integration/contract test ·
  BT-gate = backtest validation gate · RB = runbook · MON = monitoring check ·
  DOC = documented procedure · DR = operational drill.
- All timestamps are stored in **UTC**, reported in **America/Toronto**.
- "Session" = one scheduled daily cycle (pre-flight → data → signal → risk →
  orders → report). "Fail-closed" = uncertainty blocks trading (see §14).

## 2. FR-DA — Data Ingestion

| ID | Requirement | P | Verification |
|---|---|---|---|
| FR-DA-01 | Ingest EOD OHLCV for the configured universe from configured free sources (Stooq / yfinance / Alpaca), corporate-action-adjusted where provided. | M | IT (recorded fixtures) |
| FR-DA-02 | Validate every ingest batch (schema, ranges, OHLC sanity, duplicate dates) **before** persisting; reject + alert on any failure. | M | UT + IT |
| FR-DA-03 | Gap detection: bars older than a configured tolerance mark a symbol **stale**; stale input fails the session's dependent decisions (fail-closed). | M | UT |
| FR-DA-04 | Idempotent storage: re-ingesting any date range upserts without duplication; provenance (source, fetch time, adjustment method) recorded per bar. | M | UT |
| FR-DA-05 | Ingest daily CAD/USD FX rates from the Bank of Canada Valet API; required for all CAD translation. | M | IT |
| FR-DA-06 | Exchange calendar (NYSE primary; TSX when universe expands) drives scheduling, trading-day logic, and holiday handling. | M | UT (holiday fixtures) |
| FR-DA-07 | Intraday bars (hourly/15-min) for paper-realism checks and slippage studies. | S | IT |
| FR-DA-08 | Secondary-source fallback for EOD data with per-bar provenance flag. | C | IT |
| FR-DA-09 | Immutable data snapshot per backtest run (universe + range + content hash) for reproducibility. | M | IT + BT-gate |

## 3. FR-ST — Strategy Engine

| ID | Requirement | P | Verification |
|---|---|---|---|
| FR-ST-01 | Strategies are plug-in modules implementing the `Strategy` interface: consume a data window + config → emit a `TargetPortfolio` (weights + rationale). | M | UT (dummy strategy) |
| FR-ST-02 | **Determinism:** identical data + config → identical decision. No wall-clock reads, RNG, or network I/O inside strategy code; time is injected. | M | UT (double replay) |
| FR-ST-03 | Reference strategy **MQ-MOM-1** (monthly ETF momentum rotation, see [Strategy Engine Design](../04-architecture/04-strategy-engine-design.md)) shipped and documented. | M | BT-gate |
| FR-ST-04 | Every signal logs its rationale (indicator values, rankings, filters) to the audit log. | M | UT |
| FR-ST-05 | Configurable rebalance cadence (monthly default; weekly/daily supported). | S | UT |
| FR-ST-06 | **Replay equivalence:** on identical inputs, paper/live decisions must equal backtest decisions (CI golden test). | M | IT (CI) |

## 4. FR-BT — Backtesting

| ID | Requirement | P | Verification |
|---|---|---|---|
| FR-BT-01 | Backtests execute on the **same event-driven core** as paper/live ([ADR-003](../04-architecture/adr/ADR-003-single-codepath-event-driven-core.md)). | M | IT (equivalence test) |
| FR-BT-02 | Cost model: slippage (default 5 bps), liquidity-aware spread assumption, FX friction (configurable %), commissions only where the broker actually charges. | M | UT |
| FR-BT-03 | **No-lookahead enforced structurally:** the core data windowing hands strategies only data available at decision time — correctness does not depend on strategy discipline. | M | UT (truncation property test) |
| FR-BT-04 | **Point-in-time universe membership** (no survivorship bias); delisted instruments remain in historical windows. | M | UT (delisting fixture) |
| FR-BT-05 | Metrics suite: CAGR, volatility, Sharpe, Sortino, max drawdown + duration, monthly returns table, turnover, exposure, benchmark delta, per-trade log. | M | UT (golden outputs) |
| FR-BT-06 | Walk-forward runner: rolling train/test windows with per-window and aggregate reports. | M | IT |
| FR-BT-07 | Parameter sweep with ±20% perturbation and a stability report (fragile parameters are flagged). | M | IT |
| FR-BT-08 | Experiment artifacts persisted: config hash, code version, data snapshot ID, parameters, metrics — diffable across runs. | M | IT |
| FR-BT-09 | Monte Carlo (bootstrap) analysis for drawdown distribution estimates. | C | UT |

## 5. FR-EX — Execution

| ID | Requirement | P | Verification |
|---|---|---|---|
| FR-EX-01 | All order flow goes through the `BrokerAdapter` interface; the engine contains **zero** broker-specific code. | M | IT (adapter contract suite) |
| FR-EX-02 | **Idempotent submission:** every order carries a unique `client_order_id`; duplicate submissions are detected and rejected (no second order). | M | UT |
| FR-EX-03 | On timeout / uncertain state: query by `client_order_id` before any further action; **never blind re-submit**. | M | UT (fault injection) |
| FR-EX-04 | Order state machine maps broker states to internal states; unmappable/unknown state → treated as worst case (live), new orders halted, alert raised. | M | UT |
| FR-EX-05 | Market-hours gate: orders only during the broker's supported session; broker rate limits respected with exponential backoff. | M | UT + IT |
| FR-EX-06 | Error taxonomy: **transient** (network, 5xx, rate limit) → bounded retry with backoff; **permanent** (validation, insufficient funds) → abort + alert. Never retry a permanent error. | M | UT |
| FR-EX-07 | Fractional shares when the adapter's capability flag allows; otherwise whole-share rounding with residual cash tracking. | S | UT |
| FR-EX-08 | **Live mode requires an arm step** (`mode=live` config **plus** deliberate `mq arm live` action); unarmed live → session exits with no orders. | M | IT |

## 6. FR-RK — Risk Management

| ID | Requirement | P | Verification |
|---|---|---|---|
| FR-RK-01 | Pre-trade gate on **every** order: symbol whitelist; per-position cap ≤ 40% (hard); target weight cap ≤ 35%; per-order notional cap; daily order count ≤ 20; gross exposure ≤ 100% NAV (**no leverage**). Every veto is logged. | M | UT (property tests) |
| FR-RK-02 | **Daily loss circuit breaker:** NAV drop ≥ 3% within a session day → entries halted for the day + alert. | M | UT |
| FR-RK-03 | **Drawdown breaker:** ≥ 10% below high-water mark → full halt; re-arm requires CLI + 24h cooldown + logged reason. | M | UT + IT |
| FR-RK-04 | **Kill switch:** triggerable via Telegram command, admin CLI, or local file; blocks entries; cancels resting orders; optional flatten (config). All paths logged. | M | IT + DR (drill) |
| FR-RK-05 | No auto-resume: after kill or breaker, resumption requires a deliberate CLI step with cooldown. | M | UT |
| FR-RK-06 | Hard limits are **code-enforced maxima**: configuration may be stricter, never looser. | M | UT |
| FR-RK-07 | Data freshness gate: stale/missing data for any decision-relevant symbol → no dependent orders. | M | UT |
| FR-RK-08 | Volatility-target sizing option for strategy weights. | S | UT |
| FR-RK-09 | Reconciliation break or unreachable broker → trading halted (fail-closed). | M | IT |

## 7. FR-RC — Reconciliation

| ID | Requirement | P | Verification |
|---|---|---|---|
| FR-RC-01 | Pre-session reconciliation: broker positions/cash vs. internal book; any break → session aborts with **no orders** + alert with a diff table. | M | IT |
| FR-RC-02 | EOD reconciliation; an unresolved break keeps the next session halted. | M | IT |
| FR-RC-03 | Auto-repair only for whitelisted benign cases (rules documented); everything else requires manual CLI resolution. | M | UT |
| FR-RC-04 | All reconciliation results (including clean runs) logged with a full state snapshot. | M | UT |

## 8. FR-PF — Portfolio & Accounting

| ID | Requirement | P | Verification |
|---|---|---|---|
| FR-PF-01 | Book of record: positions, per-lot cost basis, cash, NAV per session. | M | UT |
| FR-PF-02 | Realized/unrealized P&L per trade and per session. | M | UT (golden fixtures) |
| FR-PF-03 | **Canadian ACB** per lot, including dividend reinvestment and return-of-capital adjustments. | M | UT (fixtures) |
| FR-PF-04 | **Superficial-loss flagging:** identical-property repurchases within 30 days flagged in records. | M | UT |
| FR-PF-05 | CAD base currency: USD values translated at Bank of Canada daily rates for books and reports. | M | UT |
| FR-PF-06 | Dividend / corporate-action ingestion (splits, distributions) updating the books. | S | UT |

## 9. FR-NT — Notifications

| ID | Requirement | P | Verification |
|---|---|---|---|
| FR-NT-01 | Daily digest after each session: NAV, positions, P&L, fills, exceptions, breaker status. | M | IT |
| FR-NT-02 | Immediate alerts: circuit breakers, reconciliation breaks, order errors, kill events, stale data, missed runs. | M | IT |
| FR-NT-03 | Telegram commands (`/status`, `/kill`) restricted to the allow-listed chat ID. | M | IT |
| FR-NT-04 | Notification failures never block the trading loop; failures are surfaced via logs + heartbeat. | M | UT |
| FR-NT-05 | Weekly research digest (live vs. backtest drift). | C | — |

## 10. FR-CF — Configuration & Admin

| ID | Requirement | P | Verification |
|---|---|---|---|
| FR-CF-01 | YAML config validated by a strict pydantic schema at startup; invalid config → non-zero exit **before** any trading action. | M | UT |
| FR-CF-02 | Modes: `backtest` \| `paper` \| `live`; `live` additionally requires the arm step (FR-EX-08). | M | IT |
| FR-CF-03 | No secrets in config files — injected at runtime via environment/credentials (SEC-01, SEC-03). | M | UT + gitleaks in CI |
| FR-CF-04 | Admin CLI: `run`, `backtest`, `status`, `arm`, `disarm`, `kill`, `reconcile`, `report`, `tax-export`. | M | IT (CLI smoke suite) |
| FR-CF-05 | Config hash stamped into every audit event and report. | M | UT |
| FR-CF-06 | Config diffs included in the first session report after any change. | S | UT |

## 11. FR-TX — Tax

| ID | Requirement | P | Verification |
|---|---|---|---|
| FR-TX-01 | Annual tax export (CSV): per-trade proceeds, ACB, capital gain/loss, CAD translation, superficial-loss flags; retained ≥ 7 years. | M | IT (golden export) |
| FR-TX-02 | Export mapping to self-serve CRA reporting documented (proceeds/ACB columns → Schedule 3 lines). | M | DOC |

## 12. FR-AU — Audit

| ID | Requirement | P | Verification |
|---|---|---|---|
| FR-AU-01 | **Append-only JSONL audit log**; every entry carries: UTC timestamp, event type, config hash, code version, actor. Retention ≥ 7 years. | M | UT |
| FR-AU-02 | Order lineage: order → risk decision → signal rationale → data-window hash — reconstructable end-to-end. | M | UT |
| FR-AU-03 | Tamper-evidence: hash-chained entries (or daily checksum tree) + an integrity-check tool. | M | UT |
| FR-AU-04 | Quarterly audit-review checklist performed and recorded. | S | DOC |

## 13. Coverage Summary

| Group | Count | Must |
|---|---|---|
| FR-DA | 9 | 7 |
| FR-ST | 6 | 6 |
| FR-BT | 9 | 8 |
| FR-EX | 8 | 8 |
| FR-RK | 9 | 9 |
| FR-RC | 4 | 4 |
| FR-PF | 6 | 5 |
| FR-NT | 5 | 4 |
| FR-CF | 6 | 5 |
| FR-TX | 2 | 2 |
| FR-AU | 4 | 3 |
| **Total** | **68** | **61** |

## 14. System-Wide Invariants (Non-Negotiable Properties)

| ID | Invariant |
|---|---|
| INV-1 | **Fail-closed default:** any uncertainty — data staleness, unclear order state, failed check, unreachable broker — results in *no new orders*, plus an alert. |
| INV-2 | **The risk gate is architecturally unavoidable:** the execution module accepts orders only via the risk manager's approval; there is no code path that bypasses it. |
| INV-3 | **Idempotency everywhere:** every external mutation (order placement, cancel) carries a client-generated idempotency key. |
| INV-4 | **The broker is the source of truth:** the internal book converges to the broker or trading halts. |
| INV-5 | **Full reproducibility:** any historical decision is reconstructable from the audit log + data snapshot + code version. |
| INV-6 | **No order without:** fresh data ∧ armed mode (if live) ∧ open market ∧ clean reconciliation ∧ risk approval. |
| INV-7 | **Every limit has a hard, code-enforced maximum** that configuration cannot loosen (FR-RK-06). |

## 15. Out-of-Scope Functional Items (W)

Recorded so "no" stays deliberate: multi-account support; web UI; options/shorting/margin;
intraday cadences below 15 minutes; automated corporate-action *trading* responses;
automatic money movement (permanently — RL-6).
