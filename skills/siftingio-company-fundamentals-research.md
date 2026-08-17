---
name: Research a US company's fundamentals from SEC filings
description: >-
  Resolve a company name to a ticker, then pull its profile, filings, XBRL financials, ratios and
  insider activity from SiftingIO's SEC-derived fundamentals API. Use when asked to research,
  summarise or compare US-listed companies from primary filing data.
api: openapi/siftingio-stocks-api-openapi.yml
operations:
  - searchStocks
  - getCompanyProfile
  - listFilings
  - getFiling
  - getFilingSections
  - getFilingSection
  - getRatios
  - getFinancials
  - getFinancialConcept
  - listInsiders
  - listEvents
generated: '2026-08-17'
method: generated
source: openapi/_original/siftingio-openapi.yaml + https://sifting.io/docs/quickstart + https://sifting.io/docs/errors
---

# Research a US company's fundamentals

Base URL `https://api.sifting.io`. Every path is pinned to `/v1`.

## Auth

Send the key as a header on every request:

```
X-API-Key: <SIFTING_API_KEY>
```

`?api_key=` also works but the docs discourage it — query strings leak into logs. Every `/v1/*`
path is gated: a missing key returns `401 {"error":"missing api key ..."}`, a bad one returns
`401 {"error":"invalid api key"}`.

## Step 1 — resolve the ticker (`searchStocks`)

Never guess a ticker. `GET /v1/fnd/stocks/search?q=apple&limit=5` returns candidates with
`ticker` and `cik`. Tickers are case-insensitive; CIKs come back as 10-digit zero-padded strings.

If you skip this and pass an unknown ticker downstream you get `404 unknown_ticker` — the ticker
must exist in the SEC's US ticker registry.

## Step 2 — profile (`getCompanyProfile`)

`GET /v1/fnd/stocks/{ticker}/profile` → `ticker`, `cik`, `name`, `exchanges`, `other_tickers`,
`sic_code`, `sic_description`, `entity_type`, `fiscal_year_end`. Use `sic_description` for sector
context and `fiscal_year_end` to interpret period boundaries before quoting any annual figure.

## Step 3 — the filings themselves (`listFilings`, `getFiling`)

`GET /v1/fnd/stocks/{ticker}/filings?form=10-K&limit=5`. Filter by `form`, `from`, `to`. Each
`Filing` carries `accession`, `form`, `filed_at`, `period_end`, `accepted_at`, `items`,
`primary_document_url` and `has_xbrl`.

- `filed_at` / `period_end` are date-only (`YYYY-MM-DD`); `accepted_at` is a full timestamp.
- Accessions come back dashed (`0000320193-25-000089`) and are accepted dashed **or** undashed.
  Any other shape returns `400 invalid_accession_format`.
- Only recent filings are in the window. An older accession returns `404 filing_not_found`.
- Page with `?cursor=<meta.next_cursor>`; stop when `meta.next_cursor` is null or absent.

## Step 4 — read the narrative (`getFilingSections`, `getFilingSection`)

Call `getFilingSections` **first** to learn which sections were actually extracted, then
`getFilingSection` for the text. Guessing a section name returns `400 invalid_section` with a
`valid_options` array in the body; a section that failed extraction returns `404
section_not_found` with an `available` array. Read those arrays instead of retrying blind.

For year-over-year risk drift use `getRiskFactorsDiff` (`/risk-factors-diff`) — it diffs Item 1A
across the two most recent 10-Ks. It needs at least two on record (`404 insufficient_filings`)
and both must be extractable (`422 risk_factors_unavailable`).

## Step 5 — numbers (`getRatios`, `getFinancials`, `getFinancialConcept`)

**These three require gzip.** Send `Accept-Encoding: gzip` or you get `406 gzip_required`. Most
HTTP clients add it for you; a raw `fetch`/`curl` does not.

- `getRatios` — computed ratio set for the ticker.
- `getFinancials` — the full XBRL bundle. Large; gzip is mandatory.
- `getFinancialConcept` — one concept as a time series: `/v1/fnd/stocks/{ticker}/financials/{concept}?taxonomy=us-gaap`.

Every monetary observation is `{ value, unit }` where `unit` is `USD`, `USD/shares` (EPS),
`shares`, or `pure` for dimensionless ratios. **Never strip the unit** — an EPS figure and a
revenue figure are both numbers and only the unit distinguishes them.

## Step 6 — who is buying and what just happened

- `listInsiders` — Form 3/4/5 transactions. Note the tighter page cap here: `limit` defaults to
  10 and maxes at 25, not 200.
- `listEvents` — 8-K material events, filterable by `item`.
- `listOwnership`, `listCompensation` — same cursor pattern.

Every one of these cites the `accession` it was extracted from. When you report a figure, cite
that accession — the provenance is in the payload, so there is no excuse for an unsourced claim.

## Errors to handle

| Status | Code | Do this |
|---|---|---|
| 401 | `unauthorized` | Key missing/invalid. Do not retry. |
| 403 | (raw message) | Key is valid but the plan doesn't include US Stocks. Do not retry. |
| 404 | `unknown_ticker` | Go back to `searchStocks`. |
| 400 | `invalid_section` | Read `valid_options` from the body. |
| 406 | `gzip_required` | Add `Accept-Encoding: gzip` and retry once. |
| 429 | `rate_limit_exceeded` | Sleep `Retry-After` seconds. Self-throttle on `X-RateLimit-Remaining`. |
| 502/503 | `upstream_error`, `malformed_upstream`, `upstream_rate_limited` | Transient upstream (EDGAR). Retry with backoff. |

Switch on the snake_case `error` field, not the status — the provider states the code is the
stable identifier and the status maps only loosely.

## Notes

- All operations are `GET`. Retrying is always safe; there is no idempotency key because there is
  nothing to de-duplicate.
- 13F positions (`getFilerHoldings`) identify securities by **CUSIP only**, and the API exposes no
  CUSIP→ticker resolver. You cannot join 13F holdings to company profiles without an external
  mapping — say so rather than guessing.
- This is aggregated, derived and calculated data, explicitly not exchange-of-record. See
  https://sifting.io/data-methodology.
