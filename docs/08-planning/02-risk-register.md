# Risk Register

| | |
|---|---|
| **Document ID** | DOC-037 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [Threat Model residual risks](../05-security/02-threat-model.md) · [Risk invariants](../04-architecture/01-system-architecture.md) · [Compliance red lines](../01-market-compliance/03-regulatory-compliance.md) |

**Scales:** Likelihood (L) and Impact (I): **L**ow / **M**edium / **H**igh.
Score is a qualitative L·I product (H·H = Critical; H·M or M·H = High;
M·M or H·L or L·H = Medium; rest = Low). Review: quarterly + after any SEV1
postmortem + at each stage gate.

| ID | Risk | L | I | Score | Primary mitigations | Residual posture |
|---|---|---|---|---|---|---|
| **RSK-01** | Strategy underperforms / regime change makes momentum unprofitable | H | M | High | Validation gates (OOS-based G1); defensive asset by design; drawdown breaker at 10%; capital sized as tuition; step-ups gated on clean quarters | Accepted (market risk is the venture's reason; bounded by sizing) |
| **RSK-02** | Overfitting — backtest flattery from tuning | M | H | High | Walk-forward OOS dominance; frozen holdout; ±20% stability gate; append-only experiment log (multiple-testing discipline) | Managed, never eliminated |
| **RSK-03** | Regulatory scope creep — drift toward advice/third-party money reclassifies the activity | L | H | Medium | Red lines RL-1…RL-7; counsel gate at P4; annual compliance re-read (CT-06) | Low if discipline holds |
| **RSK-04** | Broker API credential compromise | L | H | Medium | Trade-only scopes (SEC-02); encrypted at rest (SEC-01); rotation (SEC-04); revocation ≤ 15 min drilled (SEC-21); caps bound blast radius (SEC-13) | Bounded: attacker can trade, not steal; see [T-01](../05-security/02-threat-model.md) |
| **RSK-05** | Wealthsimple Trade API access unavailable/revoked by G3 | M | M | Medium | Broker-agnostic adapters (ADR-002); Alpaca/IBKR fallback path; slip rule — live waits, never ships unofficial | Schedule risk only |
| **RSK-06** | Free-tier policy change / OCI idle reclamation | M | M | Medium | Heartbeat detects same-day; home-server fallback (OQ-03); RB-008 rebuild ≤ 4 h; monthly policy review ([Cost §3](../06-operations/05-cost-management.md)) | Downtime, not loss (fail-closed) |
| **RSK-07** | Data licensing/ToS breach (esp. yfinance) | L | M | Medium | Source register with caveats; personal-use posture; licensing review before any non-personal use; yfinance never in a product | Managed by scope discipline |
| **RSK-08** | Reconciliation drift / accounting error corrupts books | L | H | Medium | Broker is source of truth (INV-4); reconcile every session; goldens for ACB/P&L; zero-tolerance breaks; quarterly restore drills | Detected by construction |
| **RSK-09** | Execution-path defect — duplicate/stuck orders | M | H | High | Idempotency keys (FR-EX-02); query-before-retry (FR-EX-03); unknown-state = worst case (FR-EX-04); fault-injection tests; risk maxima in code | The most-tested code in the system |
| **RSK-10** | Operator error — bad config, panic action | M | M | Medium | Schema validation; code-enforced maxima (FR-RK-06); config = code PRs; arm step; no auto-resume after breakers; runbooks | Bounded by hard caps |
| **RSK-11** | FX friction drag (historically ~1.5% conversion at Wealthsimple — verify, CT-04) | H | M | High | Modeled in every backtest; low-turnover monthly design; USD sub-account usage to avoid round-trips | Accepted cost of structure |
| **RSK-12** | Host compromise (RCE) | L | H | Medium | No inbound; Tailscale; egress allowlist; systemd hardening; minimal packages; hashed lockfile; audit tamper-evidence | Residual R-RES-2 in [threat model](../05-security/02-threat-model.md) |
| **RSK-13** | Supply-chain compromise (malicious dependency) | L | H | Medium | Minimal deps with justification; uv lockfile + hashes; pip-audit weekly; monthly update review (SEC-17–20) | Standard OSS residual |
| **RSK-14** | Tax misreporting (ACB/FX errors) | L | M | Medium | ACB engine with golden fixtures; official BoC rates per transaction; 7-year retention; accountant review as volume grows | Low; audit trail supports correction |
| **RSK-15** | Single-operator bus factor (illness/life) | M | M | Medium | Docs-first culture; runbooks for every SEV scenario; DR drills; fail-closed means absence = no trades, not chaos | The docs *are* the mitigation |
| **RSK-16** | Alert channel failure (Telegram outage) blinds operator | L | M | Medium | Independent heartbeat channel (Healthchecks); local kill path independent of Telegram; redundancy option OQ-10 | Detected via heartbeat gap |
| **RSK-17** | Broker outage while positions open | M | L | Medium | Positions are liquid ETFs; monthly cadence means no intraday dependence; fail-closed posture; next session auto-reconciles | Accepted (market doesn't need us to watch it tick) |
| **RSK-18** | Gap risk — overnight/weekend price gaps through stops/expectations | H | M | High | Diversification across ≤ 3 assets + defensive sleeve; drawdown breaker; capital cap; expectations set by gap-aware backtest (no same-bar fills) | Accepted: this is equity investing |

## Top of Register (by score)

**RSK-09 (execution defect), RSK-01 (strategy risk), RSK-02 (overfitting), RSK-11 (FX drag), RSK-18 (gap risk).**
The pattern is deliberate: the register spends its heaviest mitigation effort on
*things we control* (execution, process) and treats *market risks* (1, 11, 18)
as priced-in constraints — sized capital is the mitigation, not cleverness.

## Register Governance

1. Owner: Founder (sole role).
2. New risks enter via: incident postmortems, threat-model reviews (SDLC §5),
   quarterly review, or stage-gate preparation.
3. A risk is only closed by a structural change (control added/removed), never
   by optimism; changes are recorded in this file's revision history.
4. Security-specific residual risks live in the
   [Threat Model §7](../05-security/02-threat-model.md) and are cross-referenced,
   not duplicated, here.
