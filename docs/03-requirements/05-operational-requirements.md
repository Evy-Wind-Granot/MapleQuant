# Operational Requirements

| | |
|---|---|
| **Document ID** | DOC-013 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [Deployment](../06-operations/01-deployment-and-environments.md) · [Monitoring](../06-operations/02-monitoring-and-alerting.md) · [Backup & DR](../06-operations/03-backup-and-disaster-recovery.md) · [Runbooks](../06-operations/04-runbooks.md) · [Cost Management](../06-operations/05-cost-management.md) |

## 1. Environments & Deployment

| ID | Requirement | Verification |
|---|---|---|
| OPS-01 | Three profiles on one host — dev, paper, live — each with its own config, credentials, and database; the same binary runs all three. | Host config audit |
| OPS-02 | All services are systemd units: `Restart=on-failure`, resource limits, journald structured logging. | Unit review |
| OPS-03 | Scheduled sessions run via systemd timers (calendar + trading-calendar aware); jitter windows allowed off-market-hours only. | Host audit |
| OPS-04 | Deploys follow the documented pipeline: CI green → tagged release → deploy over Tailscale → smoke test; rollback ≤ 30 min (NFR-17). | Deploy log |
| OPS-05 | Live profile changes require: release on paper ≥ 1 full session + deliberate arm step (FR-EX-08). | Process check |

## 2. Monitoring & Alerting

| ID | Requirement | Verification |
|---|---|---|
| OPS-06 | Heartbeat: every session pings Healthchecks.io; a missed ping alerts within 15 min (NFR-05). | MON |
| OPS-07 | Metrics monitored (see [Monitoring doc](../06-operations/02-monitoring-and-alerting.md) for the full list): session success, order latency, slippage, reconciliation breaks, breaker state, host health. | MON |
| OPS-08 | SEV1 response ≤ 15 min during market hours; SEV2 ≤ same day. Alert messages carry state, cause, and runbook link (NFR-20). | Ops log |
| OPS-09 | Weekly ops review (≤ 30 min): alerts, metrics vs. targets, reconciliation reports, costs. | Ops log |

## 3. Backup & Disaster Recovery

| ID | Requirement | Verification |
|---|---|---|
| OPS-10 | Nightly automated SQLite `.backup` + weekly encrypted (age) restic snapshot to a secondary location; retention: 30 daily / 12 monthly. | Cron/backup log |
| OPS-11 | Restore is **tested quarterly**; DR rebuild (fresh host → trading) within 4 h (NFR-16) using RB-008. | DR drill log |
| OPS-12 | Audit log and data snapshots are included in backups; audit chain head replicated off-host (SEC-16). | Backup config audit |

## 4. Cost Governance

| ID | Requirement | Verification |
|---|---|---|
| OPS-13 | **Hard $0/month fixed-cost budget.** Any non-zero invoice or charge is treated as a SEV2 incident: investigate, remediate, document. Monthly cost review is part of the ops review. | Cost review log |
| OPS-14 | Free-tier limits are tracked (see [Cost Management](../06-operations/05-cost-management.md) inventory); approaching a limit is a planning item, never a surprise bill. | Inventory review |

## 5. Change & Incident Management

| ID | Requirement | Verification |
|---|---|---|
| OPS-15 | All changes (code or config) via PR + CI; `main` protected; config changes are treated as code changes. | Repo settings |
| OPS-16 | SEV1/SEV2 incidents get a blameless postmortem within 48 h; corrective actions become runbook/monitoring updates. | Postmortem log |
| OPS-17 | Runbooks cover all SEV1/SEV2 scenarios; reviewed quarterly (RB-001–RB-008 minimum set). | Review log |
| OPS-18 | Kill-switch and credential-revocation **drills** are passed before live trading (G2 gates). | Drill records |

## 6. Hygiene

| ID | Requirement |
|---|---|
| OPS-19 | NTP time sync mandatory (scheduling + audit timestamp integrity). |
| OPS-20 | Host patching: security updates ≤ 14 days; dependency updates monthly (SEC-20). |
| OPS-21 | Quarterly capacity review: disk, memory, DB size vs. NFR-10/11 budgets. |
| OPS-22 | Documentation duty: any architectural decision gets an ADR before merge; this requirements set is reviewed at each stage gate. |

## 7. Operator Load Budget (the constraint behind everything above)

| Period | Budget |
|---|---|
| Build phases (P0–P2) | Project time is the primary activity; ops load not yet bounded |
| Steady-state (P3+) | **≤ 30 min/week** routine + drills + reviews as scheduled above |
| Incidents | Unbounded but rare by design (fail-closed + alerts); that is the trade-off the architecture buys |
