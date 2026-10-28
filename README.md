# MapleQuant — Startup Documentation Baseline

> **Status:** v0.1 (Draft) — complete planning baseline, created 2026-10-27, pre-development.
> **Nothing has been built yet.** That is deliberate: this repository is the
> "measure twice" before a single line of engine code exists ([ADR-008](docs/04-architecture/adr/ADR-008-repository-strategy.md)).

**MapleQuant** (placeholder name — trademark search pending, OQ-04) is a
**zero-fixed-cost, brokerage-agnostic automated trading platform**. Phase 1 is
a personal system trading the Founder's own self-directed brokerage account
(currently Wealthsimple) after a staged research → paper → live-capped
pipeline.

> ⚠️ **Disclaimers:** Nothing here is investment, legal, or tax advice.
> The regulatory perimeter (personal-use self-directed trading only) is
> defined in [Regulatory & Compliance](docs/01-market-compliance/03-regulatory-compliance.md).
> Commercialization is hard-gated behind securities counsel (P4).

---

## The One-Paragraph Plan

Document everything → research and validate a strategy against strict
[standards](docs/07-quality/02-backtesting-validation-standards.md) (G1) →
60 days of clean paper trading (G2) → live trading capped at CAD 2,000 under
supervision (G3) → decide the venture's future with a full evidence pack (G4).
At every step: fail-closed engineering, $0/month fixed cost, official broker
APIs only, and audit-grade records.

## Reading Order (if you're new — including future-you)

| # | Start with | Why |
|---|---|---|
| 1 | [Executive Summary](docs/00-executive/01-executive-summary.md) | The whole venture in 2 pages |
| 2 | [Product Overview](docs/02-product/01-product-overview.md) | What this is / is not |
| 3 | [System Architecture](docs/04-architecture/01-system-architecture.md) | How it's shaped (esp. §7 fail-closed) |
| 4 | [Functional Requirements](docs/03-requirements/02-functional-requirements.md) | What it must do (68 FRs, 7 invariants) |
| 5 | [Regulatory & Compliance](docs/01-market-compliance/03-regulatory-compliance.md) | Red lines that keep this legal |
| 6 | [Roadmap](docs/08-planning/01-roadmap.md) | G0→G4 gates and dates |
| 7 | [Open Questions](docs/09-appendix/03-open-questions.md) | What's deliberately undecided |

## Document Map

```
maplequant/
├── README.md                      ← you are here
├── .gitignore
├── templates/                     (ADR + runbook templates)
└── docs/
    ├── 00-executive/              Executive summary · Vision/values · Business plan
    ├── 01-market-compliance/      Market analysis · Competition · Regulatory (Canada)
    ├── 02-product/                Product overview · PRD (personas, user stories)
    ├── 03-requirements/           SRS · 68 functional · NFR · security · ops · data · RTM
    ├── 04-architecture/            System architecture · Tech stack · Adapter spec ·
    │   │                          Strategy/Risk/Backtest designs · 8 ADRs
    │   └── adr/
    ├── 05-security/               Overview · Threat model · Secrets · SDLC · Incident response
    ├── 06-operations/             Deployment · Monitoring · Backup/DR · 8 runbooks · Cost policy
    ├── 07-quality/                 Testing strategy · Backtest validation standards · Release mgmt
    ├── 08-planning/                Roadmap · 18-risk register · Success metrics + stop rules
    └── 09-appendix/                Glossary · Document control · Open questions
```

Full register with IDs and review cadence: [Document Control](docs/09-appendix/02-document-control.md).

## Core Conventions

- **Requirement IDs:** `FR-<domain>-NN` (functional), `NFR-NN`, `SEC-NN`,
  `OPS-NN`; **invariants** INV-1…7; **red lines** RL-1…7; risks RSK-NN;
  runbooks RB-NNN; gates G0–G4; open questions OQ-NN.
- **Docs are versioned** (vMAJOR.MINOR) with status Draft / In Force; changes
  to Must requirements or red lines get a change-log entry.
- **This repo stays private** (ADR-008): business planning, strategy
  specifics, and personal operational detail live here. The engine code will
  live in a separate public repository.

## Non-Negotiables (the compressed version)

1. **$0.00/month fixed cost** (NFR-24) — any charge is an incident.
2. **Fail-closed everywhere** (INV-1) — uncertainty = no orders + alert.
3. **Official broker APIs only** (RL-5) — never reverse-engineered endpoints.
4. **No third-party money, no advice distribution, no performance claims** (RL-1/3/4).
5. **The broker is the source of truth** (INV-4); the risk gate is unavoidable (INV-2).
6. **Nothing trades live without passing its gates** — and *not trading is a valid output*.

## What Happens Next

1. G0 self-review pass of this baseline → mark documents **In Force**.
2. Create the public engine repo (`maplequant-engine`) with CI gates from day one.
3. Phase P0 build work per the [Roadmap](docs/08-planning/01-roadmap.md).
4. First commit of this repo = the baseline itself.
