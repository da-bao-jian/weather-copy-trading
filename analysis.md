# Weather Forecast PM Analysis

## Scope and Objective
This document captures the full analysis we discussed for backtesting a Polymarket weather strategy on **US high-temperature contracts** using historical forecast snapshots.

## Environment and Data
Backtest execution environment:
- Host: `pm-data-harry` (Fly)

Data directories used:
- Markets: `/data/prediction-market-analysis/data/polymarket/markets`
- Trades: `/data/prediction-market-analysis/data/polymarket/trades`
- Blocks: `/data/prediction-market-analysis/data/polymarket/blocks`

Important data quirks validated:
- Sidecar files `._*.parquet` exist and were excluded via strict glob pattern: `*_ [0-9]*.parquet` style selection.
- `trades.timestamp` is not reliable for this backtest timing.
- As-of timing uses `blocks.timestamp` (ISO text, e.g. `2021-01-24T22:29:06Z`) joined by `block_number`.

Schema checks performed:
- `markets`: has `question`, `clob_token_ids`, `outcomes`, `outcome_prices`, `end_date`, `closed`, etc.
- `trades`: has `maker_asset_id`, `taker_asset_id`, `maker_amount`, `taker_amount`, `block_number`, etc.
- `blocks`: has `block_number`, `timestamp`.

## Universe and Time Frame
Universe definition:
- Closed markets only: `closed = true`
- Market type: `highest temperature in` / `high temperature in`
- Fahrenheit contracts only (`°F`)
- Strict contract forms:
  - `between X-Y°F` (or `between X and Y°F`)
  - `X°F or below`
  - `X°F or higher` / `X°F or above`
- Start date filter: `end_date >= 2025-09-01`
- US city filter via Open-Meteo geocoding

Parsed coverage stats:
- `raw_rows=5123`
- `parsed_strict=3310`
- `parse_fail=1811`
- `outcome_fail=2`
- `us_contracts=2603`
- `unique_us_cities=9`

Note on outcomes:
- High-temp markets use outcomes `[
"Yes","No"]`.
- 2 contracts were excluded as ambiguous (non-binary terminal prices around 50/50):
  - Atlanta Dec 17 (between 52-53°F)
  - Atlanta Dec 17 (between 54-55°F)

## Forecast Source (NOAA Proxy)
Forecast signal source:
- Open-Meteo `previous-runs-api`
- Endpoint: `https://previous-runs-api.open-meteo.com/v1/forecast`
- Model: `gfs_seamless` (NOAA GFS proxy)
- Variables:
  - `temperature_2m_previous_day1` (for 24h horizon)
  - `temperature_2m_previous_day2` (for 48h horizon)

Construction:
- For each contract target date, compute daily max of hourly series:
  - day-1 max -> 24h signal
  - day-2 max -> 48h signal

## Price Construction and Entry Timing
Entry snapshot definitions:
- 24h strategy: snapshot at `end_date - 24h`
- 48h strategy: snapshot at `end_date - 48h`

As-of price logic:
- YES price from trades:
  - if `maker_asset_id='0'`: `yes_price = maker_amount / taker_amount`
  - else: `yes_price = taker_amount / maker_amount`
- Use last trade at or before snapshot time (`block_ts <= snapshot_ts`), ordered by `block_ts desc, block_number desc`.

Coverage of tradable snapshots:
- `snapshots=5206` (2603 contracts * 2 horizons)
- `with_price=2468`
- `missing_price=2738`
- final evaluated rows:
  - `eval24=1917`
  - `eval48=551`

## Trade Triggering Condition (This Backtest)
This run is **forecast-following**, not disagreement-only:
- Convert forecast max temp to YES/NO based on contract text.
- If forecast implies YES -> buy YES.
- Else -> buy NO.

Cost and payout:
- YES buy cost = `p_yes`
- NO buy cost = `1 - p_yes`
- Settlement payout = `1` if chosen side wins, else `0`
- `pnl = payout - cost`
- `roi = sum(pnl) / sum(cost)`

## Metrics Definitions
- `n`: number of trades in that row/group.
- `hit_rate`: percent of trades where predicted side matched final resolved outcome.
- `edge` (used for threshold filtering):
  - if buy YES: `1 - p_yes`
  - if buy NO: `p_yes`
- Threshold table rows are filtered by `edge >= threshold`.

## Main Results: Threshold Tables
### 24h Backtest (Day-1 Forecast)
| threshold | n | hit_rate | roi | avg_edge |
|---:|---:|---:|---:|---:|
| 0.00 | 1917 | 0.8445 | 0.0033 | 0.1582 |
| 0.02 | 1182 | 0.7530 | 0.0078 | 0.2529 |
| 0.05 | 922 | 0.6985 | 0.0215 | 0.3162 |
| 0.10 | 721 | 0.6408 | 0.0427 | 0.3855 |
| 0.15 | 615 | 0.6049 | 0.0644 | 0.4317 |
| 0.20 | 537 | 0.5698 | 0.0753 | 0.4701 |
| 0.25 | 463 | 0.5356 | 0.0951 | 0.5109 |
| 0.30 | 390 | 0.4897 | 0.1037 | 0.5563 |

### 48h Backtest (Day-2 Forecast)
| threshold | n | hit_rate | roi | avg_edge |
|---:|---:|---:|---:|---:|
| 0.00 | 551 | 0.9365 | 0.0285 | 0.0895 |
| 0.02 | 281 | 0.8754 | 0.0501 | 0.1663 |
| 0.05 | 150 | 0.7800 | 0.0981 | 0.2897 |
| 0.10 | 108 | 0.7222 | 0.1576 | 0.3761 |
| 0.15 | 86 | 0.6744 | 0.2136 | 0.4443 |
| 0.20 | 76 | 0.6447 | 0.2397 | 0.4799 |
| 0.25 | 70 | 0.6286 | 0.2635 | 0.5025 |
| 0.30 | 62 | 0.6290 | 0.3460 | 0.5327 |

## City-Level Performance (Totals)
### 24h
| city | n | hit_rate | pnl | roi |
|---|---:|---:|---:|---:|
| Atlanta | 296 | 0.8142 | -9.8251 | -0.0392 |
| Chicago | 35 | 0.8571 | 0.1933 | 0.0065 |
| Dallas | 300 | 0.8567 | -1.5856 | -0.0061 |
| Miami | 38 | 0.7895 | 0.2190 | 0.0074 |
| New York City | 970 | 0.8515 | 15.7955 | 0.0195 |
| Seattle | 278 | 0.8453 | 0.5499 | 0.0023 |

### 48h
| city | n | hit_rate | pnl | roi |
|---|---:|---:|---:|---:|
| Atlanta | 104 | 0.9615 | 0.9015 | 0.0091 |
| Chicago | 9 | 1.0000 | 0.1020 | 0.0115 |
| Dallas | 80 | 0.9875 | 2.9977 | 0.0394 |
| Miami | 10 | 1.0000 | 0.1400 | 0.0142 |
| New York City | 287 | 0.8990 | 10.0036 | 0.0403 |
| Seattle | 61 | 0.9836 | 0.2500 | 0.0042 |

## City x Contract Type
### 24h highlights
- `New York City|between`: `n=678`, `pnl=13.6130`, `roi=0.0255`
- `Atlanta|or_higher`: `n=46`, `pnl=-5.7215`, `roi=-0.1804`
- `Seattle|or_higher`: `n=43`, `pnl=2.4332`, `roi=0.0600`

### 48h highlights
- `Dallas|between`: `n=37`, `pnl=2.7707`, `roi=0.0809`
- `New York City|between`: `n=174`, `pnl=8.3408`, `roi=0.0593`
- `Seattle|between`: `n=27`, `pnl=-0.3300`, `roi=-0.0125`

## City PnL/ROI Over Time (Monthly)
(Period grouped by `target_date` month)

### 24h
- `Atlanta|2025-12`: `n=111`, `pnl=-2.6000`, `roi=-0.0264`
- `Atlanta|2026-01`: `n=185`, `pnl=-7.2251`, `roi=-0.0475`
- `Chicago|2026-01`: `n=35`, `pnl=0.1933`, `roi=0.0065`
- `Dallas|2025-12`: `n=118`, `pnl=-1.7835`, `roi=-0.0172`
- `Dallas|2026-01`: `n=182`, `pnl=0.1979`, `roi=0.0013`
- `Miami|2026-01`: `n=38`, `pnl=0.2190`, `roi=0.0074`
- `New York City|2025-09`: `n=182`, `pnl=1.6260`, `roi=0.0107`
- `New York City|2025-10`: `n=196`, `pnl=7.7177`, `roi=0.0456`
- `New York City|2025-11`: `n=208`, `pnl=-2.0730`, `roi=-0.0124`
- `New York City|2025-12`: `n=196`, `pnl=0.2905`, `roi=0.0018`
- `New York City|2026-01`: `n=188`, `pnl=8.2343`, `roi=0.0519`
- `Seattle|2025-12`: `n=107`, `pnl=0.8578`, `roi=0.0092`
- `Seattle|2026-01`: `n=171`, `pnl=-0.3079`, `roi=-0.0022`

### 48h
- `Atlanta|2025-12`: `n=21`, `pnl=0.2400`, `roi=0.0116`
- `Atlanta|2026-01`: `n=83`, `pnl=0.6615`, `roi=0.0084`
- `Chicago|2026-01`: `n=9`, `pnl=0.1020`, `roi=0.0115`
- `Dallas|2025-12`: `n=25`, `pnl=0.9600`, `roi=0.0399`
- `Dallas|2026-01`: `n=55`, `pnl=2.0377`, `roi=0.0392`
- `Miami|2026-01`: `n=10`, `pnl=0.1400`, `roi=0.0142`
- `New York City|2025-09`: `n=51`, `pnl=2.1097`, `roi=0.0481`
- `New York City|2025-10`: `n=43`, `pnl=1.8094`, `roi=0.0500`
- `New York City|2025-11`: `n=56`, `pnl=2.5480`, `roi=0.0561`
- `New York City|2025-12`: `n=67`, `pnl=0.8400`, `roi=0.0144`
- `New York City|2026-01`: `n=70`, `pnl=2.6966`, `roi=0.0419`
- `Seattle|2025-12`: `n=14`, `pnl=0.2200`, `roi=0.0160`
- `Seattle|2026-01`: `n=47`, `pnl=0.0300`, `roi=0.0007`

## “Best Strategy So Far” (from this analysis set)
By ROI (high selectivity):
- `48h + edge >= 0.30`
- `n=62`, `hit_rate=0.6290`, `roi≈0.3460` (threshold-table run)

Higher-volume alternatives:
- `24h + edge >= 0.15`: `n=615`, `roi≈0.0644`
- `24h + edge >= 0.25`: `n=463`, `roi≈0.0951`

## Capital Interpretation and Starting Bankroll
Clarification:
- `n * $100` is **turnover** (total capital deployed over time), not starting bankroll.
- Starting bankroll depends on overlap of open positions:
  - `starting capital ≈ $100 * peak concurrent positions`

Peak overlap estimates (separate capital requirement run):
- `48h, edge>=0.30`: peak concurrent `5` -> about `$500` starting capital
- `24h, edge>=0.15`: peak concurrent `16` -> about `$1,600`
- `24h, edge>=0.25`: peak concurrent `14` -> about `$1,400`

Turnover examples at `$100/signal`:
- `48h, edge>=0.30`: `62 * $100 = $6,200`
- `24h, edge>=0.15`: `~615 * $100 = ~$61,500`
- `24h, edge>=0.25`: `~463 * $100 = ~$46,300`

## Caveats
- This is a **forecast-following** strategy (buy forecast-implied side), not strict disagreement-only.
- Many missing as-of prices, especially 48h (`missing 2052` at 48h snapshots), can bias 48h results.
- No explicit fees/slippage/partial-fill model beyond observed trade price snapshots.
- In-sample historical backtest; forward performance may differ.

## Suggested Next Variant
Run strict disagreement-only trigger:
- trade only if forecast side contradicts market-implied side (`p_yes < 0.5` for YES call, `p_yes > 0.5` for NO call),
- then compare ROI/trade count/capital requirement directly versus forecast-following.
