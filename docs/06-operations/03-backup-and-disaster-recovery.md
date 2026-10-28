# Backup & Disaster Recovery

| | |
|---|---|
| **Document ID** | DOC-030 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | OPS-10–12 · [ADR-005 hosting](../04-architecture/adr/ADR-005-zero-cost-hosting.md) · [ADR-006 storage](../04-architecture/adr/ADR-006-storage-sqlite-first.md) · [RB-007/008](04-runbooks.md) |

## 1. Recovery Objectives (justified)

| Objective | Target | Why this is acceptable |
|---|---|---|
| **RPO** (max data loss) | ≤ 24 h | Broker is the source of truth for positions (INV-4); the DB is books + history. Worst case: one day of book history — recoverable from broker statements + audit log. |
| **RTO** (service restored) | ≤ 4 h | A missed session costs an opportunity, never safety (fail-closed). Documented, drilled (RB-008). |

## 2. What Gets Protected

| Asset | Backup | Frequency | Retention |
|---|---|---|---|
| SQLite DBs (paper/live) | `sqlite3 .backup` snapshot → restic | Nightly | 30 daily + 12 monthly |
| Audit log (JSONL + chain head) | restic + **chain head replicated off-host daily** (SEC-16) | Nightly | ≥ 7 y (FR-AU-01) |
| Config | Git (private repo) | on change | infinite |
| Code | Git + GitHub (public repo, ADR-008) | on change | infinite |
| Market data | Re-fetchable by design (ADR-004); last 90 days of snapshots cached | weekly | rolling |
| Secrets | **Not backed up by the platform** — re-issueable via broker/Bitwarden; recovery documented in RB-003/008 | — | — |

## 3. Secondary Location

Backups leave the host **encrypted (age) via restic** to a second free
location — home server or a second free-tier storage target (OQ-03 decides;
the restic repository is location-agnostic). Rule: backups never live only on
the host they protect (fire/theft/account-closure scenarios).

## 4. Backup Jobs (systemd timers, heartbeated)

| Job | Schedule (ET) | Heartbeat |
|---|---|---|
| DB snapshot + restic incremental | 02:00 | Healthchecks ping on success; no ping → SEV2 by 06:00 (OPS-06) |
| Audit chain-head replication | 02:15 | same |
| Restic check (integrity) | Sunday 03:00 | same |

## 5. Restore Procedure (summary — full runbook RB-007)

1. Verify the backup set (restic check output) before touching anything.
2. Stop scheduler + notifier units; restore DB files; verify integrity
   (`mq audit-verify` chain check).
3. `mq reconcile` against the broker — **broker wins** (INV-4); book converges
   or trading stays halted.
4. Smoke tests ([Deployment §4](01-deployment-and-environments.md)); resume.

## 6. DR Scenarios

| Scenario | Response | Target |
|---|---|---|
| Host lost/reclaimed (OCI idle reclamation) | Provision new host ([Deployment §2](01-deployment-and-environments.md)), restore, reconcile | ≤ 4 h (RB-008, drilled quarterly) |
| Data corruption (bad migration/bug) | Restore last clean nightly; broker reconciliation repairs books; postmortem + migration fix | ≤ 4 h |
| Provider account issue (cloud/broker) | Fail-closed posture already in effect; rebuild on home server; broker-side is a human escalation | best effort |
| Backup destination loss | Re-seed restic repo from host; DR drill verifies the re-seed path | — |

## 7. Drills & Records (OPS-11)

| Drill | Frequency | Record |
|---|---|---|
| Restore latest DB snapshot to a scratch host | Quarterly | date, duration, issues |
| Full host rebuild (RB-008, timed) | Quarterly | date, duration vs. 4 h, gap list |
| Restic repository integrity check | automatic weekly + quarterly manual | job log |

**A backup that has never been restored is a hypothesis, not a backup** —
untested restores are treated as missing backups.
