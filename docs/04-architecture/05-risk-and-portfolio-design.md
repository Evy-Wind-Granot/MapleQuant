# Risk Management & Portfolio Accounting Design

| | |
|---|---|
| **Document ID** | DOC-020 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [FR-RK / FR-RC / FR-PF groups](../03-requirements/02-functional-requirements.md) · [Architecture §5–7](01-system-architecture.md) · [Success Metrics](../08-planning/03-success-metrics.md) |

## 1. Risk Manager Architecture

The risk manager sits **between** portfolio translation and execution. It is
the sole minter of `ApprovedOrder` (Architecture R3) — the engine cannot
construct an order without it. Veto reasons are always logged (FR-RK-01).

### 1.1 Pre-Trade Gate (checked per order, atomically)

| Check | Initial value | Hard code maximum (FR-RK-06) |
|---|---|---|
| Symbol whitelist | configured universe | — (unknown symbol always veto) |
| Per-position weight | ≤ 35% target | **40% absolute** |
| Per-order notional | ≤ CAD 1,000 (P3) | **CAD 2,500 absolute** |
| Daily order count | ≤ 6 (typical session) | **20 absolute** |
| Gross exposure | ≤ 100% NAV | **100% (no leverage, ever)** |
| Market-hours gate | broker session open (FR-EX-05) | — |
| Data freshness | all decision symbols fresh (FR-RK-07) | — |
| Reconciliation state | clean (FR-RK-09) | — |
| Kill/breaker state | clear (or exits-only mode) | — |

Config may tighten any number; loosening beyond the code maximum is a schema
violation at startup (FR-CF-01) — **misconfiguration cannot create a runaway
bot** (SEC-13).

### 1.2 Circuit Breakers & Kill Switch

| Control | Trigger | Action | Recovery |
|---|---|---|---|
| Daily loss breaker (FR-RK-02) | session NAV drop ≥ 3% | halt **entries** for the day; alert | next trading day automatically |
| Drawdown breaker (FR-RK-03) | NAV ≥ 10% below high-water mark | full halt; alert | manual CLI re-arm + 24 h cooldown + logged reason (FR-RK-05) |
| Kill switch (FR-RK-04) | Telegram `/kill` · `mq kill` · local flag file | block entries; cancel resting orders; optional flatten (config, default **off**) | manual `mq disarm` + cooldown + logged reason |

Kill paths are independent (SEC-23): the local file works with no network;
Telegram works off-host; the CLI works over Tailscale.

### 1.3 Position Sizing

- Equal-weight by strategy (v1) with the 35% cap; optional volatility-targeting
  (FR-RK-08) = target_weight × min(1, σ_target/σ_realized), floored at 0.
- Whole-share or fractional per adapter capability (FR-EX-07); residuals stay
  as cash.

## 2. Reconciliation Design (FR-RC)

- **Broker is the source of truth** (INV-4). The book converges to the broker,
  or trading halts.
- Pre-session: positions + cash diff; tolerance = exact match (small accounts,
  few symbols — no tolerance excuse). Break → abort + alert (FR-RC-01).
- Post-trade: fills vs. expected; EOD positions vs. book (FR-RC-02).
- Auto-repair whitelist (FR-RC-03) — initially empty. First unknown cause =
  manual resolution via `mq reconcile --explain` + runbook RB-006.
- Every reconciliation (clean or not) writes a state snapshot (FR-RC-04).

## 3. Portfolio Accounting Design (FR-PF)

### 3.1 Book of Record

| Table | Content |
|---|---|
| `positions` | symbol, qty (Decimal), per-lot cost rows, currency |
| `cash` | by currency (CAD base, USD for US-listed) |
| `nav_history` | per-session NAV in CAD + components |
| `trades` / `fills` | full lifecycle incl. fees, FX applied |
| `acb_events` | lot creations, splits, dividend reinvestments, ROC adjustments |

### 3.2 Canadian Tax Semantics

- **ACB per lot** (FR-PF-03): every purchase creates lots; dividends
  reinvested and return-of-capital adjust ACB; splits re-basis lots. ACB is
  recomputed from events, never stored as an unverified running number.
- **Superficial loss flag** (FR-PF-04): sale at a loss + identical property
  (incl. the defensive ETF) repurchased within ±30 days → flag in the tax
  export (disallowance decision stays with the human).
- **CAD translation** (FR-PF-05): every USD transaction converted at the BoC
  official rate for its date (FR-DA-05); the FX rate used is stored per event
  (auditable, NFR-22).
- Exports: annual CSV mapping to Schedule 3-style reporting (FR-TX-01/02).

### 3.3 P&L & NAV

- Realized P&L per trade (ACB method); unrealized per session close at EOD
  prices; NAV = cash + positions translated to CAD; high-water mark tracked for
  the drawdown breaker.

## 4. Failure Interactions (who wins)

Any conflict resolves to the safest interpretation: freshness fail → risk
vetoes all orders regardless of signal; unknown order state → execution halts
new submissions regardless of risk approval; breaker trip → orchestrator
refuses to even propose orders; reconciliation break → session never reaches
the signal stage (Architecture §7 ordering).
