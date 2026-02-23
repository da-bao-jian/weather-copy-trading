# Polymarket Weather Copy Trading Bot - Design Document

**Mode**: Standard  
**Date**: 2026-02-23  
**Status**: Final  
**Type**: New Project

## Problem Statement

Build a production-safe copy trading bot that follows `0x594edB9112f526Fa6A80b8F858A6379C8A2c1C11` on Polymarket, while preserving the edge profile from `analysis.md` (high-selectivity weather strategy), controlling risk, and avoiding blind mirroring that can overpay or overexpose capital.

## Understanding

### Facts

- `analysis.md` shows the strongest historical profile at `48h + edge >= 0.30` with:
  - `n=62`
  - `roi≈0.3460`
  - peak concurrent positions `=5` (about `$500` capital at `$100/signal`)
- Higher-volume alternatives exist but with lower ROI:
  - `24h + edge>=0.15` (`n=615`, `roi≈0.0644`)
  - `24h + edge>=0.25` (`n=463`, `roi≈0.0951`)
- Recent target-wallet behavior (sample: latest 500 trades + 500 activity rows):
  - Trade window sampled: `2026-02-20 10:15:30 UTC` to `2026-02-22 23:41:22 UTC`
  - Activity mix: `TRADE=472`, `MERGE=22`, `REDEEM=6`
  - Trades are weather temperature markets (F and C, US and non-US cities)
  - Both-sided behavior is common: 85/159 sampled conditions had both `Yes` and `No` buys
- Copying only one side can miss the source strategy when the leader is running paired positions and later `MERGE/REDEEM`.

### Context

- This is a trading system where latency and execution quality matter.
- A copy bot needs idempotency and reconciliation (missed events are expensive).
- The leader appears to trade many low-price `Yes` and high-price `No` legs; this matches a selective pricing/structure strategy, not random directional betting.

### Constraints

- Must follow a single wallet initially.
- Must support capital constraints and hard risk controls.
- Must tolerate API hiccups and duplicate events.
- Must be deployable as an always-on service.

### Unknowns (Resolved)

- [x] Is target history available from API? -> Yes (`trades`, `activity`, `positions` endpoints expose this wallet).
- [x] Does wallet lifecycle include non-trade actions? -> Yes (`MERGE`, `REDEEM` appear frequently).
- [x] Is the market universe aligned with weather strategy? -> Mostly yes (weather high-temp markets), but includes non-US and Celsius contracts.

### Unknowns (Open)

- [ ] Exact follower bankroll and acceptable max drawdown.
- [ ] Whether follower wants global weather markets or US-only (as in `analysis.md`).
- [ ] Whether to mirror exits exactly vs hold to resolution.
- [ ] Fee/slippage tolerance per trade for production thresholds.

## Research & Input

### Relevant Pattern 1: Full Mirror

Mirror every eligible leader trade with a fixed copy ratio.

### Relevant Pattern 2: Guardrailed Mirror

Mirror only trades passing strategy gates (edge, horizon, market type, risk budget).

### Relevant Pattern 3: Pair-Aware Copy

Detect conditions where leader buys both sides (`Yes` and `No`) and treat as a bundle for optional merge-style lifecycle.

## Solutions Considered

### Option A: Pure Wallet Mirror

**Approach**: Copy every leader `TRADE` one-for-one (scaled).

**Pros**:
- Highest fidelity to leader behavior
- Simplest logic
- Fastest to ship

**Cons**:
- Copies bad fills during latency spikes
- Can overexpose capital
- Misses strategic intent for paired/merge behavior

**Sacrifices**:
- Safety and controllability

### Option B: Guardrailed + Pair-Aware Mirror (Recommended)

**Approach**:
- Ingest leader events in near real-time
- Generate copy intents only if all guards pass:
  - market filter: weather high-temperature contracts
  - edge floor: default `edge_proxy >= 0.30`
  - horizon gate: default `18h-60h to end` (48h-like behavior)
  - slippage gate: reject if expected copy fill too far from leader fill
  - risk gate: per-market and portfolio caps
- Track two modes:
  - single-leg copy
  - pair-aware copy (`Yes` + `No` same condition) with optional merge lifecycle

**Pros**:
- Preserves strategy selectivity from analysis
- Lower blow-up risk
- Better fit for observed merge/pair behavior

**Cons**:
- More engineering complexity
- Some drift from strict “exact copy”
- Requires robust state management

**Sacrifices**:
- Perfect fidelity in exchange for robustness and risk-adjusted behavior

### Option C: Strategy Clone (No Wallet Following)

**Approach**: Rebuild the forecast-based strategy from `analysis.md` and trade independent signals.

**Pros**:
- Fully controllable
- No dependence on leader latency

**Cons**:
- Not copy trading anymore
- Requires full signal pipeline and forecasting infra
- Higher model risk

**Sacrifices**:
- Leader alpha capture

## Tradeoffs Matrix

| Criterion | Option A: Pure Mirror | Option B: Guardrailed Pair-Aware | Option C: Strategy Clone |
|-----------|------------------------|-----------------------------------|--------------------------|
| Leader fidelity | High | Medium | Low |
| Risk control | Low | High | High |
| Complexity | Low | Medium/High | High |
| Time to MVP | Fast | Medium | Slow |
| Alignment with `analysis.md` edge logic | Low | High | High |
| Handles merge/pair behavior | Low | High | Medium |

## Recommendation

Choose **Option B: Guardrailed + Pair-Aware Mirror**.

**Reasoning**:
- User intent is copy trading, so we should keep leader-following as the core.
- `analysis.md` suggests edge selectivity matters; pure mirroring ignores that.
- Observed `MERGE/REDEEM` behavior indicates structure-sensitive execution; pair-aware logic is needed to avoid copying only fragments of the strategy.

## Architecture Overview

```text
Leader Wallet API Poller
  -> Event Normalizer (TRADE/MERGE/REDEEM, dedupe)
    -> Signal Engine (single-leg + pair-aware intents)
      -> Risk Engine (capital, exposure, drawdown, slippage gates)
        -> Execution Engine (CLOB orders, retries, idempotency)
          -> Position Lifecycle Manager (hold/exit/merge/redeem policy)
            -> Metrics + Alerts + Reconciler
```

## Implementation Plan

### Phase 1: Foundation (Data + Config)

1. Create config schema:
   - `leader_wallet`
   - `copy_ratio`
   - `min_edge` (default `0.30`)
   - `min_hours_to_end`, `max_hours_to_end`
   - `max_open_conditions`, `max_notional_per_condition`, `daily_loss_limit`
   - market scope (`global_weather` or `us_fahrenheit_only`)
2. Create storage tables:
   - `leader_events_raw`
   - `copy_intents`
   - `orders`
   - `positions`
   - `risk_snapshots`
3. Implement dedupe key: `tx_hash + condition_id + outcome + event_type`.

### Phase 2: Leader Ingestion + Reconciliation

1. Poll leader `activity` and `trades` endpoints on short intervals.
2. Normalize event timestamps and condition metadata.
3. Add reconciler job (e.g., every 5-15 min) against leader `positions` to detect missed state.

### Phase 3: Signal Engine (Core Strategy Logic)

1. **Single-leg intent**:
   - Build intent from leader trade if market/risk gates pass.
   - `edge_proxy = 1 - leader_fill_price` (outcome-relative, configurable).
2. **Pair-aware intent**:
   - Maintain rolling window per `condition_id`.
   - If both outcomes appear within window, mark pair bundle.
   - Prioritize pair execution when combined expected cost + fees is favorable.
3. Horizon filter:
   - Use market end time; default to 48h-like band.

### Phase 4: Execution + Risk Controls

1. Implement execution adapter to place signed CLOB orders.
2. Order policy:
   - default limit orders with slippage cap
   - cancel/reprice timeout
3. Risk controls:
   - hard cap concurrent conditions (start with `5` to match high-selectivity profile)
   - per-condition notional cap
   - daily drawdown kill switch
   - circuit breaker after repeated rejects/failures

### Phase 5: Exit Policy + Lifecycle

1. Support configurable exit modes:
   - `mirror_exit`: follow leader exits when available
   - `hold_to_resolution`: redeem at settlement
   - `pair_merge_mode`: merge paired positions when profitable/eligible
2. Ensure lifecycle engine handles `MERGE/REDEEM` events idempotently.

### Phase 6: Validation and Rollout

1. Build historical replay using saved leader events.
2. Run paper-trading mode for at least 7 days.
3. Promote to small-cap live mode with strict limits.
4. Scale limits only after stability + slippage checks pass.

## Success Metrics

- Copy coverage: `% of eligible leader events processed and attempted`.
- Fill quality: follower fill vs leader fill slippage distribution.
- Risk adherence: zero limit breaches.
- Uptime: event loop and execution service availability.
- Strategy outcome: net ROI, drawdown, and capital utilization vs configured targets.

## Open Questions

- Should v1 include only US Fahrenheit markets to match historical analysis exactly?
- What is the initial bankroll and max tolerable daily drawdown?
- Do you want strict copy fidelity or better risk-adjusted returns when these conflict?
- For paired trades, do we require both legs before executing, or allow staggered fills?
