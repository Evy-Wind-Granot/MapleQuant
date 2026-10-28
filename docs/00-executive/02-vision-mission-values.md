# Vision, Mission & Values

| | |
|---|---|
| **Document ID** | DOC-002 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [Executive Summary](01-executive-summary.md) · [Product Overview](../02-product/01-product-overview.md) |

## 1. Vision

A world where any disciplined, self-directed investor can run institutional-grade
systematic trading on their own brokerage account — without paying for
infrastructure, without surrendering their data or capital to a platform, and
without needing to be a software company to do it safely.

## 2. Mission

Build and operate a **zero-fixed-cost, brokerage-agnostic automated trading
platform** that trades the Founder's own capital according to rigorously
validated rules — and to document the entire journey (architecture, security,
operations, compliance) to a professional standard, both as an engineering
discipline and as the foundation for any future product.

## 3. Values (Operating Principles)

| # | Value | What it means in practice |
|---|---|---|
| 1 | **Capital preservation first** | Risk limits are enforced in code, not in intentions. The system fails closed. No leverage, no derivatives, no shorting in Phase 1. |
| 2 | **Evidence over opinion** | No strategy trades real money without passing written validation standards and stage gates. Backtest honesty (no lookahead, no survivorship, no cherry-picked windows) is non-negotiable. |
| 3 | **Radical cost discipline** | $0/month fixed cost is a design constraint, not a limitation to be "fixed" later. Every dependency and service must justify its existence. |
| 4 | **Security by default** | Broker credentials are the crown jewels: least privilege, encrypted at rest, never in code or logs, instantly revocable. |
| 5 | **Auditability** | Every trading decision must be reproducible years later from immutable logs + config + code version. The broker is the source of truth; our books must always match it. |
| 6 | **Honesty about uncertainty** | Markets have no guarantees. Documents and metrics report drawdowns, failures, and near-misses with the same prominence as wins. |
| 7 | **Compliance is a feature** | Personal-use red lines are respected now so that commercialization, if ever, is an option rather than a liability. |
| 8 | **Simplicity is reliability** | One host, one process model, boring technology. Complexity is only added when a documented failure demands it. |

## 4. Non-Goals

Explicitly out of scope for the venture (revisit only with a formal decision):

- High-frequency or latency-sensitive trading.
- Cryptocurrency, derivatives, options, leverage, or short-selling.
- Managing money for any third party (family included) without registration
  analysis by securities counsel.
- Subscription/revenue products prior to legal review.
- "Beating the market" as an identity — the goal is disciplined,
  risk-controlled participation with an auditable process.

## 5. Culture & Working Style

- **Docs-first:** plans, requirements, and decisions (ADRs) are written before
  code is written or changed.
- **Small batches:** every change ships through CI with tests; no `main`
  branch direct commits.
- **Blameless postmortems:** incidents produce runbook fixes, not guilt.
- **Boring is beautiful:** the platform's success metric includes how few
  surprises it produces per quarter.
