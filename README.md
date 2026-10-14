# Agent Weather

Agent Weather is open-source intelligence on AI agents and x402 payments. We read public sources on a schedule (public status pages, GitHub, npm, the MCP Registry, the OpenRouter model list and the XRP Ledger) and keep every reading with its source and timestamp. That record is the product: each paid call returns the latest raw readings plus what the record shows over 24h and 7d. Derived values show their formula; gaps are reported, not filled.

We collect verifiable signals—agent frameworks and marketplaces, on-chain agent and x402 payment activity, public API/usage stats, GitHub activity, agent directories—and turn them into agent-consumable conditions reports: near real-time (or at least daily) intelligence on current agent-ecosystem conditions, as structured JSON with sources, timestamps, and confidence. Modeled or estimated values are labeled.

Product: pay-per-request on the XRPL AI Hub via x402 (0.01 RLUSD (roughly one cent) on core routes; routes priced 0.01 or more also accept 0.01 USDC on Base). Live API: https://api.agentweather.io (v1.3.0) · Discovery: `/.well-known/x402` · Free sample: `/v1/sample/yesterday` · New in 1.3.0: [model-metrics directory](#model-metrics-directory-new-in-130) (`/v1/models`)

**Our source is private. Our sources are not.**
Agent Weather's code is closed to protect the methods we've built. What you buy is open: every reading in a report lists its public source and the time it was read, and [/v1/sources](https://api.agentweather.io/v1/sources) shows every source we read.

**Estimated cost comparison (as of Oct 13, 2026):** in our own test that day, gathering the same readings directly took about 16 requests and about 8 seconds. Depending on the model used, we estimated the model cost at roughly $0.005 to $0.69 per pass, and that excludes the history we keep. One Agent Weather call is $0.01. These are internal estimates based on model prices on that date, not a promise of savings; actual costs change over time and depend on your setup. See our [terms](https://api.agentweather.io/v1/terms).

**All products are recorded measurements with sources and timestamps; they may be late, incomplete or wrong.** Reports, decide readings and Pay Radar scans are measurements, not advice. Every paid response carries `disclaimer` and `terms` fields. Terms of use: [`/v1/terms`](https://api.agentweather.io/v1/terms).

Help grow the history: contribute signals or data (agents and people welcome) so the record is deep enough to support forecasts and predictions later.

<p align="center">
  <a href="https://api.agentweather.io/.well-known/x402">x402 catalog</a> ·
  <a href="https://xrpl-ai.org/directory">XRPL AI Hub</a> ·
  <a href="https://api.agentweather.io/openapi.json">OpenAPI</a> ·
  <a href="https://api.agentweather.io/llms.txt">llms.txt</a> ·
  <a href="./docs/methodology.md">Methodology</a> ·
  <a href="https://x.com/agentweatherio">@agentweatherio</a>
</p>


---

## TL;DR for agents

| | |
|---|---|
| **Endpoint** | `GET https://api.agentweather.io/v1/agent-weather/current` (POST behaves identically) |
| **Price** | `0.01 RLUSD` or `0.01 USDC` on Base per call (other routes: see [`openapi.yaml`](https://api.agentweather.io/openapi.json) / live catalog) |
| **Free sample** | `GET https://api.agentweather.io/v1/sample/yesterday`: yesterday's report in the same format, no payment |
| **Pay Radar** | `GET https://api.agentweather.io/v1/payradar?url={endpoint}` (0.003 RLUSD): pre-flight scan of an XRPL x402 endpoint before paying. Free demo `/v1/payradar/demo`. The old path `/v1/paycheck` still works |
| **Pre-flight decide** | `/v1/decide/cheapest-model` (0.003 RLUSD), `/v1/decide/provider-health` (0.001 RLUSD) |
| **MCP server** | `https://mcp.agentweather.io/mcp` (streamable HTTP, v0.4.2, 20 tools); server card at `/.well-known/mcp/server-card.json` |
| **Terms** | `GET https://api.agentweather.io/v1/terms` (free) |
| **Model directory (new in 1.3.0)** | `GET https://api.agentweather.io/v1/models` (free); per-model and per-category reports 0.01 RLUSD each, see [below](#model-metrics-directory-new-in-130) |
| **API version** | `1.3.0` (`GET /healthz` reports it) |
| **Protocol** | x402 v2, scheme `exact`, network `xrpl:0` (XRPL mainnet) |
| **Pay to** | `rnFHPGmgTLSg7hjzfr8wiHaTdTh3PijrZZ` |
| **RLUSD issuer** | `rMxCKbEDwqr76QuheSUMdEGf4B9xJ8m5De` |
| **Facilitator** | T54 XRPL mainnet, `https://xrpl-facilitator-mainnet.t54.ai` |
| **USDC on Base** | Accepted on the routes priced 0.01 or more (current, x402-activity, merchant, health report, model report, model category): scheme `exact`, network `eip155:8453`, asset `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`, pay to `0xA15c55f39Ec1dfd87Afb09e1959EFac2Bf44F4Ec`. Routes priced under 0.01 (pulse, price-index, decide, Pay Radar) are RLUSD-only. On USDC routes the USDC entry is listed first. XRP is not accepted. |
| **Discovery** | `GET https://api.agentweather.io/.well-known/x402` |
| **Health (free)** | `GET https://api.agentweather.io/healthz` |
| **Response** | `application/json`, schema in the [OpenAPI document](https://api.agentweather.io/openapi.json), cached up to 5 min |
| **Rate limit** | 30 requests/min per IP on the paid route |

1. An unpaid request returns `402 Payment Required`. The body and the `PAYMENT-REQUIRED` header (base64) list the RLUSD `accepts[]` entry (plus a USDC-on-Base entry on routes priced 0.01 or more) and a fresh single-use `invoiceId` ([example](./examples/402-response.json)).
2. Sign an XRPL `Payment` bound to that invoice, then retry with `PAYMENT-SIGNATURE`.
3. You receive the report ([free day-old sample](https://api.agentweather.io/v1/sample/yesterday)) plus a `PAYMENT-RESPONSE` header containing the XRPL transaction hash, which is your receipt.

**Try the format first, free:** `GET https://api.agentweather.io/v1/sample/yesterday` returns the stored conditions report from about 24 hours ago, in the same JSON format as the paid route, with a `sample` block marking it as free and not live. No payment, no 402. Buy `/v1/agent-weather/current` for current conditions.

## What you get

A snapshot of "conditions" across the agent economy. Every number is traceable to a public source:

- **Agent frameworks:** GitHub activity for LangChain, LangGraph, AutoGen, CrewAI, OpenAI Agents SDK and MCP servers
- **Provider status:** AI providers Anthropic and Morph (public status pages and active incidents, in `observations.providers`), plus GitHub platform status in `observations.platform_status`, credited per source (Agent Weather is not affiliated with any of them). Removed on 2026-10-13 and no longer measured: openai, cursor, cohere, perplexity, inference-net, amazon (null until 2026-11-12; see `/v1/sources`)
- **MCP ecosystem:** Official MCP Registry entries updated in the last hour
- **Adoption:** npm weekly downloads for `@modelcontextprotocol/sdk`, `@x402/core` and `@x402/xrpl`
- **Settlement rail:** XRPL mainnet `server_info` (ledger age, fees, load)
- **Derived indices:** visibility (data coverage), pressure (AI-provider disturbance), temperature (x402 activity), and a modeled confidence index; copied to top-level `indices`, with a plain-text `summary` built from the readings

Every source is reported with `observed_at` and a status. A source that fails comes back as `null` plus a `data_gaps` entry, so nothing is silently invented. Derived values carry `modeled: true` and their formula. See [docs/methodology.md](./docs/methodology.md).

## Why weather?

Agents need to decide whether to act *now*: which provider to call, whether a framework is healthy, whether the payment rail is congested. Agent Weather packages those signals the way a forecast does.

| Weather term | Meaning in the current report |
|---|---|
| Visibility | Fraction of expected sources that reported (observed) |
| Pressure | Share of tracked providers (Anthropic, GitHub) whose status page shows a disturbance (modeled) |
| Alerts | Active provider incidents |
| Temperature / wind / fronts / forecast | Not yet populated. They need a stored baseline and are listed in `data_gaps` rather than invented |
| Confidence | Modeled 0–1 score (`calibrated: false`): visibility scaled by 24h/7d history coverage. It measures data coverage and continuity, not accuracy |

## Quickstart: pay as an agent (x402 over HTTP)

1. `GET` a paid route. You receive HTTP 402 with a `PAYMENT-REQUIRED` header (base64 JSON, same as the body) listing `accepts[]`: scheme `exact`, network `xrpl:0` (RLUSD, issuer `rMxCKbEDwqr76QuheSUMdEGf4B9xJ8m5De`) or `eip155:8453` (USDC on Base, on routes priced 0.01+), `amount`, `payTo` and an `invoiceId`.
2. Check `amount` against your own cap, then sign the payment with any x402 client for that network (for XRPL, T54's [`x402-xrpl`](https://www.npmjs.com/package/x402-xrpl) client on npm or PyPI).
3. Retry the same request with the `PAYMENT-SIGNATURE` header. On success you get HTTP 200 with the JSON body and a `PAYMENT-RESPONSE` header (base64 JSON `{success, transaction, network, payer}`); `transaction` is your on-ledger receipt.

> **Compatibility:** the reference `@x402/xrpl` client can't currently pay through the T54 mainnet facilitator ([x402#3527](https://github.com/x402-foundation/x402/issues/3527)). Use `x402-xrpl` (npm or PyPI) until that is fixed. Payers settling through T54 carry SourceTag `804681468`, which is how the XRPL AI Hub Index attributes the sale.

## Inspect the 402 without paying

```bash
curl -i https://api.agentweather.io/v1/agent-weather/current
curl -s https://api.agentweather.io/v1/sample/yesterday | jq '.sample, .conditions'   # free sample, ~24h old
curl -s https://api.agentweather.io/.well-known/x402 | jq
curl -s https://api.agentweather.io/healthz | jq
```

## Model-metrics directory (new in 1.3.0)

Assistants and agents increasingly route each task to a different model from a different vendor. A router needs fresh, machine-readable conditions for each model: what it costs, whether that changed, whether the vendor reports an incident, whether the model is being retired. The model-metrics directory reports those conditions for the AI models on OpenRouter's public model list (465 models from 63 vendors on Oct 7, 2026).

These are **conditions reports, not rankings**. They make no statement about how well any model performs, and nothing is ordered by performance. Every derived value ships with its formula in the response (`derived_fields`).

| Route | What you get | Price |
|---|---|---|
| `GET /v1/models` | Directory index: id, name, vendor, categories, input/output modalities, context length, `price_in_usd_per_1m`, `price_out_usd_per_1m`, expiration date, `first_seen`, vendor status. Filters `?vendor=`, `?category=`, `?q=`, paging with `?limit=` (max 500) and `?offset=` | free |
| `GET /v1/sample/models-yesterday` | Day-old change summary: models added and removed, price changes, expiration-date changes and vendor status changes observed during the previous day (America/Chicago) | free |
| `GET /v1/models/report?id={id}` | One model: current prices with 24h and 7d change (absolute and %), standard job cost, context and modalities, vendor status and active incidents, events in the last 24h and 7d, days until expiration, sources, timestamps, data age, and a one-line summary | 0.01 RLUSD |
| `GET /v1/models/category/{name}` | One category: counts, 24h price movers, new and removed models, models with an expiration date, vendor disturbances, and every model in the category with its standard job cost as a plain fact. Ordered by id by default; `?sort=cost` orders by cost and says so ("sorted by cost, not quality") | 0.01 RLUSD |

Paid routes use the same x402 flow as the rest of the API (RLUSD on `xrpl:0`, or USDC on Base `eip155:8453`). An unknown model id or category gets `400` before any payment, so nothing is charged.

**Derived values**

- `price_in_usd_per_1m` / `price_out_usd_per_1m` = OpenRouter's USD-per-token price × 1,000,000. OpenRouter lists `-1` for router entries whose price depends on the routed model; those come back as `null` with a note.
- `standard_job_cost_usd` = 10,000 × input price + 1,000 × output price (a fixed reference job of 10,000 input + 1,000 output tokens; other listed fees such as images, web search or caching are excluded).
- 24h change compares with the hourly snapshot from at least 24 hours ago. 7d change compares with the daily baseline (first snapshot after midnight America/Chicago) from 7 days ago. Change % = (current − reference) ÷ reference × 100.
- `days_until_expiration` = whole days from now to the listed `expiration_date`.

**Categories** (derived only from declared metadata; each response states the rule)

| Category | Rule |
|---|---|
| `reasoning` | OpenRouter's `reasoning` object is present, or `supported_parameters` includes `reasoning` / `include_reasoning` |
| `coding` | The id or name contains the token code, coder, codex, codestral, devstral or coding, or the first sentence of the description calls it a coding/code model or agent (self-described) |
| `vision-input` | `image` is in `input_modalities` |
| `image-output` | `image` is in `output_modalities` |
| `audio` | `audio` is in the input or output modalities |
| `retiring-soon` | `expiration_date` is set and is at most 60 days away |
| `price-moved` | An input or output price change was observed in the last 24 hours |
| `free-tier` | Both prices are 0, or the id ends in `:free` |

**Data sources and attribution**

- Model list, prices, context length, modalities and expiration dates: the [OpenRouter public models list](https://openrouter.ai/api/v1/models) (no key), collected hourly. Data courtesy of OpenRouter. Prices are OpenRouter's listing and may differ from a vendor's direct price.
- Vendor status (every 10 minutes): public Atlassian Statuspage `summary.json` feeds for Anthropic and Morph, credited per source (Agent Weather is not affiliated with either). OpenAI, Cohere, Perplexity, Inference.net and Amazon were removed on 2026-10-13 and return status null with a deprecation note until 2026-11-12. Other vendors return unknown with the reason.
- Status is **`unknown`** wherever a vendor has no free status feed (for example Mistral, xAI, DeepSeek, Qwen, Meta and OpenRouter itself), or when the last check is more than 30 minutes old. It is never guessed. Status is vendor-level, not per model or per hosting provider.

**History fills in over time.** Collection started on Oct 7, 2026. 24h changes fill in from Oct 8, and 7d changes from Oct 14. Until then those fields are `null`, and the response explains why. Coverage is text-heavy (12 image-output and 4 audio-output models on OpenRouter today). Dedicated image and music services such as MidJourney and Suno are not on the list.

```bash
curl -s 'https://api.agentweather.io/v1/models?category=coding&limit=5' | jq '.total, .models[].id'
curl -s https://api.agentweather.io/v1/sample/models-yesterday | jq '.summary'
curl -i 'https://api.agentweather.io/v1/models/report?id=openai/gpt-4o'      # 402: pay 0.01 RLUSD via x402
```

One entry from `GET /v1/models` (trimmed):

```json
{
  "id": "openai/gpt-4o",
  "name": "OpenAI: GPT-4o",
  "vendor": "openai",
  "categories": ["vision-input"],
  "input_modalities": ["text", "image", "file"],
  "output_modalities": ["text"],
  "context_length": 128000,
  "price_in_usd_per_1m": 2.5,
  "price_out_usd_per_1m": 10,
  "expiration_date": null,
  "first_seen": "2026-10-07T13:39:34.249Z",
  "status": "operational"
}
```

## Pulse sources and attribution

Pulse routes (`/v1/pulse/storm`, `/v1/pulse/providers`, `/v1/pulse/facilitator`) are recorded measurements that may be late or wrong. They use:

- public AI-provider status pages via the public Statuspage API (currently the GitHub Status page and the Claude status page), polled at most once per 60 s;
- our own measurement of t54 XRPL x402 facilitator reachability (`/supported`, every 5 min);
- our own scan of public validated XRPL ledgers, our own unpaid endpoint probes and our stored snapshots.

Every pulse source in a response carries `attribution`, `terms_url` and `not_affiliated`. The free list is at [`/v1/sources`](https://api.agentweather.io/v1/sources). Agent Weather is not affiliated with, endorsed by or sponsored by any provider, status page or facilitator it measures.

Example `/v1/pulse/providers` body (pulse-0.2.0, one source shown; live responses list every tracked source):

```json
{
  "schema_version": "pulse-0.2.0",
  "signal": "providers",
  "degraded": false,
  "level": "operational",
  "reason": "all 2 tracked providers report no incident",
  "asOf": "2026-10-14T01:01:38.437Z",
  "providers": { "anthropic": "none", "github": "none", "openai": null, "cursor": null },
  "sources": [
    { "id": "github", "url": "https://www.githubstatus.com/api/v2/summary.json", "observed_at": "2026-10-14T01:01:38.539Z", "status": "ok",
      "attribution": "Source: GitHub Status (githubstatus.com), public Statuspage API, polled at most once per 60 s",
      "terms_url": "https://docs.github.com/en/site-policy/github-terms/github-terms-of-service",
      "not_affiliated": "Agent Weather is not affiliated with, endorsed by or sponsored by this source." }
  ],
  "deprecated": { "keys": ["openai", "cursor"], "until": "2026-11-12", "note": "providers.openai and providers.cursor are no longer measured and always return null; the keys will be removed after 2026-11-12. See /v1/sources." },
  "notice": "Recorded measurements. Third-party status data is provided as is, may be late or wrong, and is not a statement by the source. Sources and attributions: /v1/sources."
}
```

## Changelog

- **2026-10-14: `/v1/agent-weather/current` adds `summary` and `indices`.** `summary` is plain text built only from the readings in the report (raw measurements first); `indices` copies `conditions.visibility/pressure/temperature.value` (`conditions` is unchanged). `observations.providers` now lists the AI providers Anthropic and Morph; GitHub Status moved to `observations.platform_status`. `conditions.pressure` inputs changed from Anthropic + GitHub to Anthropic + Morph (documented in `methodology.pressure_inputs`; no 1h change is computed across the switch). MCP Registry readings now retry stalled requests. MCP server 0.4.2. Prices and terms are unchanged.
- **2026-10-13: Pulse schema pulse-0.2.0.** `/v1/pulse/providers` and `/v1/pulse/storm` no longer measure the OpenAI and Cursor status pages. `providers.openai` and `providers.cursor` stay in the response as `null` until 2026-11-12 so existing parsers keep working; after that the keys are removed. New fields: per-source `attribution`, `terms_url`, `not_affiliated`; top-level `deprecated` and `notice`. New free route `/v1/sources`. The terms at `/v1/terms` (pending legal review) add clause 4a on third-party status data. Prices and payment options are unchanged.

## Reliability

- Live on XRPL mainnet since Sep 26, 2026.
- Settlement completes **before** data is served. Each payment is then independently re-checked on-ledger (validated, `tesSUCCESS`, exact `delivered_amount`, correct destination, `InvoiceID = SHA-256(invoiceId)`).
- Invoices are single-use and tx hashes are deduplicated, so one payment can't unlock two responses.
- Status page / SLA: none published yet. Use the free [`/healthz`](https://api.agentweather.io/healthz) endpoint for liveness.

## Pricing

Prices are in RLUSD (API v1.3.0). Routes priced 0.01 or more also accept the same amount in USDC on Base; routes priced under 0.01 (pulse, price-index, decide, Pay Radar) are RLUSD-only.

| Resource | Price |
|---|---|
| `/v1/agent-weather/current` | 0.01 RLUSD |
| `/v1/agent-weather/x402-activity` | 0.01 RLUSD |
| `/v1/pulse/storm`, `/v1/pulse/facilitator`, `/v1/pulse/providers` | 0.001 RLUSD each |
| `/v1/merchant/{address}` | 0.03 RLUSD |
| `/v1/health/report` | 0.01 RLUSD |
| `/v1/price-index` | 0.005 RLUSD |
| `/v1/models/report?id={id}` (new in 1.3.0) | 0.01 RLUSD |
| `/v1/models/category/{name}` (new in 1.3.0) | 0.01 RLUSD |
| `/v1/decide/cheapest-model` | 0.003 RLUSD |
| `/v1/decide/provider-health` | 0.001 RLUSD |
| `/v1/payradar?url=` (Pay Radar; old path `/v1/paycheck`) | 0.003 RLUSD |
| `/v1/sample/yesterday`, `/v1/models`, `/v1/sample/models-yesterday`, `/v1/payradar/demo`, `/v1/terms`, `/v1/sources` | free |

All paid resources are also listed in [`openapi.yaml`](https://api.agentweather.io/openapi.json). The buyer also pays the XRPL network transaction fee (typically ~10 drops). The live [`/.well-known/x402`](https://api.agentweather.io/.well-known/x402) catalog is the source of truth.

## Links

- API: https://api.agentweather.io
- X: [@agentweatherio](https://x.com/agentweatherio)
- XRPL AI Hub directory: https://xrpl-ai.org/directory
- Contact: message [@agentweatherio](https://x.com/agentweatherio) on X

## License

This documentation (`README.md`, `docs/`) is licensed [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/). The Agent Weather name and logo are not licensed for reuse beyond referring to the service. Agent Weather's source code is private. The hosted API at `api.agentweather.io` is a commercial pay-per-call service; see the [terms](https://api.agentweather.io/v1/terms).
