---
name: Check whether a market is open and read its calendar
description: >-
  Discover market slugs, check open/closed status for one market or all of them, read the weekly
  trading schedule and the holiday/early-close calendar. Use before quoting a live price, when
  scheduling a job around a session, or when asked "is the market open?".
api: openapi/siftingio-markets-api-openapi.yml
operations:
  - listMarkets
  - getAllMarketStatus
  - getMarketStatus
  - getMarketHours
  - getMarketCalendar
  - getEconomicCalendar
generated: '2026-08-17'
method: generated
source: openapi/_original/siftingio-openapi.yaml + https://sifting.io/docs/quickstart
---

# Market hours, status and calendars

Base URL `https://api.sifting.io`, header `X-API-Key: <SIFTING_API_KEY>`.

## Step 1 — get the slug right (`listMarkets`)

`GET /v1/fnd/markets` (optional `?region=`) returns the catalog: exchanges and asset classes such
as `nyse`, `us_equities`, `forex`, `crypto`. Each `Market` carries `market`, `name`, `type`,
`timezone`, `region`, `exchanges` and `stats`.

**Always resolve the slug here first.** Every other operation in this skill takes `{market}` and an
invented slug returns `404`.

## Step 2 — status

- One market: `getMarketStatus` → `GET /v1/fnd/markets/{market}/status`
- All at once: `getAllMarketStatus` → `GET /v1/fnd/markets/status` (optional `?region=`)

Use `getAllMarketStatus` when you need a dashboard; use `getMarketStatus` when you already know
the one slug. Do not fan out N single-market calls — that is what the all-status route is for, and
it costs one call against your quota instead of N.

`MarketStatus` gives `is_open`, `state`, `session`, `next_open`, `next_close`, `timezone`. Read
`next_open`/`next_close` rather than computing them from hours yourself — they already account for
holidays and early closes.

## Step 3 — the recurring schedule (`getMarketHours`)

`GET /v1/fnd/markets/{market}/hours` returns the weekly schedule as `MarketHoursBlock` →
`HoursSpec` → `HoursBreak`. **Handle `HoursBreak`**: several Asian venues have an intraday lunch
break, so "between open and close" is not the same as "trading". Continuous markets also expose
`ForexSession` blocks rather than exchange hours — forex and crypto are not shaped like an
exchange.

## Step 4 — holidays and early closes (`getMarketCalendar`)

`GET /v1/fnd/markets/{market}/calendar?from=&to=` returns `CalendarDay` entries. `to` must be
after `from`. Use this before scheduling anything — a "next business day" computed from a plain
weekday rule will be wrong on a holiday and on a half-day.

## Step 5 — macro events (`getEconomicCalendar`)

`GET /v1/fnd/economic-calendar` with `from`, `to`, `country`, `impact`, `agency`, `event_id`,
`limit`. US macro events with consensus, actual, revisions and exact timestamps.

`to` must be **strictly** after `from` and the range is capped at **365 days** — otherwise
`400 invalid_date_range`. For a longer span, page it in yearly windows.

## Timezones — the one thing to get right

Everything is UTC. Date-only fields are `YYYY-MM-DD`; timestamps are `YYYY-MM-DDTHH:MM:SSZ`;
tick timestamps are int64 epoch milliseconds. Each market also reports its own `timezone`, and
the two are not interchangeable. Convert for display only — never store a local time.

## Errors

| Status | Code | Do this |
|---|---|---|
| 400 | `invalid_date_range` | Calendar range invalid or over 365 days. Split the window. |
| 401 | `unauthorized` | Key problem. Do not retry. |
| 404 | — | Unknown market slug. Go back to `listMarkets`. |
| 429 | `rate_limit_exceeded` | Sleep `Retry-After`. |

## Notes

- All operations are `GET`; retrying is safe.
- Cheap and cacheable. The hours and calendar data change rarely — cache them per market per day
  rather than calling on every request. Status is refreshed roughly every 15 seconds, so caching
  it for more than a few seconds defeats the point.
- There is a no-key public version of this data for humans at https://sifting.io/market-hours,
  and an embeddable badge at https://sifting.io/tools/widgets.
