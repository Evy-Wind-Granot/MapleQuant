# Testing Strategy

| | |
|---|---|
| **Document ID** | DOC-033 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [SDLC CI gates](../05-security/04-secure-development-lifecycle.md) · [FR groups](../03-requirements/02-functional-requirements.md) · [Adapter Certification](../04-architecture/03-broker-adapter-specification.md) · [Validation Standards](02-backtesting-validation-standards.md) |

## 1. Philosophy

Solo development means **tests are the second reviewer**. The strategy biases
toward: property tests over example tests for math; fault injection over hope
for I/O; golden tests over eyeballing for anything that produces numbers; and
one end-to-end harness that runs a whole session in simulation so that "it
works" is reproducible by CI, not by memory.

## 2. Test Pyramid & Requirement Mapping

| Level | What | Covers | Runs |
|---|---|---|---|
| Unit | Pure functions: indicators, risk math, ACB lots, P&L, config schemas, calendar/FX logic | FR-DA-02/04/06, FR-RK-01…07 math, FR-PF-01…05, FR-CF-01/05 | every PR |
| Property | hypothesis: invariants under randomized inputs | INV-1…7, FR-RK-01 (caps), FR-EX-02 (idempotency), FR-BT-03 (no-lookahead) | every PR |
| Contract | Adapter certification suite (paper endpoints); data-source schema fixtures | [Adapter spec §5](../04-architecture/03-broker-adapter-specification.md), FR-EX-01…06, FR-DA-01/02 | every PR |
| System | Full session E2E on recorded data + SimBroker; replay equivalence golden test | Session sequence, FR-ST-06, FR-RC, FR-NT-04, fail-closed table ([Architecture §7](../04-architecture/01-system-architecture.md)) | every PR |
| Fault injection | Simulated failures in the order path | FR-EX-03/04, NFR-06 | every PR |
| CLI smoke | `mq` commands against a scratch profile | FR-CF-04, FR-EX-08 (arming), kill switch round-trip | every PR |
| Acceptance | 60-day paper period + drills | G2 gates (§6) | pre-G3 |

## 3. The Special Tests (specified precisely)

### 3.1 Truncation property test (no-lookahead — FR-BT-03)
For sampled decision dates T: run the strategy pipeline with data up to T, then
again with the *full* dataset available to the core. The decision at T must be
bit-identical. Any difference = lookahead — the test fails the build.

### 3.2 Idempotency fault injection (FR-EX-02/03)
Adapter harness injects: (a) timeout after broker receives the order, (b)
timeout before, (c) 500-then-success, (d) duplicate submit of the same
`client_order_id`. Assertions: exactly one live order ever; uncertain states
resolve via `get_order` query; no path blind-re-submits.

### 3.3 Risk-boundary property tests (INV-2, FR-RK-01/06)
Property: *no randomly generated config + signal combination can produce an
`ApprovedOrder` violating any hard maximum* (weights ≤ 40%, gross ≤ 100% NAV,
notional/count caps, whitelisted symbols). Also: constructing an order without
a risk approval must fail to typecheck/tests (bypass attempt fixture).

### 3.4 Replay equivalence golden test (FR-ST-06)
The same recorded day runs through the backtest pipeline and the live
pipeline (paper drivers, frozen clock). `TargetPortfolio`, order diffs, and
audit entries must match exactly. This is the single most important
regression test in the codebase.

### 3.5 Accounting goldens (FR-PF-02/03/04/05)
Fixture books exercise: split mid-holding; dividend reinvestment; return of
capital; sell + 30-day repurchase (superficial loss flag); USD trade with BoC
FX translation. Expected ACB/P&L values are hand-computed and pinned.

### 3.6 Fail-closed table test (Architecture §7)
Each row of the fail-closed table gets an injected trigger in the system
harness; the session must end with zero orders, an audit entry, and an alert
attempt. **A fail-closed gap is a SEV1-class bug, fixed before any merge.**

## 4. Test Data Management

- **Fixtures** (recorded API responses, golden books, calendar holidays) live
  in the repo, small and curated — no network in tests.
- **Snapshots**: full-data backtest snapshots referenced by hash
  (FR-DA-09/FR-BT-08) — big files stay out of git, in the cold store.
- No test ever touches live credentials (SEC-05) or the live DB, by CI
  environment construction.

## 5. Coverage Policy

- **Core modules** (`risk/`, `execution/`, `accounting/`, `engine/`,
  `adapters/`): ≥ 85% line coverage, blocking (SDLC §3).
- Property/golden tests count double: they're how 85% becomes *meaningful*,
  not gamed.
- Notifications/UI polish are held to "contract tests only" — coverage theater
  is explicitly not the goal.

## 6. Acceptance: The 60-Day Paper Period (G2 gates)

The paper phase *is* the final test environment. To pass G2 (→ G3 live):

| Gate criterion | Target |
|---|---|
| Clean paper days | **60 consecutive trading days**, no unresolved SEV1/SEV2 |
| Scheduled-run success | ≥ 98% (NFR-04) |
| Slippage vs. model | average ≤ 10 bps beyond modeled costs |
| Reconciliation | zero unresolved breaks over the period |
| Drills passed | kill-switch (both paths), credential revocation (≤ 15 min), DR restore |
| Code | coverage gate green on the promoted tag |

## 7. Definition of Done (any change)

1. Tests written at the pyramid level the change touches; property test if it
   touches math or invariants.
2. CI fully green (all SDLC §3 gates).
3. Docs updated: requirements row in the RTM, runbook if operational, ADR if
   architectural.
4. Changelog entry.
5. For execution/risk/auth surface: the §3.3 bypass-attempt review + cool-down
   re-read of the diff (SDLC §2).
