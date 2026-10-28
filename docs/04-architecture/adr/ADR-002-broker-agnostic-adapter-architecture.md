# ADR-002: Broker-Agnostic Adapter Architecture

| | |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-10-27 |
| **Deciders** | Founder |

## Context

The Founder's current broker (Wealthsimple) does not yet have generally
available retail API access (OQ-01), and broker ToS/APIs can change or
disappear. Betting the engine on one broker couples strategy code to another
company's roadmap. Meanwhile, free paper trading (Alpaca) is needed for the
validation phase anyway. Requirement: switching brokers must not touch core
code.

## Decision

All broker interaction flows through a **`BrokerAdapter` protocol** with a
strict internal order/account model, capability flags, and a mandatory
**certification contract-test suite** ([Broker Adapter Spec](../03-broker-adapter-specification.md)).
The core engine contains zero broker-specific code, enforced by import lint in
CI. New brokers (or dropping one) is an adapter + config change.

## Consequences

**Positive:** broker leverage in negotiations/reality (no lock-in); paper
(Alpaca) and live (Wealthsimple or any CIRO-accessible broker) run the same
core; broker outages are contained as adapter-level failures.

**Negative:** adapters add an abstraction layer and must be certified (test
cost per broker); capability differences (e.g., fractional shares, OQ-02)
surface as runtime flags that strategies must respect.

**Mitigations:** certification suite runs on paper/sandbox endpoints; the
capability model is small and explicit; unsupported features fail closed.

## Alternatives Considered

| Alternative | Rejected because |
|---|---|
| Direct integration per broker | Locks research and risk code to a broker; massive switch cost |
| Adopting an existing multi-broker framework (LEAN) | Heavyweight; runtime/data costs; against solo-ops and $0 constraints; still US-centric for our needs |
| Brokerless/paper-only | No path to live trading — the venture's purpose |
