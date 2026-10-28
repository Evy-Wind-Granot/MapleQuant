# RB-NNN — <Scenario Title>

| | |
|---|---|
| **Severity class** | SEV1 \| SEV2 \| SEV3 |
| **Owner** | Founder |
| **Last drilled** | YYYY-MM-DD |
| **Related** | monitoring alert · requirement IDs · docs |

## Symptoms

What the operator sees/hears: alerts, digest anomalies, dashboard states.
Be specific — this section is the *detection aid*.

## Immediate actions

Numbered, ordered, < 15 minutes total for SEV1. Panic-safe: every step must be
doable from a phone or a single SSH session. State what **not** to do, if the
tempting first move is wrong.

## Diagnosis

Branches: if X then Y. Include the exact commands (`mq …`, `systemctl …`,
`journalctl …`, `restic …`) with expected outputs.

## Resolution

The path back to normal trading, including any deliberate re-arm steps and
their cooldowns (FR-RK-05).

## Prevention

What to add afterward: monitoring, tests, runbook fixes, requirement changes
(postmortem actions land here).

---

> Writing rule: **checklists, not essays**. If a step needs > 3 sentences of
> explanation, the explanation belongs in a design doc — link it, don't inline
> it.
