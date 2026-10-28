# Product Requirements Document (PRD)

| | |
|---|---|
| **Document ID** | DOC-008 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [Product Overview](01-product-overview.md) · [Functional Requirements](../03-requirements/02-functional-requirements.md) · [Success Metrics](../08-planning/03-success-metrics.md) |

## 1. Personas

### P-01 — "The Operator" (Founder; the only user in Phases 0–3)
- Technical (writes/reads Python, YAML, SQL), risk-averse, time-poor (~≤30 min/week
  steady-state, 15 min/day during P3 supervision).
- Goals: run a validated strategy hands-off; know immediately when anything
  deviates; never lose control of the account.
- Pain: manual execution is emotional; spreadsheets drift; tools assume US brokers.

### P-02 — "Self-Directed Sam" (future, P4 — S2 segment)
- Canadian developer-investor; wants their own brokerage (CIRO member), CAD
  tax support, and to pay ~nothing. Not served before MapleQuant.

## 2. User Stories & Priorities (MoSCoW)

### EP-1 — Configure

| ID | Story | Priority |
|---|---|---|
| US-101 | As the Operator, I create/edit strategy and risk configuration in versioned YAML files so changes are reviewable and auditable. | Must |
| US-102 | As the Operator, config is validated at startup (schema + sanity limits) so a bad config cannot start a trading session. | Must |
| US-103 | As the Operator, I switch modes (backtest → paper → live) via explicit config plus a deliberate **arm** step, so live trading can never start by accident. | Must |
| US-104 | As the Operator, I keep a library of strategy parameter sets to compare experiments. | Should |

### EP-2 — Research & Backtest

| ID | Story | Priority |
|---|---|---|
| US-201 | As the Operator, I run a backtest over an explicit date range and universe, producing a full metrics report (return, Sharpe, Sortino, max drawdown, turnover, exposure, per-trade log). | Must |
| US-202 | As the Operator, the backtest models costs (spread/slippage, FX friction) so results are conservative, not flattering. | Must |
| US-203 | As the Operator, I run walk-forward evaluations and parameter-sensitivity sweeps to detect fragility before trusting a strategy. | Must |
| US-204 | As the Operator, I get a machine- and human-readable validation report per experiment (config hash, data snapshot, code version, metrics) so results are reproducible. | Must |

### EP-3 — Deploy & Operate

| ID | Story | Priority |
|---|---|---|
| US-301 | As the Operator, I deploy via a documented procedure (CI → release → host) and can roll back to the previous release. | Must |
| US-302 | As the Operator, the scheduled daily cycle (pre-flight → data refresh → signal → risk gate → orders → report) runs unattended at market-appropriate times. | Must |
| US-303 | As the Operator, a missed or failed cycle pages me (heartbeat + run-failure alerts) rather than failing silently. | Must |

### EP-4 — Monitor & Intervene

| ID | Story | Priority |
|---|---|---|
| US-401 | As the Operator, I receive a daily digest (positions, equity, P&L, fills, exceptions) via Telegram after each session. | Must |
| US-402 | As the Operator, alerts fire on circuit breakers, reconciliation breaks, stale data, order errors, and kill-switch events. | Must |
| US-403 | As the Operator, I can trigger the kill switch from Telegram or the host, halting new entries, cancelling resting orders, and optionally flattening — all logged. | Must |
| US-404 | As the Operator, I can cancel a specific order or flatten a specific position via an admin CLI without touching other automation. | Must |
| US-405 | As the Operator, re-arming after a kill requires a deliberate CLI step with a cooldown, so panic actions don't auto-reverse. | Must |

### EP-5 — Trust, Audit & Tax

| ID | Story | Priority |
|---|---|---|
| US-501 | As the Operator, every decision (signal, risk veto, order, fill) is recorded immutably with timestamp, config hash, and code version. | Must |
| US-502 | As the Operator, the system reconciles broker positions/cash vs. its own books before every session and at EOD, and halts trading on unresolved breaks. | Must |
| US-503 | As the Operator, I can export tax-ready trade records with per-lot ACB and CAD translation (BoC rates) for CRA reporting. | Must |
| US-504 | As the Operator, I can reproduce any historical decision from the audit log + archived data. | Should |

## 3. Key Acceptance Criteria (selected)

**US-103 (mode arming)**
- ✅ Given a config with `mode=live`, when the session starts without the arm
  file present, then the session exits with no orders and alerts "not armed".
- ✅ Arming requires explicit CLI: `mq arm live --i-understand-risks`.

**US-302 (daily cycle)**
- ✅ Given a trading day per the NYSE calendar, when the scheduler fires, the
  full cycle completes and reports success via heartbeat; any stage failure
  fails closed (no orders) and alerts.

**US-403 (kill switch)**
- ✅ Given any state, when `/kill` is received, then within 5s: no new entry
  orders are possible, all cancel-able resting orders are cancelled, an alert
  confirms the action, and the event is in the audit log.
- ✅ Deactivation requires `mq disarm` on the host with a logged reason.

**US-502 (reconciliation)**
- ✅ Given broker position ≠ book position, then the session does not place
  orders and alerts with the diff table.

## 4. UX Surfaces

| Surface | Phase | Notes |
|---|---|---|
| YAML config files | P0 | Source of truth; validated by pydantic schemas |
| Admin CLI (`mq`) | P0 | typer-based: `mq run`, `mq backtest`, `mq status`, `mq arm`, `mq disarm`, `mq kill`, `mq report`, `mq tax-export`, `mq reconcile` |
| Telegram bot | P2 | Alerts + daily digest + `/status` `/kill` (authenticated chat ID allow-list) |
| Heartbeat (Healthchecks.io) | P2 | Dead-man's switch for scheduled runs |
| Metrics dashboard (Grafana) | P3 | Optional; see [Monitoring](../06-operations/02-monitoring-and-alerting.md) |
| Static HTML status page | Maybe | Only if zero-maintenance; not in MVP |

## 5. MVP Release Plan

| Release | Contents | Gate |
|---|---|---|
| **v0.1** | Data layer + storage + CLI skeleton + config validation | G0 exit |
| **v0.2** | Backtest engine + metrics + experiment reports | — |
| **v0.3** | Strategy v1 (MQ-MOM-1) + validation report passes [standards](../07-quality/02-backtesting-validation-standards.md) | **G1** |
| **v0.4** | Paper execution (Alpaca adapter) + risk manager + reconciliation + notifications | Paper start |
| **v0.5** | Hardening: 60-day paper, DR drill, kill-switch drill, monitoring coverage ≥ targets | **G2** |
| **v1.0** | Live execution via Wealthsimple adapter (or fallback broker), arm step, supervised | **G3** |
| v1.x | Stepped capital increases, ACB export polish, second strategy evaluation | G4 decision input |

## 6. Constraints Carried into Requirements

- $0/month fixed cost (→ [OPS-13](../03-requirements/05-operational-requirements.md), [Cost Management](../06-operations/05-cost-management.md)).
- Single host, solo operations (→ [NFR-07](../03-requirements/03-non-functional-requirements.md) maintainability).
- Broker-agnostic (→ FR-EX group; [ADR-002](../04-architecture/adr/ADR-002-broker-agnostic-adapter-architecture.md)).
- Fail-closed everywhere (→ FR-RK group; [System Architecture §7](../04-architecture/01-system-architecture.md)).
- Audit-grade record-keeping (→ FR-AU group; CRA 6-year retention).
