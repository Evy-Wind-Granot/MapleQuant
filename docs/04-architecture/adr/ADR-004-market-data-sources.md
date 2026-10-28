# ADR-004: Market Data — Free Multi-Source with Licensing Caveats

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-10-27 |
| **Deciders** | Founder |

## Context

The $0 constraint (NFR-24) rules out paid feeds. Requirements: ~10 years of
corporate-action-adjusted EOD for 8–12 liquid US-listed ETFs; official FX
rates for CAD translation; a trading calendar. Real-time TSX data is paid, so
Canadian-listed universes are out of scope for Phase 1 (see
[Market Analysis §5](../../01-market-compliance/01-market-analysis.md)).

## Decision

Adopt a **multi-source stack with explicit roles and per-bar provenance**:

| Role | Source |
|---|---|
| Primary EOD (US-listed) | **Stooq** bulk downloads (no key) |
| Spot-check / secondary EOD | **yfinance** (personal research/paper only — ToS gray zone, never a commercial dependency) |
| Intraday + paper-mode data | **Alpaca market data** (free tier) |
| FX (official, tax-grade) | **Bank of Canada Valet API** |
| Calendar | **exchange_calendars** (license verified at adoption) |

Every source's licensing terms are recorded and re-reviewed before any
non-personal use ([Data Requirements §7](../../03-requirements/06-data-requirements.md));
per-bar provenance prevents silent source blending; validation rules
(FR-DA-02/03) make data quality fail closed.

## Consequences

**Positive:** $0 data cost; redundancy across independent sources; official
FX for CRA-grade translation.

**Negative:** EOD-only depth for backtesting; yfinance fragility/rate limits;
no Canadian listings in v1; Stooq/yfinance terms could change — personal-use
posture is documented, not guaranteed.

**Mitigations:** adapters per source (same pattern as brokers); staleness
gates make bad data safe (fail-closed); licensing review is a P4 gate item
(OQ-05/OQ-07).

## Alternatives Considered

| Alternative | Rejected because |
|---|---|
| Single source (yfinance only) | Single point of failure; unverified quality; ToS gray |
| Polygon / other paid EOD | Breaks NFR-24 for a monthly-cadence strategy that doesn't need it |
| TMX/TSX real-time | Paid; the reason Canadian listings are deferred |
