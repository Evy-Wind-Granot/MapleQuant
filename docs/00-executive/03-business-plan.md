# Business Plan

| | |
|---|---|
| **Document ID** | DOC-003 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [Executive Summary](01-executive-summary.md) · [Market Analysis](../01-market-compliance/01-market-analysis.md) · [Regulatory Compliance](../01-market-compliance/03-regulatory-compliance.md) · [Cost Management](../06-operations/05-cost-management.md) |

## 1. Business Overview

| Item | Description |
|---|---|
| Entity | Unincorporated personal project (sole proprietorship by default). Incorporation deferred until a revenue path is approved (OQ-07). |
| Working name | MapleQuant (placeholder — trademark/name search pending, OQ-04) |
| Sector | Fintech / retail trading infrastructure |
| Product | Automated trading platform (software), initially operating on the Founder's own capital only |
| Geography | Canada (Founder's brokerage is Wealthsimple, a CIRO investment dealer) |
| Funding | Bootstrapped; fixed operating cost **$0/month** |

## 2. Strategic Phases

| Phase | Period | Objective | Revenue |
|---|---|---|---|
| P0 — Foundation | 2026-10 → 2026-11 | Documentation baseline, repo scaffolding, data layer | $0 |
| P1 — Research | 2026-11 → 2026-12 | Backtest engine + strategy v1 validated (G1) | $0 |
| P2 — Validation | 2027-01 → 2027-04 | 60 trading days of paper trading (G2) | $0 |
| P3 — Live (capped) | 2027-04 → 2027-07 | Live trading, CAD 2,000 cap, 90-day supervision | $0 |
| P4 — Decision | 2027-Q4 | Commercialization / open-source / status-quo decision | TBD |

## 3. Business Model Options

| # | Model | Revenue mechanics | Cost | Regulatory exposure | Verdict |
|---|---|---|---|---|---|
| 1 | **Personal tool** | None (personal returns) | $0 | None (self-directed personal account) | **Default.** |
| 2 | **Open-source core** (open-core) | GitHub Sponsors/donations; consulting | $0 (public repo, free CI) | Low — but license choice and "advice" framing matter | Strong candidate at P4; builds reputation at zero cost. |
| 3 | **Content/education** | Blog, courses, newsletter | ~$0 | Low — must avoid personalized advice | Optional parallel track. |
| 4 | **SaaS, bring-your-own-broker** | Subscriptions | Hosting + paid data licenses (breaks $0 constraint) | **High** — risk of being deemed advice/PM activity; PIPEDA; ToS | Only after securities counsel + incorporation. |

**Decision rule:** Options 2–3 can proceed with documentation and trademark
review. Option 4 is blocked behind [Regulatory Compliance](../01-market-compliance/03-regulatory-compliance.md) §4 (counsel gate).

## 4. Value Proposition

**For the Founder (now):** disciplined, emotion-free execution of a validated
strategy; institutional-style risk controls; deep learning in quant + ops +
security.

**For a future audience (P4):** the only automated-trading stack that is
(a) Canadian-localized (CAD, adjusted cost base, CIRO brokers),
(b) genuinely $0 to operate, (c) brokerage-agnostic, and
(d) documented to a professional engineering standard.

## 5. Market Summary

Target segment if commercialized: **technical self-directed retail investors in
Canada** who currently choose between US-centric platforms (QuantConnect,
Composer), broker-locked tools, or manual execution. See
[Market Analysis](../01-market-compliance/01-market-analysis.md) and
[Competitive Landscape](../01-market-compliance/02-competitive-landscape.md).

## 6. Go-to-Market (Deferred)

No GTM activity before P4. On a "go" decision, the sequence would be:
(1) open-source the engine core; (2) publish engineering/quant content from this
document set; (3) community building; (4) only then consider hosted/SaaS with
counsel. **No paid acquisition** (violates $0 constraint and pre-revenue logic).

## 7. Financial Plan

### 7.1 Operating budget (Months 1–12)

| Line | Budget | Notes |
|---|---|---|
| Infrastructure | **$0** | Oracle Cloud Always Free / home server, free monitoring, free CI |
| Software & data | **$0** | OSS licenses; free-tier data (see data licensing caveats) |
| Marketing | **$0** | None pre-P4 |
| Legal | **$0** (pre-P4) | Paid consult required before any commercialization |
| Incorporation | **$0** (deferred) | ~CAD 200–300 federal when required |
| **Total fixed** | **$0/month** | Hard policy: see [Cost Management](../06-operations/05-cost-management.md) |

### 7.2 Capital at risk

| Stage | Capital cap | Basis |
|---|---|---|
| Paper (P2) | Simulated only | — |
| Live (P3) | **CAD 2,000** | Disposable risk capital; sized so a 100% loss is an acceptable tuition |
| Post-supervision | Stepped increases, gated by [success metrics](../08-planning/03-success-metrics.md) | e.g., 2k → 5k → 10k with ≥ 2 clean quarters |

### 7.3 Break-even analysis

Not applicable pre-revenue. For models 2–3 at P4, revenue potential is
documented then. **The venture is not capital-intensive by design.**

## 8. Organization & Roles

| Role | Holder | Notes |
|---|---|---|
| Founder / Operator / Engineer / Compliance contact | Founder | All functions pre-P4 |
| Counsel (external, on-call) | Not engaged | Required before: managing any third-party money, selling advice/signals, SaaS |

## 9. Legal Structure Considerations (Canada)

- Currently a personal project; income (if any, e.g., sponsorships) would be
  self-employment income. CRA record-keeping: retain 6 years of records — the
  Platform's audit logs satisfy this for trading activity.
- Incorporation (federal, Corporations Canada) only at P4 if revenue-bearing
  activities are approved; costs break the $0 rule, hence the gate.
- The trading itself: capital gains on the Founder's own account; the Platform
  must export tax-ready trade records incl. **adjusted cost base (ACB)** —
  see [Functional Requirements](../03-requirements/02-functional-requirements.md) FR-TX.

## 10. Key Assumptions & Validations

| # | Assumption | Validation | Status |
|---|---|---|---|
| A1 | Free-tier infrastructure is reliable enough for a daily-cycle trading system | 60-day paper period uptime ≥ 98% of scheduled runs (G2 gate) | Pending |
| A2 | Free data sources are adequate for a monthly-rebalance ETF strategy | G1 validation + data quality checks | Pending |
| A3 | A broker API (Wealthsimple Trade API or equivalent) will be accessible by P3 | OQ-01 verification | Pending |
| A4 | Strategy v1 has positive expectancy after costs out-of-sample | Backtesting validation standards, G1 | Pending |
| A5 | Solo operation is sustainable (<30 min/week) | Measured during P2/P3 | Pending |

## 11. Risks (Business-Level)

Top items — full analysis in the [Risk Register](../08-planning/02-risk-register.md):

- Strategy underperformance / regime change (accepted; sized capital).
- Regulatory reclassification if scope creeps toward advice (controlled by red lines).
- Free-tier policy changes (monitored monthly via cost review).
- Wealthsimple API access unavailable or revoked (broker-agnostic adapters; paper/live can proceed on Alpaca if needed).
