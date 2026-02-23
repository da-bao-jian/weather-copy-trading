# Polymarket Wallet Copy-Trading Bot - Technical Design

**Mode**: Standard (Hammock)
**Date**: 2026-02-23
**Status**: Final
**Leader Profile URL**: `https://polymarket.com/@0x594edB9112f526Fa6A80b8F858A6379C8A2c1C11-1762688003124?via=Tgweb3`
**Leader Address Input**: `0x594edB9112f526Fa6A80b8F858A6379C8A2c1C11`

## Problem Statement

Build a production-grade copy-trading bot that follows one Polymarket wallet and mirrors its trading actions with strict risk controls, deterministic behavior, and API-correct order construction.

This design intentionally does **not** depend on `analysis.md`.

## Research Baseline (Primary Sources)

This design is based on official Polymarket docs and OpenAPI specs:

- CLOB OpenAPI: `https://docs.polymarket.com/api-spec/clob-openapi.yaml`
- Data API OpenAPI: `https://docs.polymarket.com/api-spec/data-openapi.yaml`
- Gamma OpenAPI: `https://docs.polymarket.com/api-spec/gamma-openapi.yaml`
- Order creation docs: `https://docs.polymarket.com/trading/orders/create`
- Fees docs: `https://docs.polymarket.com/trading/fees`
- Auth docs: `https://docs.polymarket.com/api-reference/authentication`
- Rate limits docs: `https://docs.polymarket.com/api-reference/rate-limits`

## Scope and Non-Goals

### In Scope

- Follow one leader wallet.
- Mirror trade activity (`TRADE`) and optionally lifecycle activity (`MERGE`, `REDEEM`).
- Place follower orders on CLOB with correct signing and headers.
- Dynamic fee/tick handling per token.
- Risk gates, reconciliation, and kill switch.

### Out of Scope (v1)

- Multi-leader blending.
- Strategy optimization or alpha generation.
- Full market-making strategy.

## Canonical Identity Resolution

Polymarket profile addresses can be queried through Gamma public profile.

1. Call:
   - `GET https://gamma-api.polymarket.com/public-profile?address=<leader_address>`
2. Use `proxyWallet` from response as canonical user ID for Data API polling.
3. If `proxyWallet` is null, fall back to the provided address.

For this leader, current profile lookup returns:
- `proxyWallet = 0x594edb9112f526fa6a80b8f858a6379c8a2c1c11`

## API Surface and Responsibilities

### CLOB API (`https://clob.polymarket.com`)

- `POST /order`: place one order.
- `POST /orders`: place batch (max 15).
- `DELETE /order`: cancel one.
- `DELETE /cancel-orders`: cancel multiple.
- `DELETE /cancel-all`: cancel all.
- `DELETE /cancel-market-orders`: cancel by market/asset.
- `GET /book`: orderbook + `tick_size`, `min_order_size`, `neg_risk`.
- `GET /fee-rate` or `/fee-rate/{token_id}`: token fee rate.
- `GET /tick-size` or `/tick-size/{token_id}`: token tick size.
- `GET /time`: server timestamp.
- `GET/PUT /balance-allowance` and `GET /balance-allowance/update`: account balance/allowance data refresh.
- `POST /v1/heartbeats`: liveness heartbeat with `heartbeat_id` chaining.

### Data API (`https://data-api.polymarket.com`)

- `GET /activity`: user activity stream (`TRADE`, `MERGE`, `REDEEM`, etc.).
- `GET /trades`: trade records with `asset`, `side`, `size`, `price`, `transactionHash`.
- `GET /positions`: follower reconciliation.
- `GET /closed-positions`: optional reconciliation/reporting.

### Gamma API (`https://gamma-api.polymarket.com`)

- `GET /public-profile?address=...`: resolve canonical profile/proxy wallet.

## Critical API Contracts

### Authentication and Signing

### L1 (API key lifecycle)

Used for `POST /auth/api-key`, `GET /auth/api-keys`, `GET /auth/derive-api-key`.

Required headers:
- `POLY_ADDRESS`
- `POLY_SIGNATURE`
- `POLY_TIMESTAMP`
- `POLY_NONCE`

Documented L1 signing message:
- `timestamp + address + nonce`

### L2 (trading/private endpoints)

Used for order placement, cancels, heartbeats, balance/allowance, etc.

Required headers:
- `POLY_API_KEY`
- `POLY_PASSPHRASE`
- `POLY_TIMESTAMP`
- `POLY_SIGNATURE`
- `POLY_ADDRESS`

Documented L2 signing message:
- `timestamp + method + requestPath + body`

### Signature Types

Order `signatureType` is one of:
- `0`: EOA
- `1`: POLY_PROXY
- `2`: POLY_GNOSIS_SAFE

Bot config must select one signature type and use it consistently.

## Order Object (must be correct before signing)

Required fields in `order`:
- `maker`
- `signer`
- `taker` (`0x000...000` for open orders)
- `tokenId`
- `makerAmount`
- `takerAmount`
- `side` (`BUY` or `SELL`)
- `expiration`
- `nonce`
- `feeRateBps`
- `signature`
- `salt`
- `signatureType`

Payload wrapper fields:
- `owner`
- `orderType` (`GTC|GTD|FOK|FAK`)
- `deferExec` (default false)

All amount math uses fixed precision expected by CLOB tooling. For v1, use official CLOB client helpers for order construction/signing to avoid fixed-math and EIP-712 mismatches.

## Order-Type Policy (explicit)

| Order Type | Bot Use | Why |
|---|---|---|
| `FAK` | **Default for copy execution** | Fills immediately against available liquidity, allows partial fills, best copy coverage. |
| `FOK` | Optional strict mode | Use only when all-or-nothing is required (e.g., strict close). Fails often in thin books. |
| `GTC` | Optional maker mode only | Resting quotes; requires heartbeat and cancel/reprice loop. |
| `GTD` | Optional timed maker mode | Same as GTC but explicit expiry. Respect GTD 60s security threshold rule. |

### Final v1 default

- **Use `FAK` for copied `TRADE` events.**
- Do **not** use post-only in default mode.
- Keep `GTC/GTD` behind feature flag (`maker_mode=false` default).

### Post-only constraints

- Post-only orders are for maker behavior only.
- Valid only with `GTC`/`GTD`.
- Invalid with `FOK`/`FAK`.

## Leader Event Ingestion Design

### Source of truth

Use `GET /activity` as primary stream, filtered to the leader user.

Recommended query template:
- `/activity?user=<proxy_wallet>&type=TRADE,MERGE,REDEEM&limit=200&offset=0&sortBy=TIMESTAMP&sortDirection=DESC`

Use `GET /trades` as secondary/fallback verification stream:
- `/trades?user=<proxy_wallet>&limit=200&offset=0&takerOnly=false`

### Polling cadence

- `activity` loop: every 1 second.
- `trades` verification loop: every 2-3 seconds.
- Add jitter: 50-150ms.

This stays comfortably under documented Data API limits (`/activity` 200 req/10s, `/trades` 200 req/10s).

### Ordering and dedupe

Process events in ascending `(timestamp, transactionHash)` order.

Idempotency key:
- `transactionHash + type + conditionId + asset + side + size + price`

Persist raw and normalized events for replay.

## Intent Classification

### Event handling in v1

- `TRADE`: create copy intent (execute).
- `MERGE` and `REDEEM`: optional lifecycle intent (feature flag).
- `SPLIT/REWARD/CONVERSION/MAKER_REBATE`: ignore for trading execution.

### Side mapping

- Leader `BUY` -> follower `BUY` same token.
- Leader `SELL` -> follower `SELL` same token.

## Sizing Policy

### Inputs

- `copy_ratio` (default `0.20` = 20% of leader notional).
- `max_usdc_per_trade`.
- `max_usdc_per_market`.
- `max_total_usdc_exposure`.

### Calculations

- Leader notional estimate: `leader_notional = leader_size * leader_price`.
- Follower target notional: `min(leader_notional * copy_ratio, max_usdc_per_trade, remaining_risk_budget)`.
- Convert to shares using execution price bound.
- Enforce `min_order_size` from `/book`.

If result is below minimum size after rounding, skip intent.

## Price and Slippage Policy

### Market data per intent

Before sending order:

1. `GET /book?token_id=<asset>`
2. `GET /tick-size?token_id=<asset>`
3. `GET /fee-rate?token_id=<asset>`

### Slippage checks

For copied `BUY`:
- `max_buy_price = leader_price * (1 + max_slippage_bps/10000)`
- Require best ask `<= max_buy_price`.
- Use FAK worst price = `max_buy_price` (tick-quantized).

For copied `SELL`:
- `min_sell_price = leader_price * (1 - max_slippage_bps/10000)`
- Require best bid `>= min_sell_price`.
- Use FAK worst price = `min_sell_price` (tick-quantized).

### Tick quantization

- BUY limits: round **down** to tick.
- SELL limits: round **up** to tick.

If quantized price violates market tick rules, refresh tick size and rebuild once.

## Tick Size, Min Size, and Neg Risk Handling

### Tick size

Tick increments are market-specific. Examples from docs:
- `0.1` -> 1 decimal
- `0.01` -> 2 decimals
- `0.001` -> 3 decimals
- `0.0001` -> 4 decimals

Always fetch dynamically using `/tick-size` (or `tick_size` in `/book`).

### Min order size

Use `min_order_size` from `/book`. Reject/suppress intents below min size.

### Negative risk

Use `neg_risk` from `/book` when creating/sending orders via client options.

## Fees: Exact Handling Policy

This section is mandatory for correct order placement.

### 1. Always fetch fee rate per token

- Call `GET /fee-rate?token_id=<token_id>`.
- Read response `base_fee`.
- Set `order.feeRateBps = str(base_fee)` **before signing**.
- Never hardcode fee values.

### 2. Include `feeRateBps` in signed payload

`feeRateBps` is part of the order signature payload. If it is stale or missing, the signed order can fail validation.

### 3. Fee-enabled vs fee-free markets

- Fee-free markets return `base_fee = 0`.
- Fee-enabled markets return non-zero `base_fee`.

Observed examples during research:
- `base_fee = 0` on one token.
- `base_fee = 1000` on another token.

### 4. Economic fee semantics from docs

Polymarket fee docs specify:
- Fees are computed in USDC.
- Buy-side fees are collected in shares.
- Sell-side fees are collected in USDC.
- Fees are rounded to 4 decimals.
- Minimum charged fee is `0.0001 USDC` (smaller rounds to zero).

### 5. Fee formula usage

Docs publish a market-type formula:
- `fee = C * feeRate * (p * (1 - p))^exponent`

Use this for **estimation/reporting only**. Settlement fee truth is exchange-calculated. Execution correctness depends on passing dynamic `feeRateBps` in the signed order.

### 6. Maker rebate assumption

Default copy strategy is taker-like (`FAK`), so assume rebate = 0 in PnL estimates.

## Balance and Allowance Handling

Before live trading and on balance-related errors:

1. `GET /balance-allowance?asset_type=COLLATERAL&signature_type=<configured>`
2. If stale state suspected, call:
   - `PUT /balance-allowance?...` or
   - `GET /balance-allowance/update?...`

Important:
- These endpoints refresh/read balance/allowance state.
- They do not replace required one-time on-chain approval flows.

## Heartbeat and Session Liveness

Use `POST /v1/heartbeats` every 5 seconds if bot can have open orders.

Rules:
- First call with empty `heartbeat_id`.
- Next calls include returned `heartbeat_id`.
- If server returns 400 invalid heartbeat id, update to returned expected id and retry.

Docs behavior:
- If valid heartbeat is not received within 10 seconds (plus 5-second buffer), open orders can be canceled automatically.

## Reconciliation Loops

### Follower state reconciliation

- Every 10-15s: `GET /positions?user=<follower_proxy_wallet>`
- Every 10-15s: query open orders from CLOB (`GET /orders` filters optional).

### Leader consistency reconciliation

- Every 30s: compare latest leader `TRADE` events in `/activity` vs `/trades`.
- If mismatch window detected, backfill last N minutes and replay idempotently.

## Error Handling Matrix

| Error/Condition | Action |
|---|---|
| `INVALID_ORDER_MIN_TICK_SIZE` | Refresh tick size, requantize price, retry once. |
| `INVALID_ORDER_MIN_SIZE` | Increase size to minimum if within risk caps; else skip. |
| `INVALID_ORDER_DUPLICATED` | Treat as idempotent success and mark processed. |
| `INVALID_ORDER_NOT_ENOUGH_BALANCE` | Refresh balance/allowance, pause execution, alert. |
| `INVALID_ORDER_EXPIRATION` | Sync server time (`/time`), rebuild order expiration, retry once. |
| `INVALID_POST_ONLY_ORDER_TYPE` | Config bug; disable post-only for FOK/FAK and alert. |
| `INVALID_POST_ONLY_ORDER` | Reprice away from crossing or disable post-only path. |
| `FOK_ORDER_NOT_FILLED_ERROR` | If strict mode disabled, downgrade to FAK once; else skip. |
| `ORDER_DELAYED` status | Track delayed state and reconcile fill outcome via trades. |
| `MARKET_NOT_READY` | Requeue with bounded retry and backoff. |
| HTTP 429 / rate-limit | Exponential backoff + token-bucket limiter. |
| HTTP 503 cancel-only/trading-disabled | Enter safe mode; stop new orders; keep cancels/reconcile active. |

## Rate-Limit Aware Scheduling

Use local token buckets per endpoint group.

Documented limits (as of research date):

### CLOB
- Read endpoints: `5000 requests / 10s`
- `POST /order`: `2400 requests / 10s`
- `DELETE /order`: `800 requests / 10s`
- `DELETE /cancel-orders`: `500 requests / 10s`

### Data API
- `/trades`: `200 requests / 10s`
- `/activity`: `200 requests / 10s`
- `/positions`: `100 requests / 10s`

### Bot target usage

- Activity poll: ~10 req/10s
- Trades verification: ~4 req/10s
- Positions: ~1 req/10s
- Orders: typically << 100 req/10s

## System Architecture

```text
Leader Identity Resolver (Gamma /public-profile)
  -> Leader Event Ingestor (Data /activity + /trades)
  -> Normalizer + Dedupe + Checkpoint
  -> Intent Builder (TRADE / MERGE / REDEEM)
  -> Risk Engine
  -> Execution Engine (CLOB /book /tick-size /fee-rate /order)
  -> Reconciliation Engine (Data /positions + CLOB /orders)
  -> Metrics + Alerts + Kill Switch
```

## Persistent Data Model

- `leader_profile(address_input, proxy_wallet, updated_at)`
- `leader_events_raw(id, source, payload, received_at)`
- `leader_events_norm(event_key, type, ts, tx_hash, condition_id, asset, side, size, price, status)`
- `copy_intents(intent_id, event_key, action, planned_notional, order_type, status, reason, created_at)`
- `orders(intent_id, order_id, token_id, side, price, size, fee_rate_bps, status, raw_response, created_at)`
- `fills(order_id, trade_id, fill_price, fill_size, fee_usdc, fee_shares, tx_hash, ts)`
- `positions(token_id, condition_id, outcome, size, avg_price, updated_at)`
- `risk_state(ts, total_exposure, market_exposure, daily_pnl, kill_switch)`

## Operational Controls

- `kill_switch` (manual + automatic).
- `read_only_mode` (ingest and simulate, no order placement).
- `max_retries_per_intent` (default 2).
- `max_pending_intents` queue bound.
- `market_allowlist` and `market_blocklist`.

## Implementation Phases

### Phase 1 - Read-Only Mirror

- Implement profile resolution, ingestion, dedupe, and intent creation.
- No live orders.
- Validate event completeness for 3-7 days.

### Phase 2 - Paper Execution

- Implement full pricing/risk/order-construction path.
- Simulate orders and estimated fees.
- Validate slippage and coverage metrics.

### Phase 3 - Live Small Capital

- Enable live FAK execution with strict caps.
- Enable fee/tick dynamic retrieval and retries.
- Enable alerts and kill switch.

### Phase 4 - Lifecycle and Maker Extensions

- Add optional `MERGE/REDEEM` mirroring.
- Add optional `GTC/GTD` maker mode with heartbeat dependency.

## Success Criteria

- `>= 95%` eligible `TRADE` intents processed without duplicate execution.
- `0` risk-limit breaches.
- `0` malformed-signature order rejects after warmup.
- Fee handling correctness: every submitted order includes dynamic `feeRateBps` from token endpoint.
- p95 execution latency from leader event to follower order submit < configurable target.

## Source Links

- `https://docs.polymarket.com/api-spec/clob-openapi.yaml`
- `https://docs.polymarket.com/api-spec/data-openapi.yaml`
- `https://docs.polymarket.com/api-spec/gamma-openapi.yaml`
- `https://docs.polymarket.com/trading/orders/create`
- `https://docs.polymarket.com/trading/fees`
- `https://docs.polymarket.com/api-reference/authentication`
- `https://docs.polymarket.com/api-reference/rate-limits`
