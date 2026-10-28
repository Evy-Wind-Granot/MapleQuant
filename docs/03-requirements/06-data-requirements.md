# Data Requirements

| | |
|---|---|
| **Document ID** | DOC-014 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [ADR-004 Market Data Sources](../04-architecture/adr/ADR-004-market-data-sources.md) · [FR-DA group](02-functional-requirements.md) · [Backtesting Validation](../07-quality/02-backtesting-validation-standards.md) |

## 1. Data Domains

| Domain | Content | Cadence | Retention |
|---|---|---|---|
| Market EOD | OHLCV + volume + adjusted prices, per symbol | Daily post-close | Full history (Parquet cold store) |
| Market intraday (research) | Hourly / 15-min bars | On demand (paper/slippage studies) | Rolling 5 years |
| FX | CAD/USD (and pairs as needed) from Bank of Canada Valet | Daily (official) | Full history |
| Corporate actions | Splits, dividends/ROCs per symbol | Event-driven | Full history |
| Universe membership | Point-in-time listing/removal dates per symbol | Static + changes logged | Full history |
| Trading calendar | NYSE sessions/holidays (TSX when scope expands) | Annual refresh | Full history |

## 2. Sources (all free-tier; licensing caveats in §7)

| Source | Use | Access | Caveats |
|---|---|---|---|
| **Stooq** | Primary historical EOD (US-listed) | Bulk CSV downloads, no key | Personal-use terms; verify at adoption |
| **yfinance** | Secondary/spot-check EOD, quick research | Python lib (unofficial Yahoo endpoint) | Unofficial — **ToS gray area**; rate limits; quality varies; personal research/paper use only, never a commercial dependency |
| **Alpaca market data** | Intraday bars for paper-mode realism + recent EOD | REST API (free tier) | US markets only; IEX-based real-time (sufficient — we trade EOD/momentum); verify plan limits |
| **Bank of Canada Valet** | Official FX rates | REST API | Official; free; ideal for tax translation |
| **exchange_calendars** (lib) | NYSE/TSX calendars | Python lib | Verify license at adoption |

**Excluded by policy:** paid feeds (Polygon, TMX real-time), scraped/automated
site access, any source requiring redistribution of data, and any
reverse-engineered endpoint ([RL-5](../01-market-compliance/03-regulatory-compliance.md)).

## 3. Schemas (normative)

| Table/File | Key fields | Store |
|---|---|---|
| `bars_eod` | symbol, date, open, high, low, close, volume, adj_factor, source, fetched_at | Parquet (cold) + SQLite index |
| `fx_rates` | date, pair, rate, source | SQLite |
| `corporate_actions` | symbol, date, type, params | SQLite |
| `universe` | symbol, name, asset_class, listing_date, delisting_date?, currency, cap_tier | SQLite (versioned changes) |
| `positions`, `orders`, `fills`, `nav_history` | see [Risk & Portfolio Design](../04-architecture/05-risk-and-portfolio-design.md) | SQLite |
| `audit` | JSONL event stream (FR-AU-01) | JSONL files + hash chain |

## 4. Data Quality Rules (enforced by FR-DA-02/03)

| Rule | Action on violation |
|---|---|
| Schema conformance (pydantic) | Reject batch, alert |
| OHLC sanity: high ≥ open/close ≥ low, all > 0, volume ≥ 0 | Reject symbol-day, alert |
| Duplicate dates / non-monotonic time | Reject batch |
| Staleness: last bar older than tolerance for decision symbols | Mark stale → fail-closed session (FR-DA-03) |
| Price spike > 50% day-over-day (universe of ETFs) | Flag for review before use (no auto-trade on flagged data) |
| FX rate missing for a session date | Use last prior official rate, log the substitution |

## 5. Freshness Requirements

| Consumer | Requirement |
|---|---|
| Backtests | Frozen snapshots (FR-DA-09) — freshness N/A, reproducibility paramount |
| Paper sessions | Bars current through prior close; intraday optional for execution realism |
| Live sessions | Bars current through prior close **at minimum**; freshness gate FR-RK-07 enforced |
| Tax/accounting | Official BoC rate for the transaction date |

## 6. Storage & Reproducibility

- **Parquet** for cold market data (compact, typed, fast columnar reads).
- **SQLite (WAL)** for operational state; single-writer discipline (scheduler).
- Every backtest run pins a **snapshot ID** (content hash of the exact input
  data); reports reference it so any historical result can be re-derived
  (FR-BT-08, NFR-13).
- Provenance is stored per bar (source, fetch time, adjustment method) — mixed
  sources never silently blend.

## 7. Licensing & Usage Caveats (read before any non-personal use)

1. All current sources are used under **personal research/paper-trading**
   terms. **Commercialization (P4) re-opens every data license** and is part
   of the P4 decision (OQ-07). yfinance in particular must never appear in a
   revenue-bearing product.
2. TSX-listed real-time data is **paid** under current market structure; this is
   why Phase 1 scope is US-listed securities. Revisit at OQ-05.
3. Redistributing stored data (even derived OHLC) is out of bounds; sharing
   *results/metrics* of personal research is different and generally fine.
4. Any new source must pass: licensing check → reliability check → recorded in
   this doc, before first use ([SDLC](../05-security/04-secure-development-lifecycle.md) trigger).
