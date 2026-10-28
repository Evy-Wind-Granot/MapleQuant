# Competitive Landscape

| | |
|---|---|
| **Document ID** | DOC-005 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [Market Analysis](01-market-analysis.md) · [Tech Stack](../04-architecture/02-tech-stack.md) |

> Pricing/licensing details below are directional as of writing; **verify before
> relying on any of them** (see [Open Questions](../09-appendix/03-open-questions.md)).

## 1. Competitive Matrix

| Competitor | Model & pricing | Strengths | Weaknesses / gaps | Relevance |
|---|---|---|---|---|
| **QuantConnect (LEAN)** | Cloud subscription tiers; engine open-source (Apache-2.0) | Mature multi-asset engine; broker integrations; large community | Data costs; US-centric; live trading requires paid cloud or self-hosting complexity; CAD/tax blind | Reference for backtest-engine design; possible library reuse |
| **NautilusTrader** | OSS (LGPL-3.0) | Professional-grade event-driven core; high performance; crypto+FX focus | Steep learning curve; geared to sophisticated users | Design inspiration for event-driven core ([ADR-003](../04-architecture/adr/ADR-003-single-codepath-event-driven-core.md)) |
| **Backtrader** | OSS (MIT — verify) | Simple, mature, well-documented backtesting | Aging maintenance; live-broker support dated; no CAD/tooling story | Candidate backtest library for Phase 1 evaluation |
| **Freqtrade** | OSS (MIT); crypto only | Best-in-class OSS bot ops model (Telegram control, dry-run, web UI) | Crypto exchanges only — out of scope | **Ops-pattern reference** (dry-run mode, kill switch ergonomics) |
| **Composer.trade** | VC-backed SaaS (US) | No-code strategy building; registered entity handles execution | US-only; closed; subscription; not broker-agnostic | Model of "no-code" future segment (S3) |
| **Interactive Brokers** | Broker-native API + fees | Deepest API; global markets | Fee structure, complexity, poor UX; still broker-locked | Fallback broker path if Wealthsimple access stalls |
| **Wealthsimple (built-in)** | Broker app features (recurring buys, robo "Invest") | Trust, zero commissions, Canadian | No general automation/API for retail (Trade API waitlisted — verify, OQ-01) | Target integration; complementary rather than competing |
| **Robo-advisors generally** | Managed, fee-based % AUM | Zero effort | Opaque; fees; not self-directed | **Not competitors** — MapleQuant is the self-directed alternative (see [Product Overview](../02-product/01-product-overview.md)) |

## 2. Positioning Axes

```
                Broker-agnostic
                      ▲
                      │
        MapleQuant ●  │   QuantConnect
                      │   NautilusTrader
        Freqtrade    │        Backtrader
        (crypto)     │
   ──────────────────┼──────────────────────►  Hand-holding / managed
     Code-first      │        Composer, robo-advisors,
                      │        Wealthsimple Invest
                 Broker-locked
```

- **Axis 1 — Code-first ↔ No-code:** MapleQuant serves code-first users (S1/S2) with plain config files and full inspectability.
- **Axis 2 — Broker-agnostic ↔ Broker-locked:** MapleQuant's adapter architecture ([ADR-002](../04-architecture/adr/ADR-002-broker-agnostic-adapter-architecture.md)) keeps the engine independent of any broker.
- **Axis 3 (not drawn) — $0 ↔ $$:** MapleQuant is the only entrant architected around a hard $0 fixed cost.

## 3. Differentiation & Honest Assessment

| Claimed edge | Honest verdict |
|---|---|
| Canadian localization (ACB/CAD/CIRO adapters) | **Real and rare** among code-first tools |
| $0 fixed operating cost | Real but defensible by anyone; the *discipline* is the differentiator more than the mechanism |
| Broker-agnostic execution | LEAN also does this; our narrower scope makes it simpler and more robust for retail use |
| Backtest validation rigor | Engines are commodities; *documented standards + stage gates* are genuinely uncommon |
| Security documentation for retail bot ops | Rare in OSS personal projects; table stakes for any future product |

**Strategic implication:** the moat is **localization + trust (auditability,
security, validation discipline)**, not raw engine capability. This is why the
documentation set you are reading exists at all.

## 4. Threat Assessment (if commercialized)

| Threat | Severity | Response |
|---|---|---|
| A large broker ships retail automation natively (e.g., Wealthsimple native) | High | Narrow but real: they serve *their* clients; MapleQuant stays broker-agnostic |
| LEAN/OSS adds polished Canadian tax/broker support | Medium | Compete on simplicity + validation discipline; consider open-core collaboration |
| Regulatory tightening of retail automation APIs | Medium | Compliance-first posture; official-API-only policy; counsel relationship |
| Paid-data requirement for any serious product | High at P4 | Acknowledged: SaaS would break the $0 constraint and is gated accordingly |
