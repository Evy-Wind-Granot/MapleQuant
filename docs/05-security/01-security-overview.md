# Security Overview

| | |
|---|---|
| **Document ID** | DOC-023 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [Security Requirements](../03-requirements/04-security-requirements.md) · [Threat Model](02-threat-model.md) · [Secrets Management](03-secrets-and-credential-management.md) · [Incident Response](05-incident-response-plan.md) |

## 1. Scope & Security Goals

This program protects three things, in priority order:

1. **The brokerage account** — its trading capability and, above all, its money.
2. **The trading state and records** — books, audit log (financial + tax truth).
3. **The strategy/IP** — modest value today; optionality for P4.

**Security goals** (from the requirements doc): an attacker with any platform
secret can trade but **cannot move money**, cannot stay undetected, and every
credential dies within 15 minutes of discovery (SEC-21).

## 2. Guiding Principles

| Principle | Application |
|---|---|
| Least privilege | Trade-only API scopes (SEC-02); non-login service user (SEC-09); minimal packages |
| Fail closed | Every uncertainty → no orders (INV-1); unknown order state → worst case (FR-EX-04) |
| Minimize secret surface | Secrets exist in exactly one place (host credential store), for the minimum set of services |
| Assume breach | Detection + revocation drills are requirements (SEC-21/23), not paperwork |
| Defense in depth | Config-level limits **and** code-enforced maxima (SEC-13); kill switch with independent paths |
| $0-compatible security | Free/open tooling only — encryption, Tailscale, gitleaks, pip-audit — chosen precisely because cost can't be the excuse |

## 3. Control Summary (maps requirements → mechanisms)

| Control area | SEC IDs | Mechanisms | Verified by |
|---|---|---|---|
| Credential lifecycle | 01–06 | age/systemd-creds encryption, 0600, trade-only scopes, rotation calendar, human 2FA | Credential register, host audit |
| Host & network | 07–10 | Tailscale-only, egress allowlist, ufw default-deny, unattended upgrades | Quarterly host audit |
| Application | 11–16 | pydantic boundaries, no dynamic eval, code-enforced risk maxima, authenticated Telegram | CI (bandit, tests) |
| Supply chain | 17–20 | uv lockfile w/ hashes, pip-audit, minimal deps, justification per dep | CI + PR review |
| Detection & response | 21–25 | alerts on anomalies, revocation/kill drills, monitored security events | Drills + monitoring |

## 4. Asset Inventory

| Asset | Location | Sensitivity | Worst-case loss |
|---|---|---|---|
| Broker API credentials (live) | Host credential store only | **Critical** | Unwanted trades (no withdrawal — SEC-02) |
| Broker account (human login) | Broker platform | **Critical** | Full account compromise — mitigated by 2FA + no API withdrawal scope |
| Telegram bot token | Host credential store | High | Fake kill/status messages; no order path |
| Tailscale / SSH keys | Host + operator devices | High | Host access (jumping trust boundary) |
| State DB + audit log | Host + backups | High | Tax/records damage; tamper-evident (SEC-16) |
| Strategy code/params | Public + private repos | Low–Medium | Copying (accepted — see [ADR-008](../04-architecture/adr/ADR-008-repository-strategy.md)) |
| Market data | Host (Parquet) | Low | Re-fetchable |

## 5. Roles & Authority

| Role | Authority |
|---|---|
| Operator (human) | Everything incl. arm/disarm, credential revocation, withdrawals (outside the platform — the bot never has this authority, RL-6) |
| Platform (service user) | Read market data, read/write its DB, place/cancel orders via trade-only scopes, send Telegram messages. **Cannot** move money, cannot reach other hosts, cannot accept inbound connections |
| Telegram bot listener | Set kill flag; send messages; read allow-listed chats |

## 6. What this program is *not*

Not a compliance certification, not a penetration-tested product, not legal
advice. It is a deliberate, proportionate security posture for a personal
system — documented so that if MapleQuant ever commercializes (P4), the
posture, evidence trail, and gaps are already mapped for professional review.
