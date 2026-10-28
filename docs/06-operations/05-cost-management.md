# Cost Management Policy (The $0 Constraint)

| | |
|---|---|
| **Document ID** | DOC-032 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | NFR-24 · OPS-13/14 · [ADR-005](../04-architecture/adr/ADR-005-zero-cost-hosting.md) · [Business Plan §7](../00-executive/03-business-plan.md) |

## 1. The Rule

**Fixed operating cost is $0.00/month.** Not "low" — zero. The only money at
risk in this venture is trading capital. Any charge anywhere is a SEV2
incident (OPS-13): investigate, remediate, document.

## 2. Free-Tier Inventory (tracked limits, OPS-14)

| Resource | Provider & tier | Known limits / risks |
|---|---|---|
| Host | **Oracle Cloud Always Free** ARM | 4 OCPU / 24 GB RAM, ~200 GB storage; **idle instances may be reclaimed**; requires heartbeat-visible activity; regional capacity at signup may vary |
| Fallback host | Home server (OQ-03) | Hardware already owned; power paid regardless |
| Admin network | **Tailscale free** | 100 devices / 3 users (we use ~3) |
| Heartbeat | **Healthchecks.io free** | ~20 checks (we use ~5) |
| Dashboards | **Grafana Cloud free** *or* self-hosted (OQ-08) | Free-tier series/retention limits — verify at setup |
| CI/CD | **GitHub Actions** | Public repo = free CI minutes (ADR-008) |
| Alerts | **Telegram Bot API** | Free |
| EOD data | **Stooq** / **yfinance** / **Alpaca free** | Personal-use terms; rate limits; US-listed only (ADR-004) |
| FX rates | **Bank of Canada Valet** | Official, free |
| Password vault | **Bitwarden free** | Unlimited personal vault |
| Backups | restic → home server / second free target | Storage must be pre-owned or free-tier |

## 3. Monthly Cost Review (part of the weekly ops review cadence — OPS-13)

Checklist (~10 minutes, monthly):

1. OCI billing page shows **$0.00** (check **before** reading anything else —
   the "what did I forget" check catches upgraded instances).
2. No GitHub Actions overage (public repo shouldn't produce any).
3. Healthchecks / Tailscale / Grafana tiers still free; no trial → paid
   conversions pending.
4. Free-tier **policy changes**: skim provider changelogs for the four
   load-bearing tiers (OCI, Tailscale, Healthchecks, Grafana).
5. Update §2 if any limit terms changed; adjust usage or note the risk to the
   [Risk Register](../08-planning/02-risk-register.md) (RSK-06).

## 4. Cost Governance Rules

1. **No credit card required ≠ safe** — trials convert, tiers change. Every
   new service must state its free-tier terms in §2 before adoption (SDLC
   trigger: new external service).
2. Instance sizes/config stays inside Always Free shapes — any "small" paid
   upgrade is a P4-level decision, not a convenience.
3. If a free tier dies: fall back (home server / alternative free tier) or
   accept downtime — **downtime is the correct cheap response** (fail-closed
   posture), never a surprise bill.

## 5. Legitimate Future Spend (all gated, none approved today)

| Spend | Gate |
|---|---|
| Canadian securities counsel | P4 commercialization decision (OQ-07) |
| Incorporation | P4 (Business Plan §9) |
| Paid market data (e.g., TSX real-time) | OQ-05, must also survive P4 review |
| Paid infra (if free tiers vanish) | Re-plan the venture economics; explicit founder decision |
| Hardware (fallback server) | OQ-03 — only from equipment already owned |

## 6. Cost Ledger

| Month | Total fixed cost | Notes |
|---|---|---|
| 2026-10 (docs phase) | **$0** | No infrastructure exists yet |
| — | — | *(append monthly)* |
