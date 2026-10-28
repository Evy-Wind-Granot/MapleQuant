# Technology Stack

| | |
|---|---|
| **Document ID** | DOC-017 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [ADRs](adr/README.md) · [Cost Management](../06-operations/05-cost-management.md) · [Secure SDLC](../05-security/04-secure-development-lifecycle.md) |

> **License discipline:** every dependency's license and terms are verified at
> adoption and recorded here (SDLC checklist item). Rows marked ⚠ carry terms
> that must be confirmed before first use or before any non-personal use.

## 1. Stack by Layer

| Layer | Choice | License / cost | Rationale | Alternatives considered |
|---|---|---|---|---|
| Language | **Python 3.12+** | PSF / $0 | Quant ecosystem, solo-maintainable ([ADR-001](adr/ADR-001-python-primary-language.md)) | Rust (perf, cost of dev), Node (weaker data stack) |
| Env & deps | **uv** (lockfile w/ hashes) | MIT / $0 | Fast, single tool, lockfile hashes support supply-chain rule SEC-17 | pip-tools, poetry |
| Data processing | **pandas + numpy** | BSD / $0 | Default standard; adequate for EOD-scale data | polars (speed, smaller ecosystem), DuckDB for analytical reads |
| Validation | **pydantic v2** | MIT / $0 | Strict schemas for all external inputs (SEC-11) | dataclasses + manual checks |
| HTTP | **httpx** | BSD / $0 | Timeouts by default, sync+async | requests |
| Retry logic | **tenacity** | Apache-2.0 / $0 | Declarative backoff for transient errors (FR-EX-06) | hand-rolled |
| CLI | **typer** | BSD / $0 | `mq` admin CLI (FR-CF-04) | argparse (fine, more boilerplate) |
| Logging | **structlog** (JSON) | Apache-2.0/MIT / $0 | Structured events → journald + files | stdlib logging |
| Storage (hot) | **SQLite** (WAL) | Public domain / $0 | One-writer, trivial backup ([ADR-006](adr/ADR-006-storage-sqlite-first.md)) | Postgres (overkill now, documented migration path) |
| Storage (cold) | **Parquet** via pyarrow | Apache-2.0 / $0 | Typed, compact market data | CSV (larger, untyped) |
| Backtest/execution engine | **Custom thin event-driven core** | — | One code path live/backtest ([ADR-003](adr/ADR-003-single-codepath-event-driven-core.md)); existing libs are backtest-only or heavyweight | backtrader, vectorbt, LEAN (all break single-codepath or add mass) |
| Market data | **Stooq** (primary EOD) · **yfinance** ⚠ · **alpaca-py** (intraday/paper) · **BoC Valet** (FX, official) | Free tiers / $0 — ⚠ terms | See [Data Requirements](../03-requirements/06-data-requirements.md) | paid feeds (blocked by $0 rule) |
| Calendar | **exchange_calendars** ⚠ | Verify at adoption | NYSE/TSX sessions, holidays (FR-DA-06) | hand-maintained calendar (fragile) |
| Secrets | **age** + `systemd-creds` (`LoadCredential=`) | MIT / $0 | Encrypted at rest, no server to run | Vault (heavy), plaintext env (violates SEC-01) |
| Human password vault | **Bitwarden** free tier | $0 | SEC-06 support | browser-saved passwords (no) |
| Host | **Oracle Cloud Always Free ARM** primary; home server fallback | $0 | 4 OCPU/24 GB always-free tier ([ADR-005](adr/ADR-005-zero-cost-hosting.md)) | paid VPS (violates $0), GH Actions as host (unsuitable) |
| OS runtime | Ubuntu LTS, systemd | Free | Boring, documented, unattended upgrades | — |
| Admin access | **Tailscale** free tier | $0 | No inbound ports (SEC-07) | public SSH (rejected) |
| Scheduling | **systemd timers** | $0 | Native, journald-integrated, trading-calendar aware (OPS-03) | cron, APScheduler |
| CI/CD | **GitHub Actions** | Free for public repos; private free minutes | [ADR-008](adr/ADR-008-repository-strategy.md) | self-hosted CI (ops cost) |
| Heartbeat | **Healthchecks.io** free tier | $0 | Dead-man's switch (OPS-06) | UptimeRobot |
| Metrics/dashboards | **Grafana Cloud free tier** vs self-hosted Prometheus+Grafana | $0 | Decide at P2 (OQ-08) | none (logs only) |
| Alerts & commands | **Telegram Bot API** | $0 | Push + commands; allow-listed chat ([ADR-007](adr/ADR-007-notifications-telegram.md)) | email-only (no commands, slow) |
| Backups | **restic** + age, to secondary free location | BSD-2 / $0 | Encrypted, deduped (OPS-10) | rclone raw copy |

## 2. Engineering Toolchain

| Purpose | Tool | CI gate |
|---|---|---|
| Lint + format | **ruff** | blocking |
| Types | **mypy** strict on `core/` | blocking |
| Tests | **pytest**, **hypothesis** (property tests for risk math), coverage ≥ 85% core | blocking |
| Security scan (code) | **bandit** | blocking |
| Vulnerability scan (deps) | **pip-audit** | blocking |
| Secret scan | **gitleaks** | blocking |
| Hooks | **pre-commit** | local |
| Release artifacts | GitHub Releases (wheel + SBOM + config schema) | tagged |

## 3. Versioning & Update Policy

- Python: one minor at a time; upgrade within 6 months of release.
- Dependencies: monthly update cadence with lockfile diff review (SEC-20);
  security fixes within 7 days.
- No unpinned dependencies anywhere; lockfile hashes required (SEC-17).

## 4. Rejection Log (what we deliberately do NOT use)

| Rejected | Reason |
|---|---|
| Docker/Kubernetes on the host | Adds state complexity for zero benefit at this scale; systemd is the deployment unit |
| Message queues (Redis, RabbitMQ) | Single-writer orchestrator needs none ([Architecture §10](01-system-architecture.md)) |
| NoSQL databases | Data is relational and small |
| Web frameworks | No web UI in MVP; Telegram + CLI is the interface |
| Broker unofficial APIs / wrappers | RL-5 — ToS violation, brittle, account risk |
| Paid anything | NFR-24 until P4 changes the rules |
