# Executive Summary

| | |
|---|---|
| **Document ID** | DOC-001 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [Vision, Mission & Values](02-vision-mission-values.md) · [Business Plan](03-business-plan.md) · [Roadmap](../08-planning/01-roadmap.md) |

> **Company name note:** "MapleQuant" is a placeholder working title. A name/trademark
> search is required before any public or commercial use (see
> [Open Questions](../09-appendix/03-open-questions.md), OQ-04).

## 1. What MapleQuant Is

MapleQuant is a **bootstrapped, zero-cost, brokerage-agnostic automated trading
platform**. In its first phase it is a **personal trading system**: a rules-based
engine that researches, backtests, and — after a supervised validation period —
executes systematic equity/ETF strategies on the Founder's own self-directed
brokerage account.

It is explicitly **not** a robo-advisor, not a fund, not a signal-selling service,
and not high-frequency trading. Those distinctions are not just positioning — they
determine the regulatory perimeter (see
[Regulatory & Compliance](../01-market-compliance/03-regulatory-compliance.md)).

## 2. The Opportunity

- Self-directed retail investing in Canada has grown rapidly since 2020, yet
  Canadian retail traders are poorly served by algorithmic tooling: the dominant
  platforms (QuantConnect, Composer, IBKR tooling) are US-centric, priced for
  professionals, or both.
- Modern retail brokers are beginning to expose official trading APIs
  (Wealthsimple announced a waitlisted Trade API; Alpaca has offered one for
  years), making "bring-your-own-broker" automation newly practical for a
  self-directed investor.
- The cost floor for running such a system has collapsed: open-source Python
  tooling, free-tier cloud, free market data (with documented caveats), and free
  monitoring/alerting make a **$0/month fixed-cost operation realistic**.

## 3. The Product (Phase 1)

A single-host Python platform that:

1. Ingests market data from free sources (with explicit data-licensing caveats).
2. Backtests systematic strategies against historical data using a shared
   event-driven core (the same code path as live trading).
3. Paper-trades a validated strategy for **60 consecutive trading days**.
4. On passing all stage gates, trades a small, capped allocation of real capital.
5. Continuously enforces hard risk limits, a kill switch, daily reconciliation
   against the broker, and audit-grade logging of every decision.

The initial strategy family is **low-frequency ETF momentum/rotation**
(monthly rebalance; see
[Strategy Engine Design](../04-architecture/04-strategy-engine-design.md)) — chosen
because it fits free data, small capital, low turnover, and strict validation
standards.

## 4. Hard Constraints

| Constraint | Meaning |
|---|---|
| **$0 fixed cost** | Infrastructure, software, and data must cost nothing per month. Trading capital is the only money at risk. Enforced by the [Cost Management Policy](../06-operations/05-cost-management.md). |
| **Broker-agnostic** | No broker-specific logic outside adapter modules. Current broker relationship: the Founder holds a Wealthsimple account; integration depends on official API access (OQ-01). Paper trading uses Alpaca's free paper API. |
| **Compliance red lines** | Personal capital only. No managing third-party money, no fee/performance-based services, no advice distribution, without prior securities-law counsel. |
| **Solo operation** | Designed for a one-person team: heavy automation, fail-closed defaults, modest operational load (<30 min/week steady-state). |

## 5. Why Now / Why Us

- **Why now:** retail broker APIs are opening; free-tier infrastructure is
  mature; the founder is building during a docs-first planning window with no
  capital committed.
- **Why us:** the wedge is Canadian localization (CAD accounting, adjusted cost
  base for taxes, CIRO-member broker adapters, America/Torizon ops calendar)
  combined with radical cost discipline — a defensible niche against US-centric
  incumbents if the project ever commercializes.

## 6. Stage-Gate Plan

| Gate | Criterion | Target |
|---|---|---|
| **G0** | Documentation set complete; architecture and requirements baselined | 2026-10 |
| **G1** | Strategy v1 passes backtesting validation standards | 2026-12 |
| **G2** | 60 consecutive trading days of clean paper trading | 2027-04 |
| **G3** | Live trading begins with a CAD 2,000 cap, supervised for 90 days | 2027-04 → 2027-07 |
| **G4** | Scale-up / commercialization decision (requires legal counsel) | 2027-Q4 |

Full detail: [Roadmap](../08-planning/01-roadmap.md).

## 7. Business Model Outlook

The venture is intentionally **not revenue-focused in Phase 1**. Documented
options, in the order they would be pursued:

1. **Personal tool** — $0 revenue, $0 cost, founder benefit and skill-building.
2. **Open-source core** — community adoption, sponsorship funding, credibility.
3. **Content/education** — low regulatory risk, low cost.
4. **SaaS (bring-your-own-broker)** — highest regulatory exposure; requires
   registration analysis by securities counsel, incorporation, and PIPEDA
   compliance. Not pursued without explicit legal advice.

Analysis: [Business Plan](03-business-plan.md).

## 8. Team & Ask

Solo founder (technical), assisted by AI-driven development. **No funding ask.**
The only anticipated non-free expenditures — legal counsel and incorporation —
are gated behind a commercialization decision and are explicitly out of scope
until then.

## 9. Top Risks (Summary)

1. **Strategy risk** — overfitting/market regime change (mitigated by validation
   standards and stage gates).
2. **Security risk** — broker API credential compromise (mitigated by
   trade-only scopes, encrypted storage, revocation runbooks).
3. **Regulatory risk** — only if commercializing (mitigated by counsel-before-revenue).
4. **Infra risk** — free-tier reclamation/degradation (mitigated by fallback host).

Full register: [Risk Register](../08-planning/02-risk-register.md).
