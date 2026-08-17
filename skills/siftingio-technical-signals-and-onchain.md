---
name: Read technical signals and on-chain positions
description: >-
  Pull RSI/MACD/moving-average signal summaries with the per-indicator votes behind them, plus the
  same signal as a time series, and read on-chain wallet portfolios and DEX pool TVL. Use when
  asked for indicator readings, signal history, a wallet's token balances or pool liquidity.
api: openapi/siftingio-signals-api-openapi.yml
operations:
  - getLiveSignal
  - getSignalHistory
  - getWalletPortfolio
  - getLastTVL
  - getDexBars
generated: '2026-08-17'
method: generated
source: openapi/_original/siftingio-openapi.yaml + https://sifting.io/docs/signals + https://sifting.io/changelog
---

# Technical signals and on-chain data

Base URL `https://api.sifting.io`, header `X-API-Key: <SIFTING_API_KEY>`.

## Technical signals

Two operations, shipped 2026-08-14:

- `getLiveSignal` → `GET /v1/last/signals/{venue}/{symbol}` — the current summarised reading.
- `getSignalHistory` → `GET /v1/hist/{venue}/{symbol}/signals` — the same signal as a time series.

Covers US stocks, forex, crypto and commodities.

### Read the votes, not just the verdict

`LiveSignal` decomposes into `SignalSummary` → `SignalGroup` → `SignalIndicator` → `SignalVote`,
plus a `SignalVerdict` and `SignalCounts`. The verdict is a roll-up of the individual indicator
votes and **the votes are published alongside it**.

Never report the verdict alone. Report which indicators agreed and which dissented — a
"buy" backed by 9 of 10 indicators and a "buy" backed by 6 of 10 are different facts, and the API
hands you the difference. `SignalCounts` gives you the tally directly.

History adds `SignalPoint` → `SignalEvent` (crossover events, e.g. MACD crosses) and
`SignalPrice`. Use `SignalEvent` when asked "when did it cross?" rather than diffing the series
yourself.

### The disclaimer is part of the contract

The provider's own words: *"The output describes indicator state, not investment advice."*
SiftingIO states site-wide that it is not a broker, exchange or investment advisor. Present signal
output as an indicator reading. Do not turn it into a recommendation, a price target or a trade
instruction.

### Errors specific to signals

| Status | Code | Do this |
|---|---|---|
| 422 | `insufficient_history` | Not enough bars to compute the signal for this symbol/interval. Coarsen the interval or pick a symbol with more history. Not retryable as-is. |
| 403 | (raw message) | The venue's market is not in the plan. |
| 404 | `not_found` | Symbol absent from the feed. |

`insufficient_history` is the common one on thin symbols and fine intervals — handle it as a
legitimate answer ("not computable yet"), not as an outage.

## On-chain

- `getWalletPortfolio` → `GET /v1/fnd/dex/wallet/{chain}/{address}` — `WalletPortfolio` with
  `chain`, `address`, `tokens[]` (`WalletToken`), `count`, `updated_at`.
- `getLastTVL` → `GET /v1/last/tvl/{chain}/{pair}` — current aggregated total value locked for a
  DEX pair. Note the shape: **chain + canonical pair** (`WETH-USDC`), not venue + symbol.
- `getDexBars` → `GET /v1/hist/dex/{symbol}/bars` — historical OHLCV for DEX pools. Requires
  `Accept-Encoding: gzip`.

Supported chains are enumerated in the `chain` parameter description — read them from the spec
(`components.parameters.Chain`) rather than assuming. Documented coverage spans Ethereum, Base,
Arbitrum, Optimism, Polygon, BNB Chain, Avalanche and Solana.

Check `updated_at` on a portfolio before quoting a balance, and say when it was last refreshed.
On-chain state moves; a balance without a timestamp is not a useful answer.

## Over MCP, mind the gap

If you are working through the SiftingIO MCP server (`npx -y siftingio-mcp`, 36 tools), be aware
that **signals have no tool** — the server's last release (2026-07-11) predates the signals
endpoints (2026-08-14). `dex_wallet` and `last_tvl` exist; `getSignalHistory`, `getLiveSignal` and
`getDexBars` do not. Use REST or an SDK for those. Full divergence map:
`mcp/siftingio-tool-crosswalk.yml`.

## Notes

- All operations are `GET`; retrying is safe, no idempotency key.
- Signals and TVL are **derived/calculated** values produced by SiftingIO's own methodology, not
  venue-published figures. https://sifting.io/data-methodology
- Rate-limit contract is the same everywhere: `X-RateLimit-Limit`, `X-RateLimit-Remaining`,
  `Retry-After`, `429 rate_limit_exceeded`.
