# Regulatory & Compliance (Canada)

| | |
|---|---|
| **Document ID** | DOC-006 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [Business Plan](../00-executive/03-business-plan.md) · [Functional Requirements](../03-requirements/02-functional-requirements.md) · [Risk Register](../08-planning/02-risk-register.md) |

> ## ⚠️ DISCLAIMER — NOT LEGAL ADVICE
> This document is an engineering-level risk assessment written by a
> non-lawyer. It is **not** legal, tax, or investment advice, and it does not
> create a compliance opinion. Securities legislation is fact-specific;
> **qualified Canadian securities counsel must be consulted before any activity
> beyond personal self-directed trading of the Founder's own capital.**

## 1. Regulatory Framework (Overview)

| Body / instrument | Role | Relevance to MapleQuant |
|---|---|---|
| Provincial Securities Acts (administered by e.g. OSC, BCSC, AMF) | Primary legislation; define regulated activities and exemptions | Registration triggers live here (advice, trading for others) |
| CSA (Canadian Securities Administrators) | Harmonization of provincial rules | Relevant to any future national product |
| CIRO (Canadian Investment Regulatory Organization, formed 2023 from IIROC + MFDA) | Self-regulatory organization for investment dealers (e.g., Wealthsimple) | Governs the *broker's* obligations; the bot interacts with a CIRO dealer as a client |
| Broker API terms of service | Contractual limits on programmatic access | Directly binds how the platform may interact with the account |
| PIPEDA / provincial privacy law | Personal information handling | Only engaged at commercialization (P4) |
| Income Tax Act (CRA) | Capital gains, record-keeping (6-year retention) | Platform must produce tax-ready records (FR-TX) |

## 2. Personal-Use Analysis (Phases 0–3)

**Activity:** a computer program, written and operated by the account holder,
executing pre-programmed rules on the account holder's own self-directed
brokerage account, for the account holder's sole benefit.

Key points (engineering-level understanding, to be confirmed by counsel if scope
ever changes):

1. **Trading one's own account is not a regulated activity** in the way that
   trading *for others* is. Self-directed investors do not register to place
   their own orders.
2. **No advice is given to any other person.** The strategy rules are the
   Founder's own decisions, pre-programmed. Advice *distribution* is what
   triggers adviser registration — none occurs.
3. **No third-party capital.** No pooling, no mandates, no performance fees.
   This is the single most important structural red line.
4. **The broker relationship governs execution.** All order flow must go
   through official, broker-sanctioned interfaces:
   - ❌ **Prohibited:** reverse-engineered private endpoints, scraping, browser
     automation, credential-sharing with unapproved services. These violate
     broker ToS, risk account termination, and create security exposure
     ([Threat Model](../05-security/02-threat-model.md)).
   - ✅ **Required:** official API access (e.g., Wealthsimple Trade API when
     granted) or sanctioned paper/simulation environments (Alpaca paper).
5. **Market-abuse rules apply to everyone.** Manipulative or deceptive trading
   (e.g., spoofing/layering, wash trading) is prohibited for all participants.
   Phase 1 strategies are low-frequency, marketable-order momentum strategies —
   structurally far from these practices — and the execution engine must never
   introduce deceptive order patterns (enforced by design review, see
   [Secure SDLC](../05-security/04-secure-development-lifecycle.md)).

## 3. Compliance Red Lines (Non-Negotiable)

| # | Red line | Why |
|---|---|---|
| RL-1 | No trading or managing of any third party's money (incl. family/friends), no pooled accounts | Portfolio-manager / dealer registration territory |
| RL-2 | No fees, performance-based compensation, or profit-sharing from anyone's trading | Registration trigger; advisor activity |
| RL-3 | No selling of signals, alerts, or tailored recommendations | Risk of being deemed investment advice |
| RL-4 | No public marketing of performance results or "returns" claims | Performance advertising rules (registered-firm regime); reputational risk |
| RL-5 | No use of unofficial/reverse-engineered broker interfaces | ToS violation; account termination; security risk |
| RL-6 | No automation of any withdrawal/money-movement capability | Out of trading scope entirely; principal security control ([SEC-02](../03-requirements/04-security-requirements.md)) |
| RL-7 | Any scope change beyond personal use requires prior counsel review | Gate condition; see §4 |

## 4. Commercialization Gate (P4)

Before any revenue-bearing activity (SaaS, sponsorships bundled with services,
paid content making performance claims, etc.):

1. **Engage Canadian securities counsel** for a registration analysis covering:
   the "advice" definition, dealer vs. software-vendor status, robo-advisor
   precedents, and provincial variations.
2. **Incorporate** (breaks the $0 constraint — requires an approved decision).
3. Implement PIPEDA-compliant privacy practices, ToS, and user disclaimers.
4. Re-run the [Threat Model](../05-security/02-threat-model.md) for third-party
   data (a materially different threat surface).

## 5. Tax Obligations (Personal)

| Obligation | Platform support |
|---|---|
| Capital gains reporting on the Founder's account | FR-TX-01: export tax-ready trade records |
| **Adjusted Cost Base (ACB)** computation incl. reinvested dividends, return of capital | FR-PF-03: accounting engine maintains per-lot ACB |
| CAD translation of USD transactions (official Bank of Canada daily rates) | FR-DA-05: FX ingestion (BoC Valet API) |
| Records retention (CRA: generally 6 years) | FR-AU-01: immutable audit log retention ≥ 7 years |
| Superficial loss rule awareness | Accounting module flags identical-property repurchases within 30 days (FR-PF-04) |

*Not tax advice; confirm treatment with a qualified accountant as volume grows.*

## 6. Broker ToS Compliance Tasks

| ID | Task | Owner | Due |
|---|---|---|---|
| CT-01 | Verify Wealthsimple Trade API program status and request access (waitlist) | Founder | At P2 start (OQ-01) |
| CT-02 | Read and archive the API terms when granted; extract automation-relevant clauses into this doc | Founder | Within 1 week of access |
| CT-03 | Verify Alpaca paper-trading ToS for simulation usage | Founder | At P1 start |
| CT-04 | Confirm wealthsimple FX fee schedule (historically ~1.5%) and USD sub-account mechanics | Founder | Before G1 cost model freeze |
| CT-05 | Name/trademark search for "MapleQuant" before any public use | Founder | Before any public artifact (OQ-04) |
| CT-06 | Annual re-read of this document + red lines | Founder | Yearly |
| CT-07 | Counsel engagement **before** any activity beyond personal use | Founder | P4 gate |
| CT-08 | Verify data-source licensing terms for any non-personal usage (see [Data Requirements](../03-requirements/06-data-requirements.md)) | Founder | Before any public/data redistribution |

## 7. Compliance posture summary

- **Now (Phases 0–3):** personal-use self-directed trading via official
  interfaces only; red lines RL-1…RL-7 enforced; compliance-relevant features
  (audit, ACB, records) built as first-class requirements.
- **Later (P4):** nothing public, paid, or third-party-facing happens without
  counsel — this is a hard gate in the [Roadmap](../08-planning/01-roadmap.md)
  and the [Risk Register](../08-planning/02-risk-register.md) (RSK-03).
