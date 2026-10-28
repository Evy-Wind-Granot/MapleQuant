# Strategy Engine Design

| | |
|---|---|
| **Document ID** | DOC-019 |
| **Version** | 0.1 (Draft) |
| **Owner** | Founder |
| **Date** | 2026-10-27 |
| **Related** | [FR-ST group](../03-requirements/02-functional-requirements.md) · [Backtesting Design](06-backtesting-design.md) · [Validation Standards](../07-quality/02-backtesting-validation-standards.md) · [Risk & Portfolio Design](05-risk-and-portfolio-design.md) |

## 1. Strategy Interface

```python
class Strategy(Protocol):
    id: str
    params: StrategyParams        # pydantic-validated, from config

    def decide(self, ctx: DecisionContext) -> TargetPortfolio:
        """Pure: (data window + params + injected time) → target weights + rationale."""

class TargetPortfolio:
    weights: dict[str, Decimal]  # symbol → target weight (0..1); sum ≤ 1.0
    cash_weight: Decimal         # explicit; no implicit leverage (INV-6)
    rationale: list[RationaleItem]  # indicator values, rankings, filters → audit (FR-ST-04)

class DecisionContext:
    data: DataWindow          # point-in-time, lookahead-safe (FR-BT-03)
    now: pd.Timestamp         # injected — never wall-clock (FR-ST-02)
    universe: UniverseView    # point-in-time membership (FR-BT-04)
    fx: FxTable               # CAD/USD official rates
```

Determinism rules (FR-ST-02) enforced by tests: no I/O, no randomness, no
datetime.now, no global mutable state. A strategy that violates this cannot
pass the double-replay test in CI.

## 2. Signal → Orders (portfolio translation)

The **engine** (not the strategy) translates a `TargetPortfolio` into orders:

1. Compare target weights vs. current book (post-reconciliation).
2. Compute order diffs; residual cash tracked (whole-share rounding, FR-EX-07).
3. Apply churn bands (configurable, default 5% weight drift) — small deviations
   do not trade, protecting against turnover drag.
4. Every diff becomes a **proposed** order → risk gate → execution.

Strategies never see balances, broker state, or order mechanics — they only
express *where capital should be*. Risk can always veto.

## 3. Reference Strategy: MQ-MOM-1 (v1)

**Class:** absolute + cross-sectional momentum on liquid US-listed ETFs.

| Element | Initial specification (parameters are config, values are starting points) |
|---|---|
| Universe | 8–12 liquid US-listed ETFs spanning equity regions/styles **plus** a short-duration Treasury ETF as the defensive asset (e.g., T-bill proxy); full list frozen at G1 (OQ-06) |
| Signal | Blended total-return momentum over 3/6/12-month lookbacks (average of the three) |
| Filter | **Absolute momentum:** candidate must have positive blended momentum and outperform the defensive asset; else weight goes to the defensive asset/cash |
| Selection | Top 3 by blended momentum |
| Weighting | Equal weight (⅓ each), subject to cap 35% per position (FR-RK-01 hard cap 40%) |
| Cadence | Monthly (first trading day; execute at/near next open); weekly/daily supported (FR-ST-05) |
| Churn control | 5% weight bands; ≤ 1 rebalance/month enforced |
| Constraints honored by construction | No leverage (weights sum ≤ 1), no shorting, no derivatives, ≤ 3 positions + defensive |

**Why this family for v1:** decades of literature on momentum; monthly cadence
matches free EOD data; low turnover matches commission-free but FX-frictioned
reality; robust to modest slippage assumptions; simple enough to validate
honestly under [Backtesting Validation Standards](../07-quality/02-backtesting-validation-standards.md).

**Explicit non-claims:** this is not a prediction of outperformance. It is a
*testable hypothesis* with defined acceptance gates. If it fails the gates, it
does not trade (G1 is a kill switch for bad research, too).

## 4. Engine-Strategy Contract Rules

| Rule | Detail |
|---|---|
| Lookahead safety is structural | `DecisionContext.data` is windowed by the core to `now`; strategies cannot request future bars (FR-BT-03) |
| Determinism is structural | Injected `now`; no clock/RNG imports allowed (lint rule) |
| Replay equivalence | Same inputs → same `TargetPortfolio` in backtest and live (FR-ST-06, CI golden test) |
| Rationale is mandatory | A decision without logged rationale is a defect, not a nicety (FR-ST-04) |
| Strategies can't trade | No order construction inside strategies — only targets (§2) |

## 5. Strategy Registry & Configuration

- Registry: entry-point/discovery map `strategy_id → factory`; config selects
  `strategy: MQ-MOM-1` + parameter block, validated against the strategy's
  pydantic schema (FR-CF-01).
- Parameter sets are first-class experiment artifacts (FR-BT-08) — comparisons
  are config diffs, not memory.
- New strategies enter through: validation report → paper gate → live cap
  (never straight to live; [Release Management](../07-quality/03-release-and-change-management.md)).

## 6. Future Strategy Queue (post-v1, evaluation only)

| Candidate | Class | Notes |
|---|---|---|
| MQ-TREND-1 | Time-series trend following (e.g., 10/12-month filters) on the same universe | Second opinion on regime |
| MQ-VAL-1 | Simple valuation tilt (e.g., CAPE-style) | Different data demands; slow cadence |
| MQ-VOL-1 | Volatility-targeted overlay of v1 | Uses FR-RK-08 sizing |

Nothing in the queue ships before v1 completes its supervised live period —
one strategy learning to behave is worth more than three half-validated ones.
