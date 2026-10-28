# Market Analysis

| | |
|---|---|
| **Document ID** | DOC-004 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [Competitive Landscape](02-competitive-landscape.md) · [Business Plan](../00-executive/03-business-plan.md) · [Data Requirements](../03-requirements/06-data-requirements.md) |

> **Note:** This analysis is deliberately qualitative. Any hard market-size
> figures will be sourced and cited only if/when the P4 commercialization
> decision requires them (see [Open Questions](../09-appendix/03-open-questions.md), OQ-07).

## 1. Market Context

- **Self-directed investing in Canada has grown sharply since 2020.** Zero-commission
  trading (pioneered in Canada by Wealthsimple) removed the per-trade cost barrier
  and normalized retail equity participation. Current user figures for major
  Canadian platforms are in the millions — *verify current numbers before any
  external use of this document.*
- **Retail brokers are opening programmatic access.** The historical blocker for
  retail automation in Canada — no official broker API — is eroding: Wealthsimple
  has announced a waitlisted retail Trade API (status must be verified, OQ-01),
  while US brokers such as Alpaca have offered free APIs for years.
- **The tooling gap persists.** Canadian retail traders who want systematic
  strategies must choose between: US-centric platforms with poor CAD/tax support,
  professional tools priced for institutions, aging open-source libraries, or
  manual execution.

## 2. Target Segments

| Segment | Description | Relevance |
|---|---|---|
| **S1 — Founder (now)** | Solo technical self-directed investor trading their own account | The only segment served in Phases 0–3 |
| **S2 — Technical self-directed traders** | Developers/engineers who write or adapt their own strategies; want control, low cost, Canadian tax/ACB support | Primary P4 candidate |
| **S3 — Quant-curious investors** | Non-coders interested in transparent, inspectable strategies | Secondary P4 (education/content track) |
| **S4 — Professional quants** | Need tick data, low latency, institutional brokers | **Out of scope permanently** (different product class, paid-data economics) |

## 3. Needs Analysis (S2/S3, if commercialized)

| Need | Current satisfaction | Gap |
|---|---|---|
| Automate my own brokerage account | Poor — few Canadian brokers expose retail APIs; tools assume US brokers | **Core gap** |
| Pay ~nothing to run it | Poor — cloud quant platforms are subscription-priced | MapleQuant wedge |
| Canadian tax-aware record-keeping (ACB, FX-translated CAD) | Poor — US platforms ignore Canadian tax mechanics | **Differentiator** |
| Broker-agnostic architecture (switch brokers, multi-broker) | Poor — most tools lock to one broker or charge per integration | Differentiator |
| Trustworthy backtesting without lookahead/survivorship bias | Mixed — engines exist, but discipline is rare; few enforce validation standards | Differentiator via [Backtesting Validation Standards](../07-quality/02-backtesting-validation-standards.md) |
| Institutional-grade risk controls for small capital | Poor — risk tooling assumes pro workflows | Differentiator |
| Security posture for broker credentials | Poor — few personal tools document one | Differentiator via [Security docs](../05-security/01-security-overview.md) |

## 4. Market Drivers

1. Zero-commission norm → turnover costs no longer dominate small accounts.
2. Broker API opening (Wealthsimple Trade API; Alpaca precedent) → execution automation feasible for retail.
3. Free-tier cloud + OSS quant stack → $0 fixed-cost operations viable.
4. AI-assisted development → solo founders can build to a professional documentation/engineering standard.
5. Growing distrust of opaque "managed" products → demand for inspectable self-directed automation.

## 5. Barriers & Headwinds

| Barrier | Impact | Mitigation |
|---|---|---|
| Canadian market data licensing (real-time TSX is paid) | Constrains universe to US-listed securities under $0 rule | Scope Phase 1 to US-listed ETFs (liquid, free EOD/intraday data); TSX support deferred (OQ-05) |
| FX friction on US-listed holdings (broker FX conversion fees; verify current Wealthsimple fee, historically ~1.5%) | Drag on returns; forces strategy design to minimize FX round-trips | USD sub-account usage; FX-aware cost modeling in backtests |
| Regulatory ambiguity for retail automation | Slows any future productization | [Compliance red lines](03-regulatory-compliance.md) now; counsel gate at P4 |
| Broker ToS restrictions on programmatic access | Could block the Wealthsimple integration | Use official API only when granted; never reverse-engineered endpoints; broker-agnostic adapters preserve optionality |
| Free-tier reclamation/degradation | Operational risk to the platform | Fallback host + [monitoring](../06-operations/02-monitoring-and-alerting.md); monthly cost review |

## 6. Canada-Specific Factors (the Localization Wedge)

1. **CIRO-member broker adapters** (Wealthsimple first) vs. everyone else's US-broker assumptions.
2. **CAD as base currency** with USD holdings; official Bank of Canada FX rates for tax translation.
3. **Adjusted Cost Base (ACB)** tracking and capital gains exports for CRA reporting; superficial-loss awareness.
4. **America/Toronto ops calendar**; TSX/NYSE holiday handling.
5. **Canadian data-licensing realities** documented honestly (see [Data Requirements](../03-requirements/06-data-requirements.md)).

## 7. Conclusion

The near-term market is a market of one (the Founder), by design. If MapleQuant
ever commercializes, its defensible position is **not** engine sophistication —
open-source engines already exist — but the combination of **Canadian
localization, radical $0 cost discipline, broker-agnostic execution, and a
documented, auditable engineering standard**. That combination is currently
unoccupied.
