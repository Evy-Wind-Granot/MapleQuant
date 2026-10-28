# Software Requirements Specification (SRS)

| | |
|---|---|
| **Document ID** | DOC-009 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | Sub-documents: [Functional](02-functional-requirements.md) · [Non-Functional](03-non-functional-requirements.md) · [Security](04-security-requirements.md) · [Operational](05-operational-requirements.md) · [Data](06-data-requirements.md) · [RTM](07-requirements-traceability-matrix.md) |
| **Standard** | IEEE Std 830-inspired structure |

## 1. Purpose

This SRS consolidates the complete requirements baseline for the MapleQuant
Platform. It is the authoritative reference for what the system must do
(functional), how well (non-functional), how securely (security), and how it
must be run (operational). **No code is written against an unstated
requirement; no requirement changes without updating this set.**

## 2. Scope

**In scope:** data ingestion, backtesting, strategy evaluation, risk
management, order execution via broker adapters (paper and live), portfolio
accounting, reconciliation, notifications, audit logging, and tax export — for
a single-account, single-host, single-operator deployment trading US-listed
cash equities/ETFs on low-frequency (daily/monthly) decision cycles.

**Out of scope:** see [Product Overview §6](../02-product/01-product-overview.md).

## 3. System Context

```mermaid
flowchart LR
  subgraph External
    DS[Market data sources\nStooq / yfinance / Alpaca]
    FX[Bank of Canada\nValet FX API]
    BR[Broker API\nWealthsimple (pending) / Alpaca paper]
    TG[Telegram Bot API]
    HC[Healthchecks.io\nheartbeat]
    GH[GitHub\nrepo + CI]
    TS[Tailscale admin access]
  end
  subgraph Host[Single Linux host — OCI Always Free ARM or home server]
    ORCH[Scheduler / Orchestrator]
    ING[Data ingestion]
    DB[(SQLite state + Parquet cold storage)]
    SE[Strategy engine]
    RM[Risk manager]
    OM[Execution manager]
    PA[Portfolio accounting]
    RC[Reconciliation]
    NT[Notifier]
    AU[(Append-only audit log)]
    KS[Kill switch]
  end
  DS --> ING --> DB
  FX --> ING
  DB --> SE --> RM --> OM --> BR
  BR --> RC --> DB
  BR --> OM
  PA --> DB
  ORCH --> ING & SE & RM & OM & RC & PA
  KS -.veto.-> OM
  NT --> TG
  ORCH --> HC
  AU -.records everything.-> SE & RM & OM & RC & PA
  TS -.admin.-> ORCH
  GH -.deploy.-> Host
```

## 4. Stakeholders

| Stakeholder | Interest |
|---|---|
| Operator/Founder (primary user) | Safe, disciplined automated trading of own capital; auditability; minimal ops load |
| Future self / P-02 users | A trustworthy, Canadian-localized platform if commercialized |
| Broker(s) | Compliance with ToS; well-behaved order flow (no scraping, no abusive patterns) |
| Regulators (indirect) | Personal-use perimeter maintained; records available |
| CRA (indirect) | Accurate capital gains / ACB records |

## 5. Operating Environments

| Environment | Location | Mode | Data |
|---|---|---|---|
| Dev | Founder's workstation | backtest/paper | Free sources |
| Paper | Host (OCI free tier or home server) | `paper` | Free sources + Alpaca paper |
| Live | Same host, separate profile + DB | `live` (armed) | Broker + free sources |

All three run the **same engine binary**; only config, credentials, and DB
differ ([ADR-003](../04-architecture/adr/ADR-003-single-codepath-event-driven-core.md)).

## 6. External Interfaces

| Interface | Direction | Notes |
|---|---|---|
| Stooq / yfinance / Alpaca data APIs | In | Licensing caveats in [Data Requirements](06-data-requirements.md) |
| Bank of Canada Valet API | In | Official, free FX rates |
| Broker REST API (adapter) | In/Out | Official APIs only; rate limits honored; never any money-movement scope |
| Telegram Bot API | Out | Alerts/commands; chat allow-list |
| Healthchecks.io | Out | Heartbeat pings |
| Tailscale | Admin | No public inbound ports |

## 7. Design Constraints

1. **$0/month fixed cost** — every component must run on free tier or self-hosted
   hardware ([Cost Management](../06-operations/05-cost-management.md)).
2. **Single host** — one Linux host; no multi-node architecture.
3. **Python 3.12+** primary language ([ADR-001](../04-architecture/adr/ADR-001-python-primary-language.md)).
4. **Official broker APIs only** (RL-5).
5. **No third-party capital** (RL-1) — single-account by design.
6. **Free/open-source dependencies only**, licenses verified at adoption
   ([Tech Stack](../04-architecture/02-tech-stack.md)).
7. **Solo-maintainable** — ops burden ≤ 30 min/week steady-state.

## 8. Assumptions & Dependencies

| # | Assumption |
|---|---|
| A-1 | Free-tier data sources remain available and usable for personal research/paper trading within their terms. |
| A-2 | A broker API (Wealthsimple Trade API or an equivalent CIRO-member broker) is available by G3; otherwise live phase slips (never ships without one). |
| A-3 | The host (OCI Always Free or home server) provides ≥ 99% effective availability during trading windows; degradations are alerted. |
| A-4 | Broker does not charge commissions on traded universe (verify per broker; model costs if not). |
| A-5 | NYSE calendar (with holidays) governs sessions; strategy operates monthly, so timezone/latency sensitivity is minimal. |

## 9. Requirement Identification & Status Conventions

| Prefix | Domain | Doc |
|---|---|---|
| `FR-DA` | Data ingestion | [Functional](02-functional-requirements.md) §2 |
| `FR-ST` | Strategy engine | §3 |
| `FR-BT` | Backtesting | §4 |
| `FR-EX` | Execution | §5 |
| `FR-RK` | Risk management | §6 |
| `FR-RC` | Reconciliation | §7 |
| `FR-PF` | Portfolio & accounting | §8 |
| `FR-NT` | Notifications | §9 |
| `FR-CF` | Configuration & admin | §10 |
| `FR-TX` | Tax | §11 |
| `FR-AU` | Audit | §12 |
| `NFR-xx` | Non-functional | [NFR doc](03-non-functional-requirements.md) |
| `SEC-xx` | Security | [Security reqs](04-security-requirements.md) |
| `OPS-xx` | Operational | [Operational reqs](05-operational-requirements.md) |

- **Priority scheme:** MoSCoW (Must/Should/Could/Won't-in-MVP).
- **Status:** Proposed → Accepted (in this baseline) → Implemented (verified per
  [RTM](07-requirements-traceability-matrix.md)).

## 10. Acceptance & Traceability

Every Must requirement carries a verification artifact (unit test, integration
test, backtest gate, runbook, or monitoring check) recorded in the
[Requirements Traceability Matrix](07-requirements-traceability-matrix.md).
Stage-gate criteria that depend on requirements are defined in the
[Roadmap](../08-planning/01-roadmap.md).

## 11. Change Management

Requirements changes follow: (1) proposal via issue; (2) update to the relevant
requirements doc + version bump; (3) ADR if architectural
([ADR process](../04-architecture/adr/README.md)); (4) RTM update. Changes to
`Must` security/risk requirements additionally require a threat-model review
trigger ([Secure SDLC §5](../05-security/04-secure-development-lifecycle.md)).
