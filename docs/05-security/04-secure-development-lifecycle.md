# Secure Development Lifecycle (SDLC)

| | |
|---|---|
| **Document ID** | DOC-026 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [Secure SDLC requirement source: SEC-11–20](../03-requirements/04-security-requirements.md) · [Tech Stack toolchain](../04-architecture/02-tech-stack.md) · [Testing Strategy](../07-quality/01-testing-strategy.md) |

## 1. Principles

Solo development is the risk: no second reviewer. Compensations: CI does the
reviewing, checklists do the remembering, and boundaries are in types and
tests rather than opinions.

## 2. Repository & Branch Policy

| Rule | Detail |
|---|---|
| Branch protection | `main` protected: no direct pushes, PR + green CI required, linear history |
| Commits | Signed; conventional messages; one logical change per PR |
| PR checklist (template enforced) | Tests added? New dependency justified? Secrets impossible? Security-relevant surface changed? Docs updated? |
| Two-person rule (approximated) | For risk-gate / execution / auth changes: self-review after a cool-down + diff read line-by-line + property tests proving the invariant still holds (INV-2, SEC-13) |
| Config changes | Treated as code: PR, review, schema validation, hash recorded (OPS-15) |

## 3. CI Gates (all blocking)

| Gate | Tool | Failure behavior |
|---|---|---|
| Lint/format | ruff | block |
| Types (core) | mypy strict | block |
| Tests + coverage | pytest (≥ 85% core) | block |
| Property tests | hypothesis (risk math, idempotency) | block |
| Secret scan | gitleaks (full history on new PRs) | block |
| Static security | bandit | block |
| Dependency CVEs | pip-audit | block |
| Lockfile integrity | uv lockfile hashes verified | block |
| Import rules | adapter/broker isolation lint (Architecture R2) | block |
| Replay equivalence | golden backtest = live-pipeline test (FR-ST-06) | block |

## 4. Dependency Policy

1. **Minimal by default** — every new dependency answers: what does it do that
   stdlib/pandas can't? Who maintains it? What's its transitive weight?
2. Justification recorded in the PR (SEC-19); license verified and noted in
   [Tech Stack](../04-architecture/02-tech-stack.md).
3. Lockfile with hashes is the only install path (SEC-17).
4. Monthly update cadence with lockfile diff review (SEC-20); security fixes
   ≤ 7 days.
5. Network- or I/O-heavy dependencies get an explicit threat-model look
   (see §5).

## 5. Threat-Model Review Triggers

Any of these **requires re-reading the [Threat Model](02-threat-model.md)** and
recording the conclusion in the PR (SEC-19):

- Adding a dependency with network, subprocess, or eval-like surface.
- Adding an external service / API integration (including new data source).
- Adding or changing a broker (new trust boundary — full STRIDE pass).
- Changes to auth, credential handling, kill-switch, or risk-gate code.
- Architectural changes (new process, new host, new ingress path).
- Any SEV1/SEV2 incident (postmortem includes model update).

## 6. Security Code Standards (core modules)

- All external data crosses a pydantic schema **at the boundary** (SEC-11) —
  no raw dicts from I/O ever reach core logic.
- No `eval`/`exec`/`__import__`-with-strings; no `shell=True` with
  interpolated input; no `pickle` on anything that ever crossed a trust
  boundary (SEC-12).
- `Decimal` for all money math (never float) — correctness *is* security for
  accounting.
- Fail-closed is the default exception posture: unknown → veto/alert (INV-1),
  never best-guess.
- Public API calls: explicit timeouts, bounded retries, no infinite loops
  (FR-EX-05/06).
- Secrets access only at service startup; never passed through function
  signatures or stored in objects that get logged (SEC-03).

## 7. Verification Cadence

| Activity | Frequency |
|---|---|
| CI full gate run | every PR |
| pip-audit scheduled scan | weekly |
| Host audit checklist (SEC-07–10) | quarterly |
| Kill-switch + revocation drill | before G3; annually after |
| Threat-model re-read | per §5 triggers; annually regardless |
