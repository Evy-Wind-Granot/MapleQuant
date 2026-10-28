# Glossary

| | |
|---|---|
| **Document ID** | DOC-039 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |

| Term | Definition |
|---|---|
| **ACB** | Adjusted Cost Base — Canadian tax concept: the true per-share cost of a holding after adjustments (reinvested dividends, return of capital, splits). Used to compute capital gains. |
| **Adapter** | The only module allowed to talk to a specific broker's API (ADR-002). |
| **ADR** | Architecture Decision Record — a numbered, immutable "why we decided X" document. |
| **Arm step** | The deliberate human action (`mq arm live`) required before live trading can ever start (FR-EX-08). |
| **Backtest** | Simulation of a strategy on historical data using the same engine core as live trading. |
| **Bar / OHLCV** | One period of price data: Open, High, Low, Close, Volume. |
| **Breaker (circuit breaker)** | Automatic halt: daily-loss breaker (≥ 3% NAV drop) or drawdown breaker (≥ 10% below HWM). |
| **CIRO** | Canadian Investment Regulatory Organization — the SRO for investment dealers (e.g., Wealthsimple), formed 2023 from IIROC + MFDA. |
| **Client order ID** | Platform-generated unique key per order — the idempotency mechanism that makes retries safe (FR-EX-02). |
| **CSA** | Canadian Securities Administrators — the provincial commissions' harmonization body. |
| **Defensive asset** | The low-volatility fallback holding (short-duration Treasury ETF / cash) when momentum is absent. |
| **Digest** | The daily post-session Telegram report (FR-NT-01). |
| **Drawdown** | Decline from a high-water mark to a trough, in %. |
| **EOD** | End-of-day (daily bars / after market close). |
| **Fail-closed** | Design rule: any uncertainty results in *no new orders* + an alert (INV-1). |
| **Fill** | A (partial or complete) execution of an order at the broker. |
| **FX friction** | The cost of converting CAD↔USD (conversion fee + spread), modeled explicitly in backtests. |
| **Gate (G0–G4)** | Stage-gate: a pre-defined set of criteria that must be met before the next phase (Roadmap). |
| **Heartbeat** | Scheduled ping to Healthchecks.io; a missed ping = missed-run alert (OPS-06). |
| **HWM** | High-water mark — highest NAV ever; baseline for the drawdown breaker. |
| **Idempotency** | Property that repeating an operation (e.g., an order submit) has no additional effect. |
| **INV-1…7** | The seven system-wide invariants (Functional Requirements §14). |
| **Kill switch** | The emergency halt: Telegram `/kill`, CLI, or local flag; blocks entries, cancels rests (FR-RK-04). |
| **MoSCoW** | Prioritization scheme: Must / Should / Could / Won't. |
| **Momentum** | Return-based signal: assets that performed relatively better over lookbacks are bought. |
| **NAV** | Net asset value — total account value (cash + positions, in CAD). |
| **NYSE** | New York Stock Exchange — the trading calendar governing Phase 1 sessions. |
| **Operator** | The human running the platform (the Founder). |
| **OOS** | Out-of-sample — data periods not used for tuning; the only results that count for G1. |
| **Paper trading** | Live-market simulation with fake money via a broker's paper API (Alpaca). |
| **PIPEDA** | Canada's federal private-sector privacy law (relevant only at P4). |
| **Property test** | Randomized-input test asserting an invariant holds broadly (hypothesis). |
| **Reconciliation** | Verifying platform books match broker reality; break = halt (FR-RC). |
| **restic** | Free, encrypted, deduplicating backup tool. |
| **RL-1…RL-7** | Compliance red lines (Regulatory §3). |
| **RSK-xx** | Risk register entry; **R-RES-x** = accepted residual risk (threat model). |
| **RTM** | Requirements Traceability Matrix — requirement → verification artifact mapping. |
| **SEV1/2/3** | Incident severity levels (Incident Response §1). |
| **Sharpe / Sortino** | Return-per-risk ratios (Sortino penalizes only downside volatility). |
| **SimBroker** | The backtest-side BrokerAdapter that simulates fills and fees. |
| **Slippage** | Difference between modeled/decision price and actual fill price, in basis points. |
| **Snapshot** | Immutable, hash-identified copy of input data pinned to a backtest run. |
| **Superficial loss** | Canadian tax rule: a capital loss is disallowed if the identical property is repurchased within 30 days; flagged, not adjudicated, by the platform. |
| **systemd-creds / age** | Linux-native / open-source mechanisms for encrypting secrets at rest on the host. |
| **Tailscale** | Zero-config private network (free tier) — the only path into the host. |
| **Walk-forward** | Validation protocol: rolling train/test windows; acceptance rests on aggregate OOS results. |
| **WAL** | Write-ahead logging — SQLite mode enabling safe concurrent readers with one writer. |
| **$0 constraint (NFR-24)** | Hard fixed-cost budget of zero dollars per month. |
