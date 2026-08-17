<!-- TOOL-HEADER:START -->
# SiftingIO API agent rules (AGENTS.md)

These instructions teach AI coding agents (OpenAI Codex, Windsurf, Gemini CLI, and any tool that reads `AGENTS.md`) how to use the **SiftingIO** market-data API correctly. Place this file at your project root. The same ruleset is mirrored across `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.windsurfrules`, and `.github/copilot-instructions.md`; Cursor uses the split files in `.cursor/rules/`.
<!-- TOOL-HEADER:END -->

<!-- BODY:START -->

SiftingIO is a **unified, multi-asset market-data API**. One key gives you US equities, forex, crypto, commodities, and on-chain DEX data across two surfaces:

- **REST** at `https://api.sifting.io`: fundamentals (SEC filings, XBRL, holdings), historical OHLCV bars, and live snapshots.
- **WebSocket** at `wss://stream.sifting.io/ws/v1`: live ticks and TVL deltas.

It is a **data provider**, not an exchange, broker, or investment advisor. The published price is a robust *reference* fair price (see the methodology section).

## Official SDKs & references

SiftingIO ships official SDKs for **Go, Python, and JavaScript/TypeScript** (hub: `https://sifting.io/sdks`, GitHub org: `https://github.com/SiftingIO`). These rules document the underlying REST/WebSocket contract every SDK wraps. Authoritative specs: OpenAPI 3.1 (`https://sifting.io/openapi.yaml`), AsyncAPI 3.0 (`https://sifting.io/asyncapi.yaml`), the LLM context index (`https://sifting.io/llms-full.txt`), and docs at `https://sifting.io/docs`.

---

## 1. Core: auth, conventions, errors

### Base URLs & versioning

- REST host: `https://api.sifting.io`
- WebSocket host: `wss://stream.sifting.io/ws/v1`
- Every REST path is pinned to **`/v1`** (e.g. `/v1/fnd/stocks/search`). Fields are additive within a version; breaking changes ship under a new version.

### Authentication

Keys look like `sft_...`.

- **REST (preferred):** header `X-API-Key: sft_...`
- **REST (fallback):** query param `?api_key=sft_...` (avoid; it can leak into logs)
- **WebSocket:** `?key=sft_...` in the URL, or send `{ "op": "auth", "key": "sft_..." }` as the first frame.

Verify a key:

```bash
# curl
curl -H "X-API-Key: $KEY" \
  "https://api.sifting.io/v1/fnd/stocks/search?q=apple&limit=5"
```

```python
# Python, requests
import os, requests
BASE = "https://api.sifting.io"
s = requests.Session()
s.headers["X-API-Key"] = os.environ["SIFTING_API_KEY"]
print(s.get(f"{BASE}/v1/fnd/stocks/search", params={"q": "apple", "limit": 5}).json()["data"])
```

```typescript
// JavaScript / TypeScript, fetch (Node 18+ or browser)
const BASE = "https://api.sifting.io";
const res = await fetch(`${BASE}/v1/fnd/stocks/search?q=apple&limit=5`, {
  headers: { "X-API-Key": process.env.SIFTING_API_KEY! },
});
if (!res.ok) throw new Error(`HTTP ${res.status}`);
console.log((await res.json()).data);
```

### Conventions

- **Identifiers**: Tickers are **case-insensitive**. CIKs are **10-digit zero-padded strings** (`"0000320193"`), never strip the zeros. Accessions are dashed (`"0000320193-25-000089"`), accepted dashed or undashed.
- **Dates & timestamps**: ISO 8601. `filed_at`/`period_end`/`transaction_date` are date-only (`YYYY-MM-DD`); `accepted_at`/`as_of` are full (`YYYY-MM-DDTHH:MM:SSZ`). Tick & bar `t` is **int64 Unix epoch milliseconds**.
- **Money & units**: `{ value, unit }` where `unit` is `USD`, `USD/shares`, `shares`, or `pure`. Don't assume USD.
- **Pagination**: `?limit` (default 50, max 200; default 10, max 25 for `/insiders`) + `?cursor` (opaque, from `meta.next_cursor`). Null/omitted cursor = last page. Round-trip cursors verbatim.
- **Envelope**: `{ "data": [...], "meta": {...} }` for lists/series; a single resource may return the object directly.

### Compression (gzip)

Send `Accept-Encoding: gzip`. All `/fnd/*` and `/hist/*` come back gzipped. **Heavy endpoints require it** (full XBRL bundles, single-concept, screener, and historical bars) → **`406 gzip_required`** without it. With `curl`: `-H "Accept-Encoding: gzip" --compressed`.

### Errors

Shape: `{ "error": "error_code", "message": "..." }`. **Switch on the `error` code (snake_case, stable), not the HTTP status.**

| HTTP | `error` code | Meaning |
| --- | --- | --- |
| 400 | *(raw message)* | Malformed query, invalid cursor, bad limit, missing required parameter |
| 401 | `unauthorized` | API key missing or invalid |
| 403 | *(raw message)* | Authenticated but not entitled to this product/market |
| 404 | `unknown_ticker` | `:ticker` not in the SEC US ticker registry |
| 404 | `unknown_filer` | `:filer` (ticker or CIK) not found |
| 404 | `filing_not_found` | Accession not in this filer's recent-filings window |
| 404 | `no_13f_filings` | Filer has never filed 13F-HR |
| 404 | `not_found` | Concept/period/unit has no data, or symbol absent from live feed |
| 404 | `section_not_found` | Filing section couldn't be extracted (body includes `available`) |
| 404 | `insufficient_filings` | Risk-factors diff needs ≥ 2 10-Ks |
| 400 | `invalid_section` | `:section` not allowed (body includes `valid_options`) |
| 400 | `invalid_accession_format` | Accession didn't match the dashed/undashed shape |
| 400 | `invalid_date_range` | `to` must be after `from`; range capped at 365 days |
| 406 | `gzip_required` | Heavy endpoint called without `Accept-Encoding: gzip` |
| 422 | `risk_factors_unavailable` | Item 1A couldn't be extracted from one or both 10-Ks |
| 429 | `rate_limit_exceeded` | Per-tier budget exhausted, inspect `Retry-After` |
| 502 | `upstream_error` / `malformed_upstream` | Filings source failed or returned an invalid payload |
| 503 | `stale_snapshot` | Snapshot older than threshold (body carries `last_t`, `server_now`) |
| 503 | `upstream_rate_limited` | Filings source throttled our pipeline, retry shortly |

### Rate limits

`X-RateLimit-Limit` (burst capacity), `X-RateLimit-Remaining` (tokens left), `Retry-After` (seconds, on 429). On `429`, back off for `Retry-After` seconds.

---

## 2. Fundamentals (`/v1/fnd/*`)

US-equity reference data from the SEC. **Glossary:** CIK (10-digit zero-padded), accession (dashed filing id), XBRL (CamelCase concept tags under `us-gaap`/`dei`/`ifrs-full`/`srt`), forms `10-K`/`10-Q`/`8-K`/`DEF 14A`/`SC 13D`/`SC 13G`/`13F-HR`/`3`/`4`/`5`.

### Discovery

- `GET /v1/fnd/stocks/search`: `q` (required), `limit` (default 25, max 100).
- `GET /v1/fnd/stocks/:ticker/profile`: name, exchanges, SIC, fiscal year end, CIK.

### SEC filings

- `GET /v1/fnd/stocks/:ticker/filings`: filter `form`, `from`/`to`, paginate `limit`/`cursor`.
- `GET /v1/fnd/stocks/:ticker/filings/:accession`: single filing.
- `GET /v1/fnd/stocks/:ticker/events`: 8-K events.
- `GET /v1/fnd/stocks/:ticker/ownership`: SC 13D/13G.
- `GET /v1/fnd/stocks/:ticker/compensation`: exec comp.
- `GET /v1/fnd/stocks/:ticker/earnings`: earnings history.

### Filing text & analysis

- `GET /v1/fnd/stocks/:ticker/filings/:accession/sections`: list sections.
- `GET /v1/fnd/stocks/:ticker/filings/:accession/sections/:section`: one section.
- `GET /v1/fnd/stocks/:ticker/risk-factors-diff`: Item 1A YoY diff.

### Financials (XBRL), gzip required

- `GET /v1/fnd/stocks/:ticker/financials`: full bundle.
- `GET /v1/fnd/stocks/:ticker/financials/:concept`: one concept (`taxonomy` query, `us-gaap` default).
- `GET /v1/fnd/stocks/screener/:concept/:period`: cross-sectional screener.
- `GET /v1/fnd/stocks/:ticker/ratios`: fundamental ratios.

### Insiders & holdings

- `GET /v1/fnd/stocks/:ticker/insiders`: Form 3/4/5 (`limit` default 10, max 25).
- `GET /v1/fnd/filers/:filer/holdings`: 13F-HR (`:filer` = ticker or CIK).

### Calendars

- `GET /v1/fnd/economic-calendar`: `to` after `from`, ≤ 365 days.
- `GET /v1/fnd/markets`, `/v1/fnd/markets/status`, `/v1/fnd/markets/:market/status`, `/v1/fnd/markets/:market/hours`, `/v1/fnd/markets/:market/calendar`.

### DEX wallets

- `GET /v1/fnd/dex/wallet/:chain/:address`: on-chain portfolio.

Single-concept response:

```json
{ "ticker": "AAPL", "cik": "0000320193", "taxonomy": "us-gaap", "concept": "Revenues",
  "series": [ { "value": 94836000000, "unit": "USD", "period_end": "2024-03-30",
    "fiscal_year": 2024, "fiscal_period": "Q2", "form": "10-Q",
    "accession": "0000320193-24-000089", "filed_at": "2024-05-02" } ] }
```

Gotchas: financials are gzip-required; concepts are CamelCase XBRL tags; read the `unit`; `unknown_ticker` ≠ `unknown_filer`.

---

## 3. Historical OHLCV (`/v1/hist/*`), gzip required

Uniform `{ data, meta }` envelope. **All bar endpoints require `Accept-Encoding: gzip`.**

- `GET /v1/hist/stocks/:ticker/bars` (regular US hours 09:30-16:00 ET)
- `GET /v1/hist/forex/:pair/bars`
- `GET /v1/hist/crypto/:symbol/bars`
- `GET /v1/hist/dex/:symbol/bars` (chain-prefixed symbol, e.g. `eth:WETH-USDC`)

Params: `start`/`end` (`YYYY-MM-DD` in US Eastern, or RFC3339 UTC), `interval` (`1m,5m,15m,30m,1h,1d,1w,1mo`, default `1m`), `limit` (default 1000, max 2000), `cursor`. Bar fields: `t` (ms), `o`/`h`/`l`/`c`, `v`. Empty window → `200` with `data: []`; unknown symbol → `404 not_found`.

```python
# Python, paginate via meta.next_cursor
def bars(s, base, ticker, **params):
    cursor = None
    while True:
        if cursor: params["cursor"] = cursor
        body = s.get(f"{base}/v1/hist/stocks/{ticker}/bars", params=params).json()
        yield from body["data"]
        cursor = body["meta"].get("next_cursor")
        if not cursor: break
```

```typescript
// JavaScript / TypeScript
async function* bars(ticker: string, params: Record<string,string>) {
  let cursor: string | undefined;
  do {
    const qs = new URLSearchParams({ ...params, ...(cursor ? { cursor } : {}) });
    const res = await fetch(`https://api.sifting.io/v1/hist/stocks/${ticker}/bars?${qs}`,
      { headers: { "X-API-Key": process.env.SIFTING_API_KEY!, "Accept-Encoding": "gzip" } });
    const body = await res.json();
    yield* body.data;
    cursor = body.meta.next_cursor ?? undefined;
  } while (cursor);
}
```

Gotchas: always send the gzip header; `t` is ms (`new Date(t)` in JS, `t/1000` in Python); date-only is US Eastern for stocks; stop when `next_cursor` is null.

---

## 4. Live market data

Symbols: **6-12 uppercase alphanumeric, no separators** (`BTCUSD`, `EURUSD`). DEX/TVL are chain-prefixed (`eth:WETH-USDC`).

### REST live

- `GET /v1/last/trade/:venue/:symbol`: `:venue` ∈ `stocks|crypto|forex|commodities|dex`.
- `GET /v1/last/quote/:venue/:symbol`
- `GET /v1/snapshot/:venue`: **gzip required**, `:venue` ∈ `crypto|forex|stocks|dex`, optional `symbols` CSV (≤ 250); omit for full market.
- `GET /v1/last/tvl/:chain/:pair`
- `GET /v1/convert/:from/:to`

Snapshot entry fields: `s`, `p`, `P`, `b`/`B`, `a`/`A`, `t` (ms). For forex/stocks `P`/`B`/`A` are `0` (expected). Stale → `503 stale_snapshot`.

### WebSocket

One connection, many subscriptions. Connect `wss://stream.sifting.io/ws/v1?key=sft_•••`. **Branch on the `f` discriminator** in every server frame.

Ops: `auth`, `subscribe`, `unsubscribe`, `ping`.

```json
{ "op": "subscribe", "product": "cex", "symbols": ["BTCUSD", "ETHUSD"] }
```

Products: `cex` · `dex` · `fx` · `us` · `com` · `tvl`.

Server frames (`f`): `ack`, `pong`, `tick`, `tvl`, `error`. Tick:

```json
{ "f": "tick", "class": "fx", "s": "EURUSD", "p": 1.16934, "P": 85790,
  "b": 1.16925, "B": 510300, "a": 1.16943, "A": 585025, "t": 1778019852426 }
```

On subscribe, the server emits **one snapshot frame per channel before live**. Idle timeout: no client frame for **90s** → close `1000` "idle timeout"; ping every ≤ 60s (**inbound traffic doesn't reset the idle timer**). WS error codes: `bad_op`, `auth_required`, `auth_timeout`, `auth_failed`, `unknown_product`, `max_connections`, `max_subscriptions`, `internal`.

```python
# Python, websockets
import asyncio, json, os, websockets
async def main():
    url = f"wss://stream.sifting.io/ws/v1?key={os.environ['SIFTING_API_KEY']}"
    async with websockets.connect(url) as ws:
        await ws.recv()  # ack op:auth
        await ws.send(json.dumps({"op": "subscribe", "product": "cex", "symbols": ["BTCUSD"]}))
        async for raw in ws:
            m = json.loads(raw)
            if m["f"] == "tick": print(m["s"], m["p"], m["t"])
            elif m["f"] == "error": print("error:", m["code"])
asyncio.run(main())
```

```typescript
// JavaScript / TypeScript, ws (Node) or browser WebSocket
const ws = new WebSocket(`wss://stream.sifting.io/ws/v1?key=${process.env.SIFTING_API_KEY}`);
ws.onopen = () => ws.send(JSON.stringify({ op: "subscribe", product: "cex", symbols: ["BTCUSD"] }));
ws.onmessage = (e) => { const m = JSON.parse(e.data);
  if (m.f === "tick") console.log(m.s, m.p, m.t); else if (m.f === "error") console.error(m.code); };
setInterval(() => ws.send(JSON.stringify({ op: "ping" })), 30_000);
```

Gotchas: symbols have no separators; `/v1/snapshot/:venue` is gzip-required; branch on `f` (and on `code` for errors); ping ≤ 60s even while receiving ticks.

---

## 5. Fair Price methodology

For every symbol SiftingIO publishes **one fair price aggregated from multiple independent venues**, built on robust statistics so a stale, frozen, or manipulated venue can't move it. **Treat it as a robust reference price**, gate on its quality flag, and don't assume it's an executable, sub-millisecond quote.

**Principles:** weighted **median** (50% breakdown point, not an average); never gate publishing (fails soft with a degraded flag); independent venues; transparent & version-tagged.

**Pipeline:** VALIDATE (drop stale; hard-kill prices far from the median) → SCORE (MAD dispersion; modified z-score, Iglewicz & Hoaglin 1993) → REMEMBER (per-venue EWMA reputation; quarantine with hysteresis; frozen-feed detection) → AGGREGATE (`volume × reputation` weighted median; composite spread that can't cross and widens on divergence).

**On the wire:** three timestamps (source/ingest/publish), a normalised cross-venue consensus value, and a quality flag (`Normal` at quorum / `Degraded` on fallback, reason exposed). Read the flag.

**Delivery cadence by plan:** Free 1 Hz · Builder 4 Hz · Pro 6 Hz · Ultra 10 Hz · Enterprise near-real-time, uncapped. Price is computed identically for everyone; only push frequency differs.

**Limits (state them honestly):** robust only to a *minority* of bad feeds (no estimator survives coordinated majority error); a reference price, **not** a sub-ms execution feed; the composite spread is representative, **not** a claim of executable depth.

**Building on it:** treat the price as a consensus mark; gate on the quality flag/consensus for settlement, marking, and risk limits; size your plan tier to the Hz you need; use the three timestamps to monitor latency.

## 6. Official SDKs

Official, MIT-licensed SDKs wrap the same REST + WebSocket API in idiomatic clients with a shared, resource-namespaced shape (`client.last.trade(...)`, `client.stocks.profile(...)`, `client.crypto.bars(...)`). Prefer them over hand-rolled HTTP. Repos: <https://github.com/SiftingIO> · hub: <https://sifting.io/sdks>. There's also an official MCP server (`SiftingIO/siftingio-mcp`).

### Python: `siftingio` (PyPI, Python 3.9+)

```bash
pip install siftingio
```

```python
from siftingio import SiftingClient, AsyncSiftingClient, auto_paginate

client = SiftingClient(api_key="sft_...")            # or get_api_key=lambda: read_secret(...)

client.last.trade("crypto", "BTCUSD")                # last trade / quote
client.stocks.profile("AAPL"); client.stocks.ratios("AAPL")
client.crypto.bars("BTCUSD", start="2024-01-01", interval="1h")   # forex/stocks/dex parallel

for filing in auto_paginate(lambda cursor: client.stocks.filings("AAPL", cursor=cursor, form="10-K")):
    print(filing)

# WebSocket (async); aauto_paginate is the async paginator
async with AsyncSiftingClient(api_key="sft_...") as c:
    async with c.ws() as socket:
        socket.on("tick", lambda t: print(t["s"], t.get("p")))
        await socket.subscribe("cex", ["BTCUSD", "ETHUSD"])
        async for frame in socket:
            ...
```

Sync WebSocket: `socket = client.ws(); socket.connect(); socket.subscribe("cex", ["BTCUSD"]); for frame in socket.stream(): ...; socket.close()`.

### JavaScript / TypeScript: `@siftingio/sdk` (npm, Node 18+, isomorphic)

```bash
npm install @siftingio/sdk    # add `ws` for WebSocket on Node
```

```typescript
import { SiftingClient, autoPaginate, collectAll } from "@siftingio/sdk";

const sifting = new SiftingClient({ apiKey: process.env.SIFTING_API_KEY });

await sifting.last.trade("crypto", "BTCUSD");
await sifting.stocks.profile("AAPL");
const { data: bars } = await sifting.crypto.bars("BTCUSD", { start: "2024-01-01", interval: "1h" });

for await (const filing of autoPaginate((cursor) => sifting.stocks.filings("AAPL", { cursor, form: "10-K" })))
  console.log(filing.accession);

const socket = sifting.ws();
socket.on("tick", (t) => console.log(t.s, t.p ?? `${t.b}/${t.a}`));
await socket.connect();
socket.subscribe("cex", ["BTCUSD", "ETHUSD"]);
```

Runs on Node 18+, Bun, Deno, edge runtimes, and browsers. Never embed a production key in the browser; proxy through your backend.

### Go: `github.com/siftingio/sdk-go` (Go 1.23+)

```bash
go get github.com/siftingio/sdk-go@latest
```

```go
import siftingio "github.com/siftingio/sdk-go"

client := siftingio.New(siftingio.WithAPIKey(os.Getenv("SIFTING_API_KEY")))

trade, err := client.Last.Trade(ctx, "crypto", "BTCUSD")
profile, err := client.Stocks.Profile(ctx, "AAPL")
bars, err := client.Crypto.Bars(ctx, "BTCUSD", &siftingio.BarsParams{Start: "2024-01-01", Interval: "1h"})

// pagination: siftingio.AutoPaginate(...) (Go 1.23 range-over-func) or siftingio.CollectAll(...)

sock := client.WS()
sock.OnTick(func(t *siftingio.Tick) { /* handle */ })
sock.Connect(ctx)
sock.Subscribe(ctx, siftingio.ProductCEX, "BTCUSD", "ETHUSD")  // replayed automatically after reconnects
defer sock.Close()
```

### Cross-SDK conventions

- API-key env var is `SIFTING_API_KEY` (Python also accepts a `get_api_key` callback).
- Pagination helpers exist everywhere: `auto_paginate`/`aauto_paginate` (Python), `autoPaginate`/`collectAll` (TS), `AutoPaginate`/`CollectAll` (Go). Pass a `cursor -> call(..., cursor)` closure and they follow `meta.next_cursor` for you.
- Historical `bars` live under the asset-class namespace (`crypto`/`forex`/`stocks`/`dex`); `last.trade`/`last.quote` under `last`; fundamentals under `stocks`.
- SDKs return the same `{ data, meta }` shapes documented above, so every field, error-code, and gzip rule in sections 1-5 still applies.

---

*SiftingIO provides market-data access via APIs. It is a data provider, not an exchange, broker, or investment advisor. This material is for technical/educational use and is not investment advice.*
<!-- BODY:END -->
