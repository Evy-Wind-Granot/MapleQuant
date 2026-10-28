# Release & Change Management

| | |
|---|---|
| **Document ID** | DOC-035 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [SDLC §2](../05-security/04-secure-development-lifecycle.md) · [Deployment pipeline](../06-operations/01-deployment-and-environments.md) · [Roadmap gates](../08-planning/01-roadmap.md) · [Testing DoD](01-testing-strategy.md) |

## 1. Versioning

- **Semantic versioning** for the engine package; releases are tagged and
  built ([Deployment §3](../06-operations/01-deployment-and-environments.md)).
- `v0.x` — pre-live development (P0–P2). Breaking config changes allowed with
  migration notes.
- **`v1.0` = the G3 release** (first live-capable version). Post-1.0: config
  schema changes are versioned and migrated; DB migrations must stay
  backward-compatible one minor release (Deployment §5).
- Every release ships: wheel, SBOM, config-schema report, release notes, and
  the tag's RTM status snapshot.

## 2. Change Flow (all changes)

```
issue/ADR (what & why) → branch → implementation + tests (DoD) → PR
  → CI gates (SDLC §3) → self-review checklist → merge
  → tag (if releasing) → deploy paper → soak → promote live (gated)
```

- **Config = code** (OPS-15): same PR path, schema-validated, hash recorded
  (FR-CF-05). A config diff appears in the next session report (FR-CF-06).
- No direct commits to `main` — including "tiny" ones, including config, no
  exceptions. The one-person team does not get to be the exception to the
  rule that protects the one-person team.

## 3. Strategy Changes (special class)

| Change | Required path |
|---|---|
| New strategy | Validation report per [Validation Standards](02-backtesting-validation-standards.md) (full G1) → paper ≥ 20 sessions → deliberate promotion |
| Parameter change (existing, live) | Re-validation (at minimum §6 stability + holdout re-check) → paper soak → promotion. **Never straight to live** |
| Universe change | Treated as a parameter change + universe-freeze rules |
| Engine change affecting fills/metrics | Replay-equivalence golden test (FR-ST-06) + re-run of the frozen G1 report to confirm no result drift |

## 4. Promotion Gates (paper → live)

1. Release ran on the **paper profile ≥ 1 full session** without SEV1/SEV2
   (OPS-05).
2. Smoke tests green on the host ([Deployment §4](../06-operations/01-deployment-and-environments.md)).
3. Reconciliation clean post-deploy.
4. For the **first** live promotion (G3) the full G2 checklist applies
   ([Testing §6](01-testing-strategy.md)); subsequent promotions use this
   section's gates.
5. Live arming is always a deliberate human step (FR-EX-08) — deployment
   tooling can never arm.

## 5. Emergency Path

Security fixes (e.g., vulnerable dependency actively exploited) may skip the
paper soak: deploy → smoke → arm with a postmortem note recording the reason
and the compensating checks done. The note is mandatory; skipping the soak
silently is a process violation.

## 6. Definition of Done (merged = releasable)

Every merged PR has: tests at the right pyramid level · CI green · docs
updated (requirements/RTM row, runbook, ADR where applicable) · changelog
entry · no TODOs on invariant-bearing paths (INV-1…7).

## 7. Release Log (append-only)

| Date | Tag | Summary | Promotion notes |
|---|---|---|---|
| — | — | *(no releases yet)* | — |
