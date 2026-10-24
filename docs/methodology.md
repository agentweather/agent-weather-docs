# Agent Weather methodology

Applies to report `schema_version` **0.1.0-draft**, which is what the live API serves today (server v1.3.0).

New in 1.3.0, the model-metrics directory (`/v1/models`) is a conditions report on AI models, not a ranking. It uses the models.dev catalog (MIT) and the vendors' free status feeds, and every derived value carries its formula (details in the [README](../README.md#model-metrics-directory-new-in-130)).

## Principles

- **Never invent.** A failed source yields `null` plus a `data_gaps` entry.
- **Label everything.** Values are `observed: true` (read directly from a source) or `modeled: true` (derived; each reading lists its sources and time).
- **Timestamp everything.** Every entry in `sources[]` has `name`, `url`, `observed_at` and `status`.
- **Cache:** reports are cached for up to 5 minutes. `valid_from`/`valid_to` give a 15-minute validity window.

## Sources used in the live report

| Area | Source | Endpoint | Notes |
|---|---|---|---|
| Agent frameworks | GitHub REST API | `api.github.com/repos/{owner}/{repo}` for langchain-ai/langchain, langchain-ai/langgraph, microsoft/autogen, crewAIInc/crewAI, openai/openai-agents-python, modelcontextprotocol/servers | Stars, forks, open issues+PRs (GitHub counts PRs as issues), push recency. Subject to GitHub rate limits; on failure the fields are `null` and a gap is recorded. |
| Provider status | Statuspage v2 | AI providers: Morph `status.morphllm.com` (Anthropic `status.claude.com` paused since 2026-10-20; see `/v1/sources`) (`observations.providers`). Platform: GitHub `www.githubstatus.com` (`observations.platform_status`, since 2026-10-14). Removed 2026-10-13: openai, cursor, cohere, perplexity, inference-net, amazon (null until 2026-11-12) | Indicator, description, degraded components. |
| MCP ecosystem | Official MCP Registry | `registry.modelcontextprotocol.io/v0/servers` | Count of server entries updated in the last hour. The registry is in preview (v0). |
| Adoption | npm downloads API | `api.npmjs.org/downloads/point/last-week/{pkg}` for `@modelcontextprotocol/sdk`, `@x402/core`, `@x402/xrpl` | Weekly downloads; npm data lags ~1–2 days. |
| Settlement rail | XRPL JSON-RPC `server_info` | xrplcluster.com | Validated ledger sequence and age, base fee, reserve, load factor. Read-only. |

## Pulse sources (pulse-0.2.0, 2026-10-13)

| Pulse input | Source | Notes |
|---|---|---|
| Provider status | Public Statuspage API: GitHub `www.githubstatus.com` (see `/v1/sources` for the current list) | Indicator per page; level = worst; cached 60 s. OpenAI and Cursor are no longer measured in pulse (`null` until 2026-11-12). |
| Facilitator | Own measurement of t54 `/supported` reachability (5 min) plus own XRPL ledger scan | A measurement of reachability and on-ledger success rate, not a statement about T54 Labs. |
| Fronts | Own stored 15-minute snapshots | First-party. |

Each pulse source carries `attribution`, `terms_url` and `not_affiliated`; see `/v1/sources`. Agent Weather is not affiliated with any source it measures. The "Sources used in the live report" table above covers the full conditions report, which is separate from pulse.

## Indices

| Field | Kind | Definition |
|---|---|---|
| `conditions.visibility` | observed | Fraction of expected sources that reported successfully (0–1): 15 public sources (6 GitHub repos, 4 status pages, 3 npm packages, XRPL `server_info`, MCP Registry) plus the x402 activity section, which counts as not reporting when it is unavailable or stale (see `confidence.overall`). |
| `conditions.pressure` | modeled | Provider disturbance index (0–100) across monitored AI providers; source statuses and timestamps are listed in the report. |
| `conditions.temperature` | not yet computed | Planned activity reading. `null` until enough history exists. |
| `conditions.wind` | not yet computed | Planned. `null` until history exists. |
| `confidence.overall` | modeled, `calibrated: false` | A 0–1 data-quality summary reflecting source coverage and history continuity. Not calibrated against outcomes. |
| `alerts[]` | observed | Active provider incidents. |
| `fronts[]`, `forecast[]` | empty | Not produced until history exists, and listed in `data_gaps`. |

## Known gaps (as reported in `data_gaps`)

- Trend-based conditions (temperature, wind, humidity): no stored 30/90-day baseline yet.
- Forecast: no history yet, so it's omitted rather than invented.
- x402 payment volume: not included in the live report yet.
- Framework repo stats can be partially missing when the GitHub API rate-limits.

## Schema

The report schema in the [OpenAPI document](https://api.agentweather.io/openapi.json) is draft 0.3 and stays backward-compatible with 0.1 reports. It defines optional sections (baselines, x402 activity and its heuristic real-vs-automated split); those sections are absent from 0.1 responses.

## Sources evaluated but not used

- x402scan: no documented public API.
- OpenRouter rankings: HTML only.
- XRPL AI Hub (xrpl-ai.org): its terms prohibit systematic data extraction. It's used as a directory only, not as a data source.
- Some provider status pages (Mistral, OpenRouter, Hugging Face, Groq): blocked, or not machine-readable Statuspage JSON.
