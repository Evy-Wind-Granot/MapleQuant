# Threat Model

| | |
|---|---|
| **Document ID** | DOC-024 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [Security Overview](01-security-overview.md) · [Security Requirements](../03-requirements/04-security-requirements.md) · [Incident Response](05-incident-response-plan.md) |

Method: informal STRIDE per trust boundary, then scenario-driven analysis for
the top threats. Reviewed on trigger events (SEC-19): new dependency, new
external service, new broker, architectural change, incident.

## 1. Trust Boundaries

| # | Boundary | Crossers |
|---|---|---|
| B1 | Internet → host | Tailscale only; no inbound ports (SEC-07); egress allowlist (SEC-08) |
| B2 | Broker API ↔ platform | Adapter calls with trade-only scoped keys |
| B3 | Data sources ↔ platform | httpx fetches; pydantic validation on entry (SEC-11) |
| B4 | Operator ↔ Telegram bot | Allow-listed chat; commands: status/kill only (SEC-14) |
| B5 | CI ↔ host | CI holds paper secrets only (SEC-05); deploy via Tailscale |

## 2. STRIDE per Component

| Component | S (spoofing) | T (tampering) | R (repudiation) | I (info disclosure) | D (DoS) | E (elevation) | Primary SEC controls |
|---|---|---|---|---|---|---|---|
| Session process | — | config tamper (local) | **audit log = anti-repudiation** (FR-AU) | log secrets leak | restart (systemd) | service user is non-login | 09, 16, 03 |
| Notifier (Telegram) | fake chat **(allow-list)** | fake kill (impact = safe direction) | audit | token in logs **(forbidden)** | Telegram outage | — | 14, 01, 03 |
| Broker adapter | stolen key = spoofing as us | — | order lineage (FR-AU-02) | key leakage | broker outage → **fail-closed** | scope escalation impossible (issue-time check) | 02, 21 |
| Data ingestor | bad source | **poisoned data** | provenance (FR-DA-04) | — | stale data → **fail-closed** | — | 11, FR-DA-02/03 |
| Host (OS) | SSH spoofing (keys only) | file tamper | journald | — | resource exhaustion (limits) | **root compromise = game over** | 07–10 |
| CI/CD | stolen GH token | malicious workflow | audit trail | secrets (paper only) | — | runner escape → code tamper | 05, 17–19 |
| State DB / audit | — | **record tampering** | — | — | corruption → backups | — | 16, OPS-10/11 |

## 3. Top Threat Scenarios

### T-01 — Broker credential leak (repo/log/CI/chat)
**Path:** secret committed or logged → scraped by bots → orders placed.
**Impact:** unwanted trades; **no direct theft** (trade-only scopes, SEC-02).
**Controls:** gitleaks in CI (SEC-03); secrets never in code (SEC-01/05);
rotation ≤ 90 d (SEC-04); rate/order caps bound the damage (SEC-13);
revocation ≤ 15 min (SEC-21, RB-003). *Residual: a leak could trade until
revoked — bounded by caps and breakers.*

### T-02 — Host compromise (RCE via service or supply chain)
**Path:** exposed service or malicious dependency → shell as service user →
read credentials → place orders.
**Controls:** no inbound (SEC-07); egress allowlist exfil-blocks (SEC-08);
systemd hardening (SEC-09); hashed lockfile + pip-audit (SEC-17/18);
code-enforced risk maxima survive attacker control of config (SEC-13).
*Residual: a full host compromise lets the attacker trade within hard caps —
monitoring + revocation is the response path (RB-003, SEC-24).*

### T-03 — Malicious dependency (supply chain)
**Path:** typosquat/compromised package → install → backdoor.
**Controls:** minimal dependency set (SEC-19 review); lockfile with hashes
(SEC-17); pip-audit (SEC-18); monthly review cadence (SEC-20); egress
allowlist limits callback exfil.

### T-04 — Broker account takeover (human-side)
**Path:** phished email → password reset → broker UI compromise (worse than API).
**Controls:** unique passwords + TOTP everywhere (SEC-06); bot's trade-only
scope means the *API* surface can't move money even if the key leaks (SEC-02);
broker-side withdrawal controls live with the human, not the platform.
*Residual: operator's own broker session is outside this system's control.*

### T-05 — Poisoned or faulty market data
**Path:** bad data (source bug, manipulation near open, silent adjustment
change) → bogus signal → trade.
**Controls:** multi-source provenance (FR-DA-04); schema/range validation
(FR-DA-02); spike flags; staleness fail-closed (FR-DA-03); low-frequency
monthly decisions reduce exposure windows; replay/monitoring catches drift
(NFR-13).

### T-06 — Manipulation of the bot's own decisions (insider/fat-finger)
**Path:** operator config mistake, or a "helpful" manual tweak mid-session →
runaway orders.
**Controls:** config = code (OPS-15, schema-validated FR-CF-01); code-enforced
maxima ignore config (FR-RK-06); idempotency (FR-EX-02); kill switch + no
auto-resume (FR-RK-05); arm step for live (FR-EX-08).

### T-07 — Tampering with records/history
**Path:** someone (or a bug) edits books/audit to hide losses or fake tax basis.
**Controls:** append-only + hash-chained audit (FR-AU-01/03); chain head
replicated off-host (SEC-16); broker is independently authoritative (INV-4) —
books that disagree with the broker are *detectable by construction*.

## 4. Monitoring Hooks (detection assumptions)

The threat model only works if [monitoring](../06-operations/02-monitoring-and-alerting.md)
watches for: unknown order states, reconciliation breaks, order-rate spikes,
SSH/Tailscale auth anomalies, config-file change events, CI secret exposure,
heartbeat gaps. These are SEV2+ alert conditions, not dashboard-only metrics.

## 5. Attack-Tree Summary (one line each)

- Steal money: impossible via platform (no withdrawal scope); via broker
  account = human-security problem (SEC-06) — accepted split.
- Steal strategy: public repo already discloses family; docs private.
- Steal data: re-fetchable; low value.
- Sabotage trading (DoS): possible; fail-closed → cost is opportunity only.
- Damage records: detectable via broker cross-check + hash chain.

## 6. Security Event Review Triggers

Any of these re-opens this document: new broker, new external service, new
dependency with network/I/O surface, first live capital increase, any SEV1,
host OS major upgrade, annual review (whichever first).

## 7. Residual Risks (accepted, with rationale)

| ID | Residual risk | Why accepted |
|---|---|---|
| **R-RES-1** | A compromised operator workstation with an active broker session defeats platform controls | Human-side risk; mitigated by SEC-06 hygiene; platform can't defend the human's session — documented instead of pretended away |
| R-RES-2 | A leaked live key can trade (within caps) until revoked | Bounded by SEC-13 caps + breakers + detection (SEC-24); ≤ 15-min revocation drilled (SEC-21) |
| R-RES-3 | Third-party (Telegram/OCI) outages blind alerting or hosting | Independent heartbeat channel (Healthchecks); fail-closed trading posture; fallback host |
| R-RES-4 | Free-tier provider policy changes | Monthly cost review + DR runbook; worst case = downtime, not loss |
