# Product Overview

| | |
|---|---|
| **Document ID** | DOC-007 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [PRD](02-product-requirements-document.md) · [SRS](../03-requirements/01-software-requirements-specification.md) · [Executive Summary](../00-executive/01-executive-summary.md) |

## 1. What the Product Is

**MapleQuant Platform** — a single-host Python system that ingests market data,
backtests systematic strategies, and (after staged validation) trades a
capped allocation of the Founder's own capital through a broker-agnostic
execution layer, under strict risk controls and full audit logging.

## 2. What the Product Is NOT

| Not this | Why |
|---|---|
| A robo-advisor or fund | No third-party capital (compliance red line RL-1) |
| A signal/alert selling service | Advice-distribution risk (RL-3) |
| High-frequency / latency arbitrage | Incompatible with free data + daily cycle design |
| A crypto, options, FX, or leveraged product | Phase 1 scope is US-listed cash equities/ETFs only |
| A guaranteed-return anything | No performance claims are ever made (RL-4) |
| A multi-tenant SaaS | Solo, single-tenant by design until P4 gate |

## 3. Target User

**The Operator** (founder) for Phases 0–3: technical, risk-aware, intolerant of
hidden magic, wants plain files, inspectable decisions, and boring reliability.
Any future user (S2/S3 segments) is intentionally the same kind of person first,
see [Market Analysis](../01-market-compliance/01-market-analysis.md).

## 4. Core Value Propositions

1. **Discipline as software** — the strategy runs or it doesn't; there is no
   discretionary override without an audited action.
2. **One code path** — the same event-driven engine runs backtests, paper, and
   live; what you backtest is what you trade ([ADR-003](../04-architecture/adr/ADR-003-single-codepath-event-driven-core.md)).
3. **$0 fixed cost** — free-tier/self-hosted by design ([Cost Management](../06-operations/05-cost-management.md)).
4. **Broker-agnostic** — Wealthsimple adapter (when API access is granted),
   Alpaca paper as the reference adapter; the engine knows nothing about brokers
   ([ADR-002](../04-architecture/adr/ADR-002-broker-agnostic-adapter-architecture.md)).
5. **Canadian by default** — CAD base currency, ACB accounting, BoC FX rates,
   Toronto calendar.
6. **Fail-closed** — stale data, unclear order states, failed checks → **no
   trade**, plus alert. Silence is failure, not inaction.

## 5. MVP Definition (what earns G3 — live trading)

| Capability | Detail |
|---|---|
| Data ingestion | Daily OHLCV for ~100-symbol universe; EOD/hourly cadence; FX rates (BoC) |
| Strategy v1 | Monthly-rebalanced ETF momentum rotation ("MQ-MOM-1", see [Strategy Engine Design](../04-architecture/04-strategy-engine-design.md)) |
| Backtest engine | Event-driven, cost-aware, bias-controlled (no lookahead/survivorship) |
| Execution | Paper mode (Alpaca); live mode (Wealthsimple API or fallback broker) gated by config arm step |
| Risk manager | Position/sector caps, daily-loss & drawdown circuit breakers, order caps, kill switch |
| Reconciliation | Broker-vs-book position and cash check before every session and at EOD |
| Accounting | Per-lot ACB, realized/unrealized P&L, CAD translation |
| Notifications | Telegram: daily report, alerts, commands (`/status`, `/kill`) |
| Audit | Immutable JSONL decision log; every order traceable to signal + config hash |

## 6. Explicitly Out of Scope (MVP)

- Web UI / dashboards (a static status page is the maximum, and only if free).
- Options, shorting, margin, leverage, crypto.
- Intraday strategies below 15-minute bars.
- TSX-listed universe (blocked by real-time data licensing under $0 rule; OQ-05).
- Multi-user anything; horizontal scaling; microservices.
- Automatic withdrawal/movement of money — **never** (RL-6, [SEC-02](../03-requirements/04-security-requirements.md)).

## 7. Product Principles

1. **Files over databases, boring over clever** — YAML config, SQLite, JSONL logs.
2. **The operator is always 15 minutes away from full stop** — kill switch and
   runbooks are first-class features, not afterthoughts.
3. **Every number must be re-derivable** — logs + data snapshot + code version
   reproduce any historical decision.
4. **Never surprise the broker** — official APIs, conservative order flow,
   rate-limit respect.
