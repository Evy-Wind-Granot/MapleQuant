# Monitoring & Alerting

| | |
|---|---|
| **Document ID** | DOC-029 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | OPS-06–09 · [Heartbeat/Healthchecks (ADR-005)](../04-architecture/adr/ADR-005-zero-cost-hosting.md) · [Telegram (ADR-007)](../04-architecture/adr/ADR-007-notifications-telegram.md) · [Runbooks](04-runbooks.md) |

## 1. Monitoring Stack ($0)

| Function | Tool | Notes |
|---|---|---|
| Dead-man's heartbeat | **Healthchecks.io** (free) | Multiple ping checks: session-ran, backup-ran, host-alive |
| Metrics + dashboards | Grafana Cloud free tier **or** self-hosted Prometheus+Grafana (decide at P2 — OQ-08) | Metrics exported by session/ notifier via a tiny `/metrics` endpoint or push gateway |
| Push alerts + commands | Telegram bot (ADR-007) | Primary channel |
| Logs | journald + JSONL files (structlog) | On-host; sampled into daily digest |
| Host health | node exporter (if self-hosted) or cloud console alarms | Disk, memory, load |

## 2. Metric Catalog (minimum set)

| Metric | Source | Alert threshold (initial) |
|---|---|---|
| Session outcome (success/degraded/failed) | orchestrator | any failure → SEV2 alert |
| Missed session (heartbeat) | Healthchecks | no ping within grace → SEV2 (OPS-06) |
| Order latency (submit→ack) | adapter | p95 > 10 s → SEV3 note |
| Realized slippage vs. model (bps) | fills vs. open | > 10 bps beyond model → SEV2 (G2 gate: ≤ 10 bps average) |
| Reconciliation result | reconciler | any break → SEV2 (halt is automatic) |
| Breaker state | risk manager | any trip → SEV1/SEV2 per type |
| Kill-switch state | notifier/orchestrator | state change → immediate alert (any direction) |
| NAV + P&L vs. backtest expectation | accounting | drift beyond band → weekly digest highlight |
| Data freshness per symbol | ingestor | stale symbol on a decision day → SEV2 (already fail-closed) |
| Backup completed | backup job | no ping by 06:00 ET → SEV2 |
| Host: disk / memory / load | host | disk > 80% → SEV3; > 90% → SEV2 |
| Security events (SSH fails, Tailscale joins) | host/journald | any → SEV2 review (SEC-24) |
| Monthly cost | invoice check | **any non-zero charge → SEV2 (OPS-13)** |

## 3. Alert Routing & Rules

| Condition | Severity | Channel | Runbook |
|---|---|---|---|
| Credential suspicion, unwanted orders, gate bypass | SEV1 | Telegram (alert tone) + repeat every 5 min until ack | RB-003 |
| Reconciliation break / broker down / monitoring dead / missed session / non-zero invoice | SEV2 | Telegram, immediate | RB-001/002/004/005/006 |
| Stale data, quality rejection | SEV2 (auto fail-closed) | Telegram | RB-005 |
| Cosmetic/report issues | SEV3 | daily digest | — |

Rules: every alert carries **state + cause + next step (runbook link)** —
NFR-20. Alerts **deduplicate per condition per session** (no storm); a still-
unacked SEV1 re-fires every 5 minutes. Notification failure never silences
trading (FR-NT-04) but heartbeat reflects it.

## 4. Daily Digest (FR-NT-01) Contents

NAV (CAD), positions vs. targets, fills, realized slippage, exception list,
breaker/kill states, data freshness, backup status, cost status, and a link to
the full session report artifact. The digest is the operator's 30-second
answer to "is everything fine?" — missing digest is itself an alert
condition.

## 5. Review Cadence

| Review | Frequency | Content |
|---|---|---|
| Daily | ≤ 30 s | digest scan |
| Weekly ops review (OPS-09) | ≤ 30 min | alerts, metrics vs. [targets](../08-planning/03-success-metrics.md), reconciliation, cost, upcoming calendar |
| Quarterly | — | thresholds tuning, runbook review (OPS-17), DR drill (OPS-11) |
