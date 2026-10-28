# ADR-007: Notifications & Commands via Telegram

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-10-27 |
| **Deciders** | Founder |

## Context

The operator needs: push alerts within minutes (NFR-05), a daily digest
(FR-NT-01), and a kill switch reachable from anywhere including off-host
(FR-RK-04, SEC-23) — on $0. Email push on phones is unreliable as an
immediate channel; SMS gateways are paid.

## Decision

Use the **Telegram Bot API** for alerts, digests, and authenticated commands
(`/status`, `/kill`). Security constraints: allow-listed chat ID (SEC-14),
bot token stored as a secret (SEC-01), commands are read/act-on-kill only —
no other state-changing commands over Telegram. Notification failures never
block trading (FR-NT-04); an independent local kill path always exists
(SEC-23).

## Consequences

**Positive:** free, instant push, two-way (commands), trivial bot API,
worldwide reach, independent failure mode from the host (works when the host
is degraded — `/kill` sets the flag the next session sees).

**Negative:** Telegram is a third party (its outage = blind alerts — RSK-16);
requires internet egress to telegram.org (added to allowlist SEC-08); bot
token is a credential to protect.

**Mitigations:** heartbeat (Healthchecks.io) covers missed-run detection
independently; audit log remains authoritative on-host; redundancy option
tracked as OQ-10.

## Alternatives Considered

| Alternative | Rejected because |
|---|---|
| Email-only (SMTP) | Push latency unreliable; no command channel; SMTP creds/relay needed |
| Discord | Same capability; slightly heavier personal-account dependency |
| Twilio/SMS | Paid — breaks NFR-24 |
| Pushover | Excellent, but paid after trial |
