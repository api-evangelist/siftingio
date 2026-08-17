---
name: Pull live prices and historical OHLCV bars across asset classes
description: >-
  Get a live trade, quote or previous close for any venue/symbol, and backfill historical OHLCV
  bars for stocks, forex, crypto, commodities or DEX pools — all through one Bar shape. Use when
  asked for a current price, a price history, a chart series or a backtest dataset.
api: openapi/siftingio-live-api-openapi.yml
operations:
  - getLastTrade
  - getLastQuote
  - getLastClose
  - getSnapshot
  - convertAmount
  - getStockBars
  - getForexBars
  - getCryptoBars
  - getCommodityBars
  - getDexBars
generated: '2026-08-17'
method: generated
source: openapi/_original/siftingio-openapi.yaml + https://sifting.io/docs/quickstart + https://sifting.io/docs/errors
---

# Live prices and historical bars

Base URL `https://api.sifting.io`, header `X-API-Key: <SIFTING_API_KEY>`.

## Pick the right call

| Need | Operation | Path |
|---|---|---|
| Last traded price + size | `getLastTrade` | `/v1/last/trade/{venue}/{symbol}` |
| Best bid/ask + sizes | `getLastQuote` | `/v1/last/quote/{venue}/{symbol}` |
| Prior trading day's close | `getLastClose` | `/v1/last/close/{venue}/{symbol}` |
| Whole venue at once | `getSnapshot` | `/v1/snapshot/{venue}` |
| Convert an amount | `convertAmount` | `/v1/convert/{from}/{to}` |
| DEX pool TVL | `getLastTVL` | `/v1/last/tvl/{chain}/{pair}` |

Do not call `getLastTrade` when you want the spread — a last trade is not a quote. Do not derive
"yesterday's close" from bars when `getLastClose` exists; it is market-calendar-aware.

## Historical bars — one shape, five asset classes

| Asset class | Operation | Path | Key param |
|---|---|---|---|
| US stocks | `getStockBars` | `/v1/hist/stocks/{ticker}/bars` | `ticker` |
| Forex | `getForexBars` | `/v1/hist/forex/{pair}/bars` | `pair` (EURUSD) |
| Crypto | `getCryptoBars` | `/v1/hist/crypto/{symbol}/bars` | `symbol` (BTCUSD) |
| Commodities | `getCommodityBars` | `/v1/hist/commodities/{symbol}/bars` | `symbol` (XAUUSD, WTIUSD, NATGAS) |
| DEX pools | `getDexBars` | `/v1/hist/dex/{symbol}/bars` | `symbol` |

Every one returns the same `Bar`: `{t, o, h, l, c, v}` — `t` is **int64 Unix epoch
milliseconds**, UTC. That single shape across all five classes is the point of the API; write one
parser and reuse it.

Parameters: `start`, `end`, `interval`, `limit`, `cursor`, and on **commodities and forex only**
an `order` param (`asc` default / `desc`).

> Use `order=desc` when you want the most recent N bars. It anchors the first page at the newest
> bar, so it costs one request instead of paging the whole window. Every other historical route is
> ascending-only — check before assuming.

## Gzip is mandatory on the heavy routes

`getStockBars`, `getForexBars`, `getCryptoBars`, `getCommodityBars`, `getDexBars` and
`getSnapshot` all declare `Accept-Encoding: gzip` as a **required** header and return
`406 gzip_required` without it. Set it once on your client.

## Paginate a backfill

```
GET /v1/hist/crypto/BTCUSD/bars?start=2026-01-01&end=2026-06-30&interval=1m&limit=200
  -H "X-API-Key: $KEY" -H "Accept-Encoding: gzip"
```

Then loop on `meta.next_cursor`, stopping when it is null or absent. `meta` also carries `as_of`,
`symbol` and `interval` — confirm `interval` matches what you asked for before resampling.

## Historical depth is a plan limit, not an error

Free = 1 month, Builder = 1 year, Pro/Ultra/Enterprise = full. A `start` older than your tier's
depth is a plan boundary, not a bug. `403` (raw message) means the key is valid but the
subscription does not include that market — entitlement is **per market**, so a Crypto key does
not read Commodities.

## Errors

| Status | Code | Do this |
|---|---|---|
| 400 | (raw message) | Bad interval, bad cursor, bad limit, missing param. Fix and retry. |
| 401 | `unauthorized` | Key problem. Do not retry. |
| 403 | (raw message) | Market not in the plan. Do not retry. |
| 404 | `not_found` | Symbol absent from the live feed. Check `/v1/fnd/markets` and the symbol catalog. |
| 406 | `gzip_required` | Add the header, retry once. |
| 422 | `insufficient_history` | Not enough bars for this symbol/interval. Widen the window or coarsen the interval. |
| 429 | `rate_limit_exceeded` | Sleep `Retry-After`. |
| 503 | `stale_snapshot` | The live snapshot is older than the freshness threshold (default 5s). The body carries `last_t` and `server_now` — decide whether the staleness matters, then retry. |

`stale_snapshot` is a feature, not a fault: the API refuses to serve a price it considers stale
rather than silently returning one. Surface the age to the user; don't swallow it.

## Streaming instead of polling

If you are polling `getLastTrade` in a loop, stop and use the WebSocket instead:
`wss://stream.sifting.io/ws/v1?key=<KEY>` (query param only — browsers cannot set headers on the
WS handshake). Send `subscribe` frames, receive `tick` / `tvl` frames. Contract:
`asyncapi/siftingio-asyncapi.yaml`. Connection and subscription counts are capped per tier.

## Notes

- All operations are `GET` — retry freely, no idempotency key needed.
- Prices are aggregated, normalised, derived reference values, explicitly **not**
  exchange-of-record and not raw venue feeds. https://sifting.io/data-methodology
- There is no symbol-listing operation. The catalog is a web page
  (https://sifting.io/symbols); `searchStocks` only covers US equities.
