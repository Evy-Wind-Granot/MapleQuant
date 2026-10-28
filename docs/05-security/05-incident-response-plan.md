# Incident Response Plan

| | |
|---|---|
| **Document ID** | DOC-027 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [Runbooks](../06-operations/04-runbooks.md) · [Secrets §5](03-secrets-and-credential-management.md) · [Threat Model](02-threat-model.md) · [Monitoring](../06-operations/02-monitoring-and-alerting.md) |

## 1. Severities & Response Targets

| Level | Definition | Examples | Response target (OPS-08) |
|---|---|---|---|
| **SEV1** | Live capital at risk, credential compromise, unauthorized trading, platform safety controls failing | Key leak suspicion; unwanted orders executing; risk gate bypassed; unexplainable positions | **≤ 15 min** during market hours; immediately otherwise |
| **SEV2** | Trading impaired or control/detection degraded, but risk contained | Reconciliation break; broker adapter down; monitoring or heartbeat dead; **any non-zero invoice (OPS-13)**; kill switch unreachable via one path | Same trading day |
| **SEV3** | Degraded with no money or safety impact | Report formatting; digest delayed; cosmetic UI/log issues | Within one week |

## 2. Roles

Solo operation — one role, two hats: **Responder** (protect capital first) and
**Investigator** (evidence first) — in that order of precedence. All external
escalation paths (broker support, Telegram platform, cloud console) are
pre-verified and listed in §6.

## 3. Response Flow

```
Detect (alert/heartbeat/digest anomaly)
  → PROTECT: kill switch (FR-RK-04) — stops new orders in seconds
  → CONTAIN: revoke credentials at the provider (RB-003), cancel resting orders broker-side
  → ERADICATE: rotate remaining secrets; patch root cause; rebuild host if compromise suspected (RB-008)
  → RECOVER: verify clean reconciliation (FR-RC), paper-run one session, deliberate re-arm (FR-RK-05)
  → LEARN: blameless postmortem ≤ 48 h (OPS-16); runbook/monitoring fixes become issues
```

**Priority rule:** when in doubt, over-escalate. A SEV1 false alarm costs a
re-arm; a missed SEV1 costs capital. Nobody has ever been fired from this
company for pulling the kill switch.

## 4. SEV1 Playbook (order of operations)

| # | Action | Time budget | Tool |
|---|---|---|---|
| 1 | Activate kill switch | 0–2 min | Telegram `/kill` — or local flag / CLI if Telegram suspect |
| 2 | Verify entries blocked (status query) | 2–5 min | `mq status` over Tailscale |
| 3 | **Revoke broker API key at the broker** | 5–15 min | Broker dashboard (SEC-21 — the platform is not needed for this) |
| 4 | Cancel all resting orders | ≤ 15 min | Broker dashboard |
| 5 | Flatten positions if warranted (judgment call) | as needed | Broker dashboard |
| 6 | Snapshot evidence: audit log tail, journald, state DB copy, config | after containment | `mq incident-snapshot` (or manual copy) |
| 7 | Rotate *all* other credentials (bot token, SSH, CI paper keys) | ≤ 24 h | [Secrets §4](03-secrets-and-credential-management.md) |
| 8 | Root cause; postmortem; fixes | ≤ 48 h | §5 template |

## 5. Postmortem Template (fill within 48 h; blameless — OPS-16)

```markdown
## Incident <YYYY-MM-DD>-<slug>
- Severity / duration / dates (UTC + Toronto)
- Impact: positions, P&L, records, time lost
- Timeline: detect → contain → recover (timestamps)
- Root cause (technical; "human error" is a starting point, not a conclusion)
- What worked / what failed
- Contributing factors
- Action items: runbook fix | monitoring addition | requirement change (→ RTM)
- Threat-model update needed? (SDLC §5 triggers)
- Incident log entry added below (this doc, §7)
```

## 6. External Escalation Directory (verify quarterly)

| Party | Purpose | Channel |
|---|---|---|
| Wealthsimple support | Key revocation, account security, order inquiries | App / phone (verify) |
| Alpaca support | Paper key issues | Dashboard / email |
| Tailscale admin | Device revoke, tailnet audit | Admin console |
| Oracle Cloud console | Instance reclaim/compromise response | Cloud console |
| Telegram @BotFather | Token re-issue | Telegram |

## 7. Incident Log (append-only)

| Date | Severity | Summary | Postmortem link |
|---|---|---|---|
| — | — | *(no incidents recorded)* | — |

## 8. Drills

| Drill | Requirement |
|---|---|
| Kill-switch drill (both paths, timed) | Pass before G2; annually after (SEC-23) |
| Credential revocation drill (timed ≤ 15 min, paper key) | Pass before G3; annually (SEC-21) |
| Host rebuild drill (RB-008, timed ≤ 4 h) | Quarterly (OPS-11) |
| Monitoring-blind drill (pause alerts, verify heartbeat fires) | Quarterly |
