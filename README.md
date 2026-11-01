# SiftingIO

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Cross-asset market data APIs covering US equities, forex, cryptocurrency, DeFi/on-chain, commodities, and SEC/EDGAR fundamentals, aggregated across venues and normalized into one JSON schema so every asset class shares the same fields, auth model and integration flow. Delivered over REST (37 operations under a public OpenAPI 3.1 contract), a WebSocket stream described by a public AsyncAPI 3.0 contract, and a documented FIX 4.4 market-data feed. Fundamentals cover filings, filing-text extraction and risk-factor diffs, XBRL financials, ratios, insiders, ownership and 13F holdings; reference data covers market status, hours and holiday calendars; derived data covers technical signals and on-chain TVL. Official Go, Python and JavaScript SDKs, an open-source (locally-run) MCP server with 36 tools, and a provider-published agent ruleset. Operated by SaltingIO LLC, a Wyoming company; explicitly not a broker, exchange or trading venue.

**Areas:** Financial market data · Stocks/equities · Foreign Exchange · Cryptocurrency · DeFi/on-chain · Commodities · SEC Filings · XBRL · Fundamentals · Fintech · Quant/trading infrastructure · Real-Time Streaming

## SiftingIO on APIs.io

Everything profiled for SiftingIO is published on APIs.io rather than in this repository. Start at the provider page — **[apis.io/providers/siftingio/](https://apis.io/providers/siftingio/)** — or go straight to one of the APIs or properties below.

### APIs (13)

- [SiftingIO Live Stream](https://apis.io/apis/siftingio/siftingio-live-stream/) — WebSocket streaming API for live market data, described by an AsyncAPI 3.0 contract. Also offers a documented FIX API.
- [SiftingIO MCP Server](https://apis.io/apis/siftingio/siftingio-mcp-server/) — LOCALLY-RUN MCP server, not a hosted endpoint. The client launches it (`npx -y siftingio-mcp`, or the Python package) and it runs on the user's own machine,…
- [SiftingIO Commodities API](https://apis.io/apis/siftingio/siftingio-commodities-api/) — Historical OHLC bars for commodities (metals, energy, industrials).
- [SiftingIO Convert API](https://apis.io/apis/siftingio/siftingio-convert-api/) — Live currency and token conversion across forex and crypto.
- [SiftingIO Crypto API](https://apis.io/apis/siftingio/siftingio-crypto-api/) — Historical OHLCV bars for crypto symbols.
- [SiftingIO DEX API](https://apis.io/apis/siftingio/siftingio-dex-api/) — On-chain wallet portfolios and historical DEX bars.
- [SiftingIO Filers API](https://apis.io/apis/siftingio/siftingio-filers-api/) — 13F institutional holdings.
- [SiftingIO Forex API](https://apis.io/apis/siftingio/siftingio-forex-api/) — Historical OHLC bars for FX pairs.
- [SiftingIO Live API](https://apis.io/apis/siftingio/siftingio-live-api/) — Live market data read from the engine's latest snapshot.
- [SiftingIO Markets API](https://apis.io/apis/siftingio/siftingio-markets-api/) — Market catalog, status, hours, and holiday calendars.
- [SiftingIO Signals API](https://apis.io/apis/siftingio/siftingio-signals-api/) — Technical-analysis signals (live and historical) across stocks, forex, crypto, and commodities.
- [SiftingIO Stocks API](https://apis.io/apis/siftingio/siftingio-stocks-api/) — US-equity fundamentals (SEC) and historical bars.
- [SiftingIO Economic Calendar API](https://apis.io/apis/siftingio/siftingio-economic-calendar-api/) — US macro economic events.

### Published properties

Each property below is a page on APIs.io carrying the machine-readable artifact and what it says.

#### AsyncAPI (1)

Event-driven and webhook contracts.

- [SiftingIO Live Stream](https://apis.io/asyncapis/siftingio/siftingio-asyncapi/)

#### Event channels (1)

Individual events and webhooks the provider emits.

- [/ws/v1](https://apis.io/channels/siftingio/ws-v1/)

#### MCP servers (2)

Model Context Protocol servers that expose the API to agents.

- [io.sifting/mcp](https://apis.io/servers/siftingio/io-sifting-mcp/)
- [mcp](https://apis.io/servers/siftingio/mcp/)

#### Agentic access (1)

Which operations an agent can call, which act, and which need a human in the loop.

- [Siftingio Agentic Access](https://apis.io/agentic-access/siftingio/siftingio-agentic-access/)

#### Overlays (11)

OpenAPI overlays, including conversational phrasing for each operation.

- [Siftingio Commodities Overlay](https://apis.io/overlays/siftingio/siftingio-commodities-overlay/)
- [Siftingio Convert Overlay](https://apis.io/overlays/siftingio/siftingio-convert-overlay/)
- [Siftingio Crypto Overlay](https://apis.io/overlays/siftingio/siftingio-crypto-overlay/)
- [Siftingio Dex Overlay](https://apis.io/overlays/siftingio/siftingio-dex-overlay/)
- [Siftingio Economiccalendar Overlay](https://apis.io/overlays/siftingio/siftingio-economiccalendar-overlay/)
- [Siftingio Filers Overlay](https://apis.io/overlays/siftingio/siftingio-filers-overlay/)
- [Siftingio Forex Overlay](https://apis.io/overlays/siftingio/siftingio-forex-overlay/)
- [Siftingio Live Overlay](https://apis.io/overlays/siftingio/siftingio-live-overlay/)
- [Siftingio Markets Overlay](https://apis.io/overlays/siftingio/siftingio-markets-overlay/)
- [Siftingio Signals Overlay](https://apis.io/overlays/siftingio/siftingio-signals-overlay/)
- [Siftingio Stocks Overlay](https://apis.io/overlays/siftingio/siftingio-stocks-overlay/)

#### Security (2)

Authentication, trust centre, and vulnerability disclosure.

- [Siftingio Authentication](https://apis.io/security/siftingio/siftingio-authentication/)
- [Siftingio Domain Security](https://apis.io/security/siftingio/siftingio-domain-security/)

#### Plans and pricing (1)

Published plans, tiers and prices.

- [Siftingio Plans Pricing](https://apis.io/plans/siftingio/siftingio-plans-pricing/)

#### Rate limits (1)

Published request limits.

- [Siftingio Rate Limits](https://apis.io/rate-limits/siftingio/siftingio-rate-limits/)

## Where to find this API

Links to the company's own pages. This repository publishes no copy of their specifications — the canonical document is theirs and lives at the URLs below.

- [Website](https://sifting.io)
- [Documentation](https://sifting.io/docs)
- [Getting started](https://sifting.io/docs/quickstart)
- [API reference](https://sifting.io/docs)
- [Pricing](https://sifting.io/pricing)
- [Sign up](https://sifting.io/register)
- [Support](https://sifting.io/contact)
- [Changelog](https://sifting.io/changelog)
- [Blog](https://sifting.io/blog)
- [Terms of service](https://sifting.io/legal/terms-of-use)
- [Privacy policy](https://sifting.io/legal/privacy-policy)
- [GitHub](https://github.com/SiftingIO)
- [Status](https://siftingio.instatus.com/)

## API Evangelist rating

**Kin Score 61.1** — strong · rubric 0.25.0 · scored 2026-10-23

| Facet | Score |
|---|---|
| Discoverability | 63.3 |
| Contract quality | 54.3 |
| Contract governance | 4.5 |
| Operational transparency | 57.9 |
| Developer ergonomics | 83.3 |
| Access clarity | 76.3 |

**Agent readiness 43.2** — agent-native

| Agent-readiness dimension | Result |
|---|---|
| Spec Presence | yes |
| Agentic Access | derived (partial credit) |
| Reversibility Documented | na |
| MCP Server | documented (partial credit) |
| Auth Clarity | bearer (partial credit) |
| Idempotency | na |
| Error Semantics | documented (partial credit) |
| OpenAPI Examples | partial |
| Rate Limit Signal | verified |
| Event Surface Described | derived (partial credit) |
| Agent Skills | yes |
| Well Known Catalog | — |
| Consent Identity | — |
| Agent Card | — |
| Dry Run Mode | na |
| Delegated Identity | — |
| Protected Resource Metadata | — |
| Dynamic Client Registration | — |
| Agentic Commerce | — |
| Pagination Ratio | partial |
| Operation Distinctiveness | distinct |

**Regulatory alignment 28.7** — Securities & Market Data

The rubric is published at [github.com/api-evangelist/kin-score](https://github.com/api-evangelist/kin-score). The per-check breakdown behind these numbers — which checks were earned, partial or missed — along with score history and how this provider compares to its peers, is available at [apis.io/providers/siftingio/](https://apis.io/providers/siftingio/).

## How this profile is made

API Evangelist profiles a company's public API surface the way a developer would find it: the website, developer portal and documentation, the specifications it publishes, its public repositories, and its pricing, status and changelog pages. From that it derives the properties above — contracts split per API, workflows, schemas, collections, governance rules, plans and rate limits — and scores them against the published [Kin Score rubric](https://github.com/api-evangelist/kin-score).

The artifacts are published on [APIs.io](https://apis.io), not in this repository. This repository is the public record for SiftingIO: the place to say something is wrong, missing or out of date, and to ask for a re-score or a removal.

## Open an issue

- **Something about this listing** — a wrong description, a missing API, a stale artifact or a question about the rating: [open an issue on this repository](https://github.com/api-evangelist/siftingio/issues/new).
- **A new API, or one that is not in the catalog yet** — [submit it on APIs.io](https://apis.io/add/).
- **Anything else** — [the APIs.io inbox](https://github.com/api-search/inbox/issues/new) or [info@apievangelist.com](mailto:info@apievangelist.com).
