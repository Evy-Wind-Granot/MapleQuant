# Architecture Decision Records (ADRs)

| | |
|---|---|
| **Document ID** | DOC-022 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |

ADRs record **why** a significant decision was made, so future-you doesn't
re-litigate it blind. They are immutable once Accepted: changes get a new ADR
that supersedes the old one.

## Index

| ADR | Title | Status |
|---|---|---|
| [ADR-001](ADR-001-python-primary-language.md) | Python as the primary language | Accepted |
| [ADR-002](ADR-002-broker-agnostic-adapter-architecture.md) | Broker-agnostic adapter architecture | Accepted |
| [ADR-003](ADR-003-single-codepath-event-driven-core.md) | Single code path: custom event-driven core for backtest + live | Accepted |
| [ADR-004](ADR-004-market-data-sources.md) | Market data: free multi-source with licensing caveats | Accepted |
| [ADR-005](ADR-005-zero-cost-hosting.md) | Hosting: Oracle Cloud Always Free with home-server fallback | Accepted |
| [ADR-006](ADR-006-storage-sqlite-first.md) | Storage: SQLite-first, Parquet cold store, documented Postgres path | Accepted |
| [ADR-007](ADR-007-notifications-telegram.md) | Notifications & commands via Telegram | Accepted |
| [ADR-008](ADR-008-repository-strategy.md) | Repository strategy: public engine code, private business docs | Accepted |

## Process

1. Write the ADR (template: [`templates/adr-template.md`](../../templates/adr-template.md)) when a decision is first needed — not after.
2. Status: `Proposed` → `Accepted` (or `Rejected`, `Superseded`).
3. Numbered sequentially; never edit an Accepted ADR's decision — supersede.
4. ADRs that change security posture trigger a [threat-model review](../../05-security/02-threat-model.md).
5. Cross-reference from the affected requirement/design docs so links stay navigable.
