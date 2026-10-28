# ADR-001: Python as the Primary Language

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-10-27 |
| **Deciders** | Founder |

## Context

The platform is built and operated by a single person with a $0 budget. The
dominant workloads are data manipulation, research iteration, and API
integration — not low-latency execution (strategies are monthly; latency budget
is seconds, NFR-01). The quant ecosystem depth determines research velocity.

## Decision

Use **Python 3.12+** as the only implementation language for the engine, CLI,
and tooling. Use `uv` for environment/dependency management.

## Consequences

**Positive:** richest data/quant ecosystem (pandas, pydantic, exchange
calendars); solo-maintainable; every dependency in the [Tech Stack](../02-tech-stack.md)
is free and well-known; AI-assisted development works best against a
mainstream stack.

**Negative:** runtime performance is modest (irrelevant at this data scale);
GIL limits parallelism inside a session (irrelevant — the orchestrator is
sequential by design, Architecture §6).

**Mitigations:** strict typing (mypy) on core; property tests on risk math.

## Alternatives Considered

| Alternative | Rejected because |
|---|---|
| Rust | Dev-velocity cost too high for a solo operator; ecosystem for Canadian tax/quant niceties is thin |
| Node.js/TypeScript | Weaker data/quant libraries |
| Mixed stack | Two ecosystems to maintain; violates simplicity value (#8) |
