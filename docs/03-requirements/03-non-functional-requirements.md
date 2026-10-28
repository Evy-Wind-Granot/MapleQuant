# Non-Functional Requirements

| | |
|---|---|
| **Document ID** | DOC-011 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [SRS](01-software-requirements-specification.md) · [Security Requirements](04-security-requirements.md) · [Operational Requirements](05-operational-requirements.md) |

> Quantitative targets below are **design budgets**, chosen for a low-frequency
> monthly-decision system on free-tier infrastructure. They are reviewed at
> each stage gate.

## 1. Performance

| ID | Requirement | Target | Verification |
|---|---|---|---|
| NFR-01 | Decision latency: bar availability → order submission | ≤ 5 s (p95) | MON + UT |
| NFR-02 | Order submission → broker ack | ≤ 2 s typical (broker-dependent) | MON |
| NFR-03 | Daily session wall-clock (data refresh + decision + report) | ≤ 10 min | MON |

*Justification:* strategies are monthly/weekly; sub-second latency buys nothing.
No HFT requirements by design.

## 2. Reliability & Availability

| ID | Requirement | Target | Verification |
|---|---|---|---|
| NFR-04 | Scheduled sessions succeed | ≥ 98% during P2/P3 (fail-closed failures count against this) | MON |
| NFR-05 | Missed-run detection | ≤ 15 min after expected fire | MON (heartbeat) |
| NFR-06 | Crash recovery: process restart mid-session | Safe resume via idempotency keys; never duplicate orders | UT (fault injection) |
| NFR-07 | **Maintainability for a solo operator:** ≤ 30 min/week steady-state ops; changes shippable by one person in one evening | Coverage ≥ 85% on core modules; strict typing on core; docs kept current | CI + ops log |
| NFR-08 | Durability: acceptable state loss window | RPO ≤ 24 h (DB backups; broker is source of truth for positions) | DR drill |

## 3. Capacity

| ID | Requirement | Target |
|---|---|---|
| NFR-09 | Universe size | ≤ 100 symbols (Phase 1: ~12 ETFs) |
| NFR-10 | History depth | ≥ 10 years daily OHLCV + intraday samples; Parquet cold storage; hot SQLite ≤ 1 GB |
| NFR-11 | Host | 1 vCPU / 1 GB RAM sufficient (OCI Always Free ARM is ~4× headroom) |

## 4. Observability & Auditability

| ID | Requirement | Target | Verification |
|---|---|---|---|
| NFR-12 | Every session outcome and every order visible in structured (JSON) logs | 100% | IT |
| NFR-13 | Reproduce any decision from audit log + snapshot + code version | 100% of trading decisions | UT + DOC |
| NFR-14 | Audit log retention | ≥ 7 years | OPS-06 + DOC |

## 5. Portability & Deployment

| ID | Requirement | Target |
|---|---|
| NFR-15 | Single-host Linux; no cloud-vendor lock-in (standard systemd + Python; state in files/SQLite) |
| NFR-16 | Rebuild host to operating state (fresh machine → trading) | ≤ 4 h using DR runbook (RB-008) |
| NFR-17 | Rollback to previous release | ≤ 30 min |

## 6. Usability (Operator-Facing)

| ID | Requirement | Target |
|---|---|---|
| NFR-18 | Every SEV1/SEV2 scenario has a runbook | 100% coverage; quarterly review |
| NFR-19 | CLI-first; every operator task achievable without remembering internals | DOC (runbooks) |
| NFR-20 | Alert messages actionable: state, cause, next step (runbook link) | IT |

## 7. Localization

| ID | Requirement |
|---|---|
| NFR-21 | America/Toronto operating timezone; UTC storage; NYSE calendar governs sessions |
| NFR-22 | CAD base currency for all reporting; USD positions translated at official BoC rates |
| NFR-23 | Canadian tax semantics (ACB, superficial loss) are first-class, not add-ons |

## 8. Cost

| ID | Requirement | Verification |
|---|---|---|
| NFR-24 | **Hard $0/month fixed infrastructure cost** (see [Cost Management Policy](../06-operations/05-cost-management.md)); trading capital is the only money at risk | Monthly cost review (OPS-13) |

## 9. Security & Compliance

- Security requirements are specified separately and carry their own IDs
  ([Security Requirements](04-security-requirements.md)) — they are **not**
  downgraded NFRs.
- Compliance posture (personal-use perimeter) is defined in
  [Regulatory & Compliance](../01-market-compliance/03-regulatory-compliance.md);
  the platform enforces its side via FR-AU, FR-TX, FR-RC groups.

## 10. Trade-Offs Accepted

| Trade-off | Rationale |
|---|---|
| Best-effort availability (no HA cluster) | $0 constraint; fail-closed design means downtime costs opportunity, not safety |
| SQLite over Postgres | Single writer (the scheduler), trivially backed up; migration path documented ([ADR-006](../04-architecture/adr/ADR-006-storage-sqlite-first.md)) |
| Free data with licensing caveats | Personal research/paper use; commercial use re-opens licensing (see [Data Requirements §7](06-data-requirements.md), tied to OQ-07) |
| No 24/7 process model | Bot only needs to be alive around sessions; heartbeat covers the rest |
