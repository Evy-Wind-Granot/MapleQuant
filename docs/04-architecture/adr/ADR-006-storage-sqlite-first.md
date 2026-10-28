# ADR-006: Storage — SQLite-First, Parquet Cold Store

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-10-27 |
| **Deciders** | Founder |

## Context

State: positions/lots, orders, fills, NAV history, FX, corporate actions, and
a small universe table — megabytes, not gigabytes. One writer (the session
process — Architecture R7). Market data: ~12 symbols × 10 y of EOD bars =
trivially small but analytically scanned. $0 budget rules out managed
databases; solo-ops rules out anything needing a DBA (NFR-07).

## Decision

**SQLite (WAL mode)** for operational state with single-writer discipline;
**Parquet** (pyarrow) for cold market data with SQLite holding only an index;
JSONL files for the append-only audit log (hash-chained, FR-AU-03). Backup =
nightly `sqlite3 .backup` + restic (OPS-10).

**Documented migration trigger** (not before needed): DB > ~5 GB, or real
concurrent readers (e.g., a dashboard service querying live) → PostgreSQL on
the same host or Supabase free tier; schema stays portable (no SQLite-only
magic beyond what's recorded in this ADR).

## Consequences

**Positive:** zero infrastructure, trivially backed up and restored, portable
files, easy offline inspection (sqlite3 CLI), fits NFR-24/07 perfectly.

**Negative:** no built-in replication (accepted: broker is the source of truth
for positions — INV-4; worst case is one day of history, RPO 24 h NFR-08);
readers block writers without WAL discipline (enforced by module design R7).

**Mitigations:** WAL mode + one-writer rule + busy_timeout; nightly backups;
quarterly restore tests (OPS-11).

## Alternatives Considered

| Alternative | Rejected because |
|---|---|
| PostgreSQL from day one | Needs a server process and its own patching/backup story for zero present benefit |
| Supabase/Neon free tiers now | External dependency + network egress rules for a database that fits in one file |
| DuckDB for everything | Excellent analytically, but weaker as an OLTP book of record; Parquet+DuckDB read path can be added later if metrics get heavy |
