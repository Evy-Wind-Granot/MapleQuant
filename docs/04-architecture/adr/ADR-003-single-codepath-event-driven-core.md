# ADR-003: Single Code Path — Custom Event-Driven Core for Backtest + Live

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-10-27 |
| **Deciders** | Founder |

## Context

The classic failure mode of retail trading bots is a backtest engine that
differs subtly from the live engine: fills modeled one way, data alignment off
by a bar, risk checks that exist only in production. Every difference is a
place where live behavior silently diverges from research. Requirement
FR-ST-06 (replay equivalence) demands structural sameness.

## Decision

Build a **thin custom event-driven core** (orchestrator + data window + risk
gate + execution manager) that runs **unchanged** in three modes:
backtest (historical window + SimBroker), paper (live data + paper broker),
live (live data + real broker). Drivers are swappable
([ADR-002](ADR-002-broker-agnostic-adapter-architecture.md)); the core is one
code path. A CI golden test enforces decision-level equivalence.

Do **not** adopt a backtest-only library for research and a separate live
stack.

## Consequences

**Positive:** what you backtest is what you trade — by construction; one
auditing/monitoring surface; bug fixes apply everywhere at once; the codebase
stays small (~few thousand lines, not tens of thousands).

**Negative:** we own the engine (no community library for fills/metrics);
vectorized ultra-fast parameter sweeps are slower than numpy-native research
tools (acceptable: sweeps run in minutes on 12 ETFs).

**Mitigations:** validation standards catch engine bugs
([Testing Strategy](../../07-quality/01-testing-strategy.md)); research-scale
exploratory work may use throwaway notebooks, but any *accepted* result must
be reproduced through the core before it counts (see
[Validation Standards §2](../../07-quality/02-backtesting-validation-standards.md)).

## Alternatives Considered

| Alternative | Rejected because |
|---|---|
| Backtrader/vectorbt for research + custom live | Two code paths = the exact divergence this ADR exists to prevent; their live-trade support is dated |
| LEAN / NautilusTrader as the core | Large codebases to master and operate; too much engine for 12-ETF monthly rotation; Nautilus is excellent but aimed at faster/multi-venue trading |
| Notebook-only research | No reproducibility, no risk integration, no audit — fails INV-5 |
