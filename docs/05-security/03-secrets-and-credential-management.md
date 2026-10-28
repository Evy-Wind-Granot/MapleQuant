# Secrets & Credential Management

| | |
|---|---|
| **Document ID** | DOC-025 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [SEC-01–06](../03-requirements/04-security-requirements.md) · [Threat Model T-01](02-threat-model.md) · [RB-003](../06-operations/04-runbooks.md) |

## 1. Credential Inventory & Register

A **credential register** (this file, §1, maintained by hand) records every
issued credential: what, where, scope, issue date, rotation due, revocation
path. The register stores *metadata only* — never a secret.

| Credential | Stored where | Scope | Rotation | Revocation |
|---|---|---|---|---|
| Wealthsimple Trade API key (live) *(pending access)* | Host credential store only | **Trade-only** (verify at issuance — SEC-02) | ≤ 90 d | Broker dashboard, ≤ 15 min (RB-003) |
| Alpaca paper key | Host + CI (paper only, SEC-05) | Paper trading | ≤ 90 d | Alpaca dashboard |
| Telegram bot token | Host credential store | Send messages; listen | ≤ 90 d | BotFather re-issue |
| Tailscale auth key | One-time use, not stored | Device join | ephemeral | Tailscale admin (revoke device) |
| Host SSH keys | Operator device + host `authorized_keys` | Shell (operator user only) | ≤ 1 y | Remove key + rotate |
| Bitwarden master password + recovery | Operator memory + sealed recovery kit | Human password vault | per policy | — |
| Email / GitHub / broker account credentials | Bitwarden only | Human accounts, TOTP 2FA (SEC-06) | ≤ 180 d | Platform-specific |

## 2. Storage Scheme (host)

- **Mechanism:** `age`-encrypted secrets file(s), decrypted at boot into
  systemd-credentialed services via `LoadCredential=` (or `systemd-creds`
  directly). Plaintext secrets exist **only** in the process environment of
  the consuming service — never on disk unencrypted, never in the repo,
  never in backups unencrypted (SEC-01, SEC-21 of NFR set).
- **Permissions:** encrypted blob `0600 root:maplequant`; service runs as the
  `maplequant` non-login user (SEC-09).
- **CI:** paper-only secrets via GitHub Actions secrets; **live credentials
  never enter CI** (SEC-05).
- **Format rule:** every secret is an environment variable consumed at
  startup and validated for presence/shape before any trading action —
  missing credentials fail closed with a clear message.

## 3. Never List (hard rules)

1. Never in git — pre-commit + CI gitleaks (SEC-03) make violation loud.
2. Never in logs, error messages, reports, or config files.
3. Never pasted into chat (incl. AI tools), tickets, screenshots, or emails.
4. Never in the docs repo (this register holds metadata only).
5. Never a withdrawal/money-movement scope on any bot credential (SEC-02).
6. Never reused across environments (paper key ≠ live key).

## 4. Rotation Schedule

| Credential class | Routine | Event-driven |
|---|---|---|
| Broker API keys | 90 days | Immediately on any suspicion, personnel/device change, or after any SEV1 (RB-003) |
| Bot tokens | 90 days | Host re-provision |
| SSH keys | 12 months | Device loss/change |
| Human account passwords | 180 days or on platform breach news | — |

Rotation events are logged (date, credential, reason) in this file's §6.

## 5. Emergency Revocation (summary — full procedure RB-003)

Order of operations for a suspected credential compromise:
**kill switch first** (stop new orders) → **revoke at broker** (≤ 15 min) →
**cancel resting orders via broker UI** → **rotate everything else** →
investigate → postmortem. The platform is *not* required for any of these
steps — all revocation paths are broker/human-side by design.

## 6. Rotation & Incident Log (append-only)

| Date | Credential | Action | Reason |
|---|---|---|---|
| — | — | — | *(no entries yet)* |

## 7. Human-Side Hygiene (SEC-06/25)

- Password manager (Bitwarden free) is the only password store; unique
  passwords everywhere; TOTP 2FA minimum on email, GitHub, broker, Telegram,
  cloud consoles; hardware key where supported.
- Email is the account-recovery root of trust → strongest protection, alerts
  enabled for every login.
- Operator workstation: full-disk encryption, auto-lock, patched ≤ 14 days,
  no shared use, no trading-account logins on untrusted devices.
