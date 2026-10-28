# Deployment & Environments

| | |
|---|---|
| **Document ID** | DOC-028 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [ADR-005](../04-architecture/adr/ADR-005-zero-cost-hosting.md) · [ADR-008](../04-architecture/adr/ADR-008-repository-strategy.md) · [SDLC CI gates](../05-security/04-secure-development-lifecycle.md) · [Release Management](../07-quality/03-release-and-change-management.md) · OPS-01–05 |

## 1. Environments (one host, three profiles — OPS-01)

| Profile | Config | Credentials | DB | Mode |
|---|---|---|---|---|
| dev | local overrides | none/paper | scratch | backtest/paper |
| paper | `paper.yaml` | paper keys | `paper.db` | paper |
| live | `live.yaml` | live keys (trade-only, SEC-02) | `live.db` | live + **arm step** (FR-EX-08) |

**Parity rule:** same binary, same code path (ADR-003), same calendars/data;
differences are *only* config, credentials, and database. Any behavior
difference beyond those is a defect.

## 2. Host Provisioning (checklist — also the RB-008 rebuild script baseline)

1. OCI Always Free ARM VM, Ubuntu LTS, SSH key-only ([ADR-005](../04-architecture/adr/ADR-005-zero-cost-hosting.md)).
2. `ufw` default-deny inbound; allow outbound to allowlist only (broker, data
   sources, Telegram, Healthchecks, Tailscale) — SEC-08.
3. Tailscale installed; SSH reachable via tailnet only — SEC-07.
4. `unattended-upgrades` security pocket; NTP enabled (OPS-19).
5. Create `maplequant` non-login service user (SEC-09); systemd units with
   hardening options.
6. Install release (wheel + SBOM); import secrets into systemd credentials
   (SEC-01); **never** via files in the repo checkout.
7. Provision systemd units:
   - `mq-scheduler.timer` → `mq-session.service` (one-shot per session, per
     trading calendar, OPS-03)
   - `mq-notifier.service` (long-running: Telegram listener + heartbeats)
8. Run smoke tests (§4); record in deployment log (§6).

## 3. Release Pipeline

```
main → PR + full CI gates (SDLC §3)
     → tag vX.Y.Z → CI builds wheel + SBOM + config-schema report
     → GitHub Release (public repo, ADR-008)
     → deploy over Tailscale: install wheel into /opt/maplequant/<tag>/
     → point systemd at the new tag (symlink flip)
     → smoke test paper profile
     → run ≥ 1 full paper session (OPS-05)
     → promote to live profile (config flip) — live re-arms deliberately (FR-EX-08)
```

Live credentials never pass through this pipeline (SEC-05): they live on the
host in systemd credentials and are bound to the `live` profile at service
start.

## 4. Smoke Tests (post-deploy, pre-promotion)

| Check | Command / expected |
|---|---|
| Config validity | `mq status` — schema + sanity validation exits clean (FR-CF-01) |
| Data freshness | last bars/freshness gates pass (FR-DA-03) |
| Broker connectivity | adapter handshake + account snapshot (paper) |
| Reconciliation dry-run | `mq reconcile --dry-run` clean on paper (FR-RC-01) |
| Kill switch round-trip | set + clear + verify audit entries (FR-RK-04) |
| Notification round-trip | test digest arrives (FR-NT-01) |
| Heartbeat | first ping visible in Healthchecks (OPS-06) |

## 5. Rollback (NFR-17: ≤ 30 min)

1. Flip the symlink back to the previous tag's install directory.
2. `systemctl restart` the affected units.
3. If a state migration shipped with the failed release, roll forward the
   *code* but keep the old DB schema if compatible — **migrations must be
   backward-compatible one minor release** (engine policy).
4. Smoke tests (§4) — then record the rollback in §6 with reason + linked issue.

## 6. Deployment Log (append-only)

| Date | Tag | Profiles | Smoke result | Notes |
|---|---|---|---|---|
| — | — | — | — | *(no deployments yet)* |

## 7. Cadence & Rules of Engagement

- Deploy windows: outside market hours; never between signal generation and
  order submission (OPS-12).
- Paper always sees a release before live (≥ 1 full session, OPS-05).
- Config-only changes follow the same PR path (config = code, OPS-15) but may
  apply without a binary release — hash recorded either way (FR-CF-05).
- Emergency path: security-fix releases may skip the paper soak **only** with
  a postmortem note justifying it.
