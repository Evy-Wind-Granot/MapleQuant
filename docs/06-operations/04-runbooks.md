# Operational Runbooks

| | |
|---|---|
| **Document ID** | DOC-031 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [Incident Response](../05-security/05-incident-response-plan.md) · OPS-17 (quarterly review) · Template: [`templates/runbook-template.md`](../../templates/runbook-template.md) |

> **Format:** each runbook is deliberately short. In an incident you need a
> checklist, not an essay. Details live in the linked design docs.

## RB-001 — Bot unresponsive / missed heartbeat

| | |
|---|---|
| **Symptoms** | No Healthchecks ping; no digest; `/status` silent. |
| **Immediate** | 1. Don't panic-trade manually — the account is safe: fail-closed means worst case is *no trades*. 2. `systemctl status mq-*` over Tailscale. 3. Check journald for the last session's exit. |
| **Diagnose** | Timer misfire → `systemctl list-timers`; process crash → restart policy kicked in?; host down → OCI console / power state; Tailscale split → check tailnet. |
| **Resolve** | Fix cause; `mq run --now` to catch up a missed session **only if** market logic allows (no chasing stale signals — when in doubt, wait for the next session). |
| **Prevent** | Heartbeat grace tuned so this alerts before the next market open (OPS-06). |

## RB-002 — Kill switch: activation & deactivation

| | |
|---|---|
| **Activation** | `/kill` (Telegram) — or `mq kill` — or `touch /opt/maplequant/KILL`. All logged (FR-RK-04). Entries blocked instantly; resting orders cancelled per config; flatten only if configured on. |
| **Verify** | `mq status` shows `kill=ACTIVE`; audit entry exists. |
| **Deactivation** | Deliberate, never automatic (FR-RK-05): 1. Resolve the reason that triggered it. 2. `mq disarm --reason "<why>"`. 3. If a breaker tripped (not manual kill), respect the cooldown (FR-RK-03). 4. Confirm next session proposes normally. |
| **Drill** | Timed kill drill before G2, annually after (SEC-23). |

## RB-003 — Suspected credential compromise (SEV1)

| | |
|---|---|
| **Immediate** | Follow the [IR SEV1 playbook](../05-security/05-incident-response-plan.md) §4 in order: **kill (≤ 2 min) → revoke at broker (≤ 15 min) → cancel resting orders → snapshot evidence → rotate everything (≤ 24 h)**. |
| **Rules** | Revoke *before* investigating. The platform is not needed for revocation (broker dashboard is). Preserve audit log tail before any remediation that could overwrite process state. |
| **After** | Postmortem ≤ 48 h; rotation log updated ([Secrets §6](../05-security/03-secrets-and-credential-management.md)); threat-model review (SDLC §5). |

## RB-004 — Broker API outage / degradation

| | |
|---|---|
| **Symptoms** | Adapter `Unavailable`, repeated 5xx, auth errors; session marked degraded. |
| **Immediate** | Nothing to stop — the system already fail-closed (FR-EX-06/FR-RK-09). Confirm no resting orders are exposed; check broker status page. |
| **If auth errors** | Treat as **potential credential issue → escalate to RB-003**. Never assume "broker problem" for auth failures. |
| **Resolve** | Wait out the outage; next session reconciles automatically. If the outage spans a decision day: skip — the calendar will offer the next window. |

## RB-005 — Data feed anomaly / stale data

| | |
|---|---|
| **Symptoms** | Quality-check failures, staleness flags, spike alerts (FR-DA-02/03). |
| **Immediate** | Session has already fail-closed for affected decisions. Verify with the secondary source (yfinance spot-check) — is it source-wide or symbol-specific? |
| **Diagnose** | Source outage vs. genuine market event vs. adjustment glitch (provenance metadata shows source + fetched_at). |
| **Resolve** | Re-run ingest when the source recovers; if data was *wrong but accepted*, run the spike-flag review and write a data-quality incident note. |
| **Prevent** | Multi-source provenance (ADR-004); spike flags gate auto-trading on flagged days. |

## RB-006 — Reconciliation break

| | |
|---|---|
| **Symptoms** | Pre-session/EOD diff alert; session held (FR-RC-01/02). |
| **Immediate** | `mq reconcile --explain` → diff table (positions, cash, lots). |
| **Diagnose** | Missed fill update → book is behind; manual/other-source trade on the account (did *you* trade on the phone app?); corporate action unprocessed; real bug. |
| **Resolve** | Broker wins (INV-4): `mq reconcile --adopt-broker` records the diff as an accounting event (audited). If a defect caused it: book a fix; **do not** hand-edit the DB. |
| **Re-arm** | Trading stays halted until a clean reconciliation passes (FR-RC-02). |

## RB-007 — State DB recovery from backup

| | |
|---|---|
| **When** | Corruption, bad migration, accidental damage. |
| **Steps** | 1. Stop units. 2. `restic check` the repo. 3. Restore snapshot per [Backup §5](03-backup-and-disaster-recovery.md). 4. `mq audit-verify` chain check. 5. `mq reconcile` — broker wins. 6. Smoke tests. 7. Resume. |
| **Rule** | Never restore a DB without immediately reconciling against the broker; never trade on an unreconciled book. |

## RB-008 — Host rebuild (DR, RTO 4 h — NFR-16)

| | |
|---|---|
| **Steps** | 1. Provision host per [Deployment §2](01-deployment-and-environments.md) checklist (scriptable — keep the script in the engine repo as `infra/provision.sh`). 2. Join Tailscale; firewall; hardening. 3. Install latest release wheel + SBOM. 4. Re-issue credentials (fresh — never reuse); load into systemd creds. 5. Restore DBs per RB-007. 6. Smoke tests. 7. Reconcile. 8. First session supervised (`mq run --now --supervised`). |
| **Drill** | Quarterly, timed (OPS-11); duration logged. |
| **Note** | Old host's credentials: **revoke all** after rebuild if the host was lost/untrusted — assume compromise (RB-003 posture). |

## Coverage check (OPS-17)

SEV1 scenarios → RB-002/003 (+ IR plan). SEV2 scenarios → RB-001/004/005/006
(+ cost incident per OPS-13). Recovery paths → RB-007/008. Any incident that
lacked a runbook creates one within its postmortem.
