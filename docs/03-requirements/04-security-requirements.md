# Security Requirements

| | |
|---|---|
| **Document ID** | DOC-012 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [Threat Model](../05-security/02-threat-model.md) · [Secrets Management](../05-security/03-secrets-and-credential-management.md) · [Incident Response](../05-security/05-incident-response-plan.md) · [Regulatory RL-6](../01-market-compliance/03-regulatory-compliance.md) |

## 1. Security Goals

1. An attacker who obtains **any** platform secret can trade your positions but
   **cannot withdraw, transfer, or move money** and is detected quickly.
2. A compromised host is required to reach live credentials; no secret is ever
   reachable from the repository, CI, logs, or chat.
3. Every security-relevant action is auditable; every credential is revocable
   within 15 minutes.

## 2. Credential & Secret Handling

| ID | Requirement | Verification |
|---|---|---|
| SEC-01 | Broker API credentials and bot tokens are **encrypted at rest** (age-encrypted file or systemd-creds `LoadCredential=`), file mode 0600, owned by the service user, never plaintext on disk outside the kernel-provided credential. | Host audit + UT |
| SEC-02 | Bot credentials must **never include withdrawal, transfer, or money-movement scopes**. Scopes are verified and recorded at issuance; if a broker cannot issue trade-only keys, that broker is not integrated. | DOC (credential register) + adapter capability check |
| SEC-03 | Secrets never appear in: the repo, config files, logs, error messages, reports, chat, or CI. `gitleaks` runs in CI on every PR. | CI (blocking) |
| SEC-04 | Credential rotation: routine ≤ 90 days; **immediate** on any suspicion ([runbook RB-003](../06-operations/04-runbooks.md)). | Rotation log |
| SEC-05 | CI holds **paper-environment secrets only**. Live credentials exist only on the host. | CI config review |
| SEC-06 | Human accounts (email, GitHub, broker, Telegram, password manager) use unique passwords + TOTP 2FA minimum; hardware key where supported. | DOC (annual checklist) |

## 3. Host & Network

| ID | Requirement | Verification |
|---|---|---|
| SEC-07 | **No public inbound ports.** Admin access via Tailscale only; SSH key-only, no root login, no password auth. | Host audit (quarterly) |
| SEC-08 | Egress allowlist: broker domains, data sources, Telegram, Healthchecks, Tailscale; everything else denied by firewall. | Host audit |
| SEC-09 | Platform services run as a **dedicated non-login service user** with systemd hardening: `NoNewPrivileges`, `ProtectSystem=strict`, `PrivateTmp`, `ProtectHome`, capabilityBoundingSet minimal. | Unit file review |
| SEC-10 | Unattended security upgrades enabled; minimal package set; firewall (ufw/iptables) default-deny inbound. | Host audit |

## 4. Application Security

| ID | Requirement | Verification |
|---|---|---|
| SEC-11 | All external input (API responses, files) is schema-validated (pydantic); malformed input fails closed. | UT |
| SEC-12 | No `eval`, `exec`, dynamic imports from strings, or shell-out with interpolated input. Static check (bandit) in CI. | CI (blocking) |
| SEC-13 | Order-path defense-in-depth: risk caps are enforced **in code** independent of configuration (FR-RK-06), so a config mistake cannot bypass them. | UT (property tests) |
| SEC-14 | Telegram bot: allow-listed chat ID only; commands authenticated; bot token treated as a secret; `/kill` works but nothing else is state-changing. | IT |
| SEC-15 | All external API usage respects rate limits and terms of service (no scraping, no reverse-engineered endpoints — RL-5). | Design review |
| SEC-16 | Audit log tamper-evidence (hash chain) and off-host backup of the chain head. | UT + OPS-06 |

## 5. Supply Chain

| ID | Requirement | Verification |
|---|---|---|
| SEC-17 | Dependencies: minimal set; **lockfile with hashes**; CI fails on unlisted/hash-mismatched installs. | CI |
| SEC-18 | `pip-audit` (vulnerability scan) in CI on every PR and weekly schedule; findings gate releases. | CI |
| SEC-19 | New dependencies require a documented justification + threat-model trigger review ([SDLC §5](../05-security/04-secure-development-lifecycle.md)). | PR template |
| SEC-20 | Monthly dependency-update cadence with lockfile diff review. | OPS log |

## 6. Detection & Response

| ID | Requirement | Verification |
|---|---|---|
| SEC-21 | Any broker-key revocation is completable within **15 minutes** using RB-003, drilled at least once before live trading (G2 gate) and annually. | DR |
| SEC-22 | Anomalous conditions (unknown order state, position mismatch, unexpected API behavior) alert immediately and halt trading (INV-1). | IT |
| SEC-23 | Kill switch reachable via ≥ 2 independent paths (local + Telegram) with independent failure modes. | DR (drill) |
| SEC-24 | Security events (auth failures on SSH, new Tailscale devices, config-tamper detection) monitored and alerted. | MON |
| SEC-25 | Workstation hygiene: full-disk encryption, automatic lock, no shared use, patched within 14 days. | DOC (annual checklist) |

## 7. Threat-Coverage Map (Summary)

| Threat (from [Threat Model](../05-security/02-threat-model.md)) | Primary controls |
|---|---|
| T-01 Credential leak (repo/logs/chat) | SEC-01/03/05, gitleaks, rotation SEC-04 |
| T-02 Host compromise | SEC-07/08/09/10, egress allowlist, no inbound |
| T-03 Malicious dependency | SEC-17–20, minimal deps, hashed lockfile |
| T-04 Broker account takeover (human-side) | SEC-02 (trade-only keys), SEC-06 (human 2FA), SEC-21 |
| T-05 Poisoned/faulty market data | FR-DA-02/03 (validation, staleness), INV-1 fail-closed |
| T-06 Manipulation of the bot's own orders (insider/fat-finger) | SEC-13, FR-RK group, kill switch, runbooks |
| T-07 Tampering with history/records | SEC-16 (hash-chained audit), off-site backup |

## 8. Explicit Non-Requirements

- Multi-tenant isolation (single operator, single account).
- Protection against a fully compromised workstation with an active logged-in
  broker session (accepted residual risk R-RES-1, see threat model §7).
- Zero-trust between components on the single host (local IPC is trusted).
