---
name: apify-mcpc
description: "Finds, evaluates, and runs Apify Actors using the mcpc CLI. Searches Apify Store, compares Actors by stats and ratings, reads input schemas to build correct inputs, runs Actors via call-actor, and retrieves results. Covers 8 marketing intelligence use cases (audience analysis, brand monitoring, competitor intelligence, content analytics, influencer discovery, lead generation, market research, trend analysis) plus multi-actor workflow patterns with domain-specific Actor suggestions and gotchas. Use when user wants to scrape data, extract information from websites, run automation, find tools on Apify platform, or search Apify/Crawlee documentation. Do NOT use for developing new Actors (use apify-actor-development skill instead)."
allowed-tools: Bash(mcpc *), Bash(jq *), Bash(gh issue *), Read, Grep, Glob
argument-hint: "[use-case or search query]"
---

# mcpc-apify

## Prerequisites

Run before first use in a session:

```bash
bash ${CLAUDE_PLUGIN_ROOT}/skills/apify-mcpc/scripts/check_apify.sh
```

The script checks: mcpc CLI installed, OAuth profile for mcp.apify.com, active @apify and @apify-docs sessions.
If it fails:
- mcpc missing → `npm i -g @apify/mcpc`
- OAuth missing → `mcpc mcp.apify.com login`
- Sessions missing → `mcpc @apify restart`

## Environment

mcpc version: !`mcpc --version`

Active sessions:
!`mcpc`

## User-Agent Tracking

**ALWAYS include the `-H` header in ALL `mcpc @apify` and `mcpc @apify-docs` calls** for Apify usage analytics (same pattern as `run_actor.js` User-Agent in other Apify skills):

```
-H "User-Agent: apify-agent-skills/apify-mcpc-1.3.1/<action>"
```

Replace `<action>` with a short label describing the call: `search`, `fetch_details`, `call_actor`, `get_output`, `get_run`, `rag_browser`, `search_docs`, `fetch_docs`.

Example:
```bash
mcpc -H "User-Agent: apify-agent-skills/apify-mcpc-1.3.1/search" @apify tools-call search-actors keywords:="instagram"
```

## mcpc CLI Reference

Available tools and arguments (for building correct mcpc calls):
!`mcpc --help 2>/dev/null | head -30 || echo "mcpc not available — install with: npm install -g @apify/mcpc"`

## Use Cases

When possible, match the user's intent to a use case. Each file has suggested Actors, search keywords, and domain-specific gotchas.

| Use Case | File | When |
|---|---|---|
| Audience Analysis | [audience-analysis.md](references/use-cases/audience-analysis.md) | Demographics, follower behavior, engagement quality |
| Brand Monitoring | [brand-monitoring.md](references/use-cases/brand-monitoring.md) | Reviews, ratings, sentiment, brand mentions |
| Competitor Intelligence | [competitor-intelligence.md](references/use-cases/competitor-intelligence.md) | Competitor strategies, ads, pricing, positioning |
| Content Analytics | [content-analytics.md](references/use-cases/content-analytics.md) | Engagement metrics, campaign ROI, content performance |
| Influencer Discovery | [influencer-discovery.md](references/use-cases/influencer-discovery.md) | Find influencers, verify authenticity, partnerships |
| Lead Generation | [lead-generation.md](references/use-cases/lead-generation.md) | B2B/B2C leads, contact enrichment, prospecting |
| Market Research | [market-research.md](references/use-cases/market-research.md) | Market conditions, pricing, geographic opportunities |
| Trend Analysis | [trend-analysis.md](references/use-cases/trend-analysis.md) | Emerging trends, viral content, content strategy |
| Multi-Actor Workflows | [multi-actor-workflows.md](references/use-cases/multi-actor-workflows.md) | Chaining Actors, handle discovery via SERP, enrichment |

**Important**: Actor tables in use-case files are **suggestions, not closed lists**. In case of problems use `search-actors` for the latest options.

## mcpc Argument Syntax

All tool arguments use `:=` (no spaces around it). Values are auto-parsed: valid JSON becomes that type, otherwise treated as string.

```bash
keywords:="web scraper"          # string
limit:=5                         # number (valid JSON)
enabled:=true                    # boolean (valid JSON)
input:='{"startUrls":[...]}'     # object — entire JSON as single-quoted string
```

If the first argument starts with `{` or `[`, the whole input is parsed as one JSON structure (not key-value pairs). Pipe JSON via stdin as alternative.

## Connection Modes

### Docs-only mode (no token needed)

```bash
mcpc 'mcp.apify.com?tools=docs' connect @apify-docs
```

Tools: `search-apify-docs`, `fetch-apify-docs` — for searching Apify/Crawlee documentation.

### Full mode (requires authentication)

Authenticate via OAuth (opens browser):

```bash
mcpc mcp.apify.com login
```

Then create a persistent session:

```bash
mcpc mcp.apify.com connect @apify
```

Tools: `search-actors`, `fetch-actor-details`, `call-actor`, `get-actor-output`, `get-actor-run`, `apify-slash-rag-web-browser` + docs tools.

### Missing auth handling

If the task requires Actors and user is not authenticated:

1. Tell the user — they need to run `mcpc mcp.apify.com login` (opens browser for OAuth)
2. Do NOT fall back to docs-only mode silently — the user expects Actor results
3. If auth unavailable, explain what's possible without it (docs only) and ask how to proceed

## Quick Data Retrieval: rag-web-browser

For one-off web searches or URL fetching, use the dedicated tool directly — no need for the full search → call workflow:

```bash
mcpc @apify tools-call apify-slash-rag-web-browser query:="san francisco weather" maxResults:=10
mcpc @apify tools-call apify-slash-rag-web-browser query:="https://www.example.com"
```

Parameters: `query` (required), `maxResults` (default 3), `outputFormats` (array: `"markdown"` default, `"text"`, `"html"`). Use `outputFormats:='["text"]'` to save tokens.

Use when: user wants immediate data ("get me info about X", "fetch this URL"), not a specialized scraper.

## Workflow: Find → Understand → Validate → Run → Verify

**This workflow is the core of this skill. ALWAYS follow it. NEVER skip steps. NEVER run an Actor without completing the preceding steps.**

### STOP — Determine workflow scope first

**ALWAYS determine scope BEFORE doing anything else.** Wrong scope = wasted tokens and money.

- **Docs-only** (user asks about Crawlee, Apify platform, configuration) → skip to [Docs-only Workflow](#docs-only-workflow) section. **STOP — do NOT search for Actors.**
- **Find-only** (user wants actor recommendation, comparison, schema inspection) → do Steps 1-2, present comparison, **STOP. Do NOT proceed to Steps 3-7 unless user explicitly says "run it".**
- **Full pipeline** (user wants data extracted/scraped) → follow all 7 steps with mandatory gates

**Mandatory gates — these are HARD STOPS, not suggestions:**

```
- [ ] Step 1: Search for Actors (ALWAYS at least 2 searches: broad then narrow)
- [ ] Step 2: Fetch details + input schema (ALWAYS read README)
- [ ] GATE: STOP — show user the Actor choice + planned input. Wait for confirmation.
- [ ] Step 3: Build input from schema + README
- [ ] Step 4: Validate inputs (ALWAYS challenge assumptions about identifiers)
- [ ] GATE: STOP — if ANY input value is guessed, tell user. Do NOT run with unverified inputs.
- [ ] Step 5: Run the Actor (test run with limit=1 FIRST when uncertain)
- [ ] Step 6: ALWAYS verify results — zero results = wrong input, not "no data"
- [ ] Step 7: Get full results and present
```

### Step 1: Match use case and search for Actors

0. If no use case matches user's intent, build it with user, ask questions, clarify, find the right actors together.
1. Check the [Use Cases](#use-cases) table — prefer suggested Actors
2. Search Apify Store for latest options:

```bash
# First search: broad (platform name only)
mcpc @apify tools-call search-actors keywords:="instagram"
# Second search: narrow if needed (platform + data type)
mcpc @apify tools-call search-actors keywords:="instagram profile"
```

- **ALWAYS do at least two searches**: first broad (just platform name), then narrow (platform + data type) if the broad search returns too many irrelevant results
- Prefer broad keywords first — `"TikTok"` before `"TikTok posts"`
- Use keywords from the use-case file when available
- Avoid generic terms like "crawler", "data extraction"
- Pick all valid candidates for comparison
- Actor identifier in results is `fullName` field (e.g., `apify/website-content-crawler`)
- Parameters: `keywords` (string), `limit` (1–100, default 10), `offset` (default 0), `category` (string, filter by category — leave empty for all)

### Step 2: Fetch Actor details and input schema

`fetch-actor-details` returns description, README, all stats, AND the full input schema in one call. Parameter is `actor` (not actorId):

```bash
mcpc @apify tools-call fetch-actor-details actor:="apify/website-content-crawler"
```

Use `output` parameter to select specific sections and save tokens:

```bash
mcpc @apify tools-call fetch-actor-details actor:="apify/website-content-crawler" output:='{"inputSchema": true}'
mcpc @apify tools-call fetch-actor-details actor:="apify/website-content-crawler" output:='{"readme": true}'
```

Valid `output` fields (`additionalProperties: false` — typo = error `-32602`):

| Field | Returns |
|-------|---------|
| `description` | Actor description text |
| `stats` | `actorInfo.stats`: `totalUsers`, `monthlyUsers`, `successRate`, `bookmarks` |
| `pricing` | `actorInfo.pricing`: `model` (FREE/PAY_PER_EVENT/PRICE_PER_DATASET_ITEM/FLAT_PRICE_PER_MONTH), `isFree`, and for PAY_PER_EVENT: `events[]` with per-tier costs (FREE/BRONZE/SILVER/GOLD/PLATINUM/DIAMOND tiers) |
| `rating` | `actorInfo.rating`: `average` (out of 5), `count` |
| `metadata` | `actorInfo`: `developer`, `categories`, `modifiedAt`, `isDeprecated` |
| `inputSchema` | Full JSON Schema of input parameters |
| `readme` | README summary (or full if no summary) |
| `outputSchema` | Inferred output schema from recent runs |
| `mcpTools` | MCP tools list (only for MCP server Actors) |

Default (no `output`): all fields **except** `mcpTools`. With `output`: **only `true` fields return data** — `false` flags are ignored (same as omitting them). `output:='{}'` returns the default.

Always-present fields in `actorInfo` (no flag needed): `title`, `url`, `fullName`, `pictureUrl`.

For structured access, use `--json` + `jq`:

```bash
# Success rate
mcpc @apify tools-call fetch-actor-details actor:="apify/website-content-crawler" output:='{"stats": true}' --json | jq '.structuredContent.actorInfo.stats.successRate'

# Stats + rating together
mcpc @apify tools-call fetch-actor-details actor:="apify/website-content-crawler" output:='{"stats": true, "rating": true}' --json | jq '.structuredContent.actorInfo | {stats, rating}'
```

- Verify the Actor does what's needed (description, README)
- Check `isDeprecated` — never use deprecated Actors
- Read the input schema — it contains `properties`, `required`, `prefill` for each field
- Compare candidates — see "How to Compare Actors" below
- Interpret schema fields — see "How to Read Input Schema" below
- **Read the README thoroughly** — not just for technical parameters, but for **methodology advice**: how to find correct input values, what formats the target service uses, edge cases, limitations, and tips the author provides. The README often explains how the service being scraped works and what values actually make sense.

### Step 3: Build input and confirm with user

The schema alone is not enough — ALWAYS cross-reference with the README to understand what values the Actor actually expects:

1. **Identify required fields** from the schema's `required` array
2. **Read each field's `description`** — it often specifies exact formats, allowed values, or constraints not captured by `type` alone (e.g., "URL must include protocol", "comma-separated list", "ISO country code")
3. **Check `prefill` values** — use as starting point, adapt to user's actual needs
4. **When the schema description is unclear**, fetch the README (`output:='{"readme": true}'`) — it typically has usage examples with realistic inputs
5. **STOP — Show the user** the planned input JSON with explanations. **NEVER run without user confirmation.** Before displaying: if any field name matches `/(cookie|token|password|secret|auth|key)/i`, replace the value with `<REDACTED>` and confirm only the type/format needed — never display credentials in chat. Wait for explicit approval.

### Step 4: Validate inputs against the target service

Before running the Actor, critically evaluate whether your input values make sense **in the context of how the target service actually works**:

1. **Think about what the service expects** — Does the service use usernames, display names, or IDs? Are the identifiers case-sensitive? Does the service have a specific URL format? Would the service even have the data you're looking for?
2. **Challenge your assumptions** — If you're using identifiers (usernames, URLs, search terms) that you constructed yourself rather than obtained from a reliable source, ask: how confident am I that these are correct? A person named "John Smith" might use "jsmith", "johnsmith123", or "totally_unrelated_handle" on any given platform.
3. **Verify identifiers when uncertain** — If input values are based on guesswork (e.g., assuming a social media username from a person's real name), verify them before running a paid scrape. Options: search the web, check the person's known profiles for cross-links, or run a minimal test (1 result) to confirm the identifier resolves to the expected entity.
4. **Consider the service's data model** — Does the service distinguish between profiles, pages, and groups? Between posts, stories, and reels? Make sure you're targeting the right entity type.

If you cannot confidently verify an input value, tell the user what you're uncertain about and suggest how to resolve it before spending money on a scrape.

### Step 5: Run the Actor — NEVER without Steps 3-4 completed

```bash
mcpc @apify tools-call call-actor actor:="apify/website-content-crawler" \
  input:='{"startUrls":[{"url":"https://example.com"}],"proxyConfiguration":{"useApifyProxy":true}}' \
  callOptions:='{"timeout": 300}'
```

`callOptions` accepts `timeout` (seconds, 0 = infinite) and `memory` (MB, power of 2, 128–32768).

Use `previewOutput:=false` to skip inline items and save tokens — then fetch selectively with `get-actor-output`:

```bash
mcpc @apify tools-call call-actor actor:="<actor>" input:='...' previewOutput:=false --json \
  | jq '.structuredContent | {runId, datasetId, itemCount}'
```

**Sync response** (`structuredContent`): `runId`, `datasetId`, `itemCount`, `items[]`, `instructions`.
**Async response** (`async:=true`): `runId`, `actorName`, `status`, `startedAt`, `input`. No cost data — `_meta` is only returned for sync runs.
**Cost tracking** (sync only): `--json` response has `_meta` at the **JSON top-level** (not inside `structuredContent`): `jq '._meta.usageTotalUsd'` returns total run cost in USD.

On input validation error, go back to Step 3.

By default, `call-actor` runs synchronously (waits for completion). For long-running Actors, use `async:=true` — poll status with `get-actor-run`.

#### Async resume

Save `runId` immediately when using `async:=true` — sessions can reset mid-run:

```bash
mcpc ... async:=true --json | jq -r '.structuredContent.runId' | tee run.id
mcpc @apify tools-call get-actor-run runId:="$(cat run.id)"  # resume/check
```

**Cost tracking**: auto-logged to `~/.apify-costs.log` per `call-actor --json` run. Check with `/apify-status`.

### Step 6: ALWAYS verify results — never skip

Before presenting results to the user or fetching the full dataset, ALWAYS do a sanity check on the initial output:

1. **Check result count** — Does the number of results match expectations? Zero results likely means wrong input values, not "no data exists". A suspiciously low or high count warrants investigation.
2. **Spot-check content** — Look at a few result items. Do they correspond to what was requested? If you scraped a person's profile, does the bio/description match who they actually are? If you scraped a business, is it in the right location/industry?
3. **Cross-reference with general knowledge** — You know things about the world. If a famous athlete's profile shows 50 followers, that's almost certainly the wrong account. If a major company's page has no posts, something is off. Use common sense.
4. **Flag anomalies to the user** — If anything looks wrong, say so before proceeding. It's cheaper to re-run with corrected inputs than to deliver bad data.

#### Minimal test run (1 result)

When uncertain about inputs, run a cheap test first — get 1 result and verify before full scrape:

1. Find the Actor's max results field (name varies per Actor):

```bash
mcpc @apify tools-call fetch-actor-details actor:="<actor>" output:='{"inputSchema": true}' --json \
  | jq '[.structuredContent.inputSchema.properties | to_entries[] | select(.key | test("max|limit"; "i")) | {key, title: .value.title}]'
```

2. Run with limit 1 + short timeout as safety net:

```bash
mcpc @apify tools-call call-actor actor:="<actor>" \
  input:='{"<maxResultsField>": 1, ...}' \
  callOptions:='{"timeout": 120}'
```

3. Verify the single result makes sense before scaling up.

`callOptions.timeout` (seconds) auto-aborts the run if the Actor ignores its limit. There is no `abort` tool in mcpc — for manual abort use Apify REST API: `POST https://api.apify.com/v2/actor-runs/{runId}/abort?token={APIFY_TOKEN}`.

### Step 7: Get results

**NEVER load full datasets into context.** Large responses flood the context and degrade performance. Always follow the three-step pattern below.

#### 7a. Check total count first

```bash
mcpc @apify tools-call get-actor-output datasetId:="<id>" limit:=1 --json \
  | jq '.structuredContent | {totalItemCount, fields: (.items[0] | keys)}'
```

This tells you: how many items exist and what fields are available — without loading any real data.

#### 7b. Sample to verify data quality

Before saving or presenting, spot-check a few items:

```bash
mcpc @apify tools-call get-actor-output datasetId:="<id>" limit:=5
```

Read the output. Verify the data matches the expected entity (see Step 6). If it looks wrong, re-run from Step 3 — don't save bad data.

#### 7c. Save to file — don't load into context

Once the sample looks correct, save the full dataset to a local file. The `>` redirect sends data to disk; nothing enters the context.

```bash
# All items, all fields
mcpc @apify tools-call get-actor-output datasetId:="<id>" --json \
  | jq '.structuredContent.items' > results.json

# Selected fields only (reduces file size)
mcpc @apify tools-call get-actor-output datasetId:="<id>" \
  fields:="title,url,price" --json \
  | jq '.structuredContent.items' > results.json

# Large dataset — paginate into one file
for offset in 0 100 200; do
  mcpc @apify tools-call get-actor-output datasetId:="<id>" \
    limit:=100 offset:=$offset --json \
    | jq '.structuredContent.items[]' >> results.jsonl
done
```

Then confirm the save and summarize to the user:

```bash
jq 'length, .[0:2]' results.json   # item count + 2 examples
```
**Other formats** — `get-actor-output` is JSON only. For CSV/Excel/XML download directly: `https://api.apify.com/v2/datasets/{datasetId}/items?format=csv&clean=true&token=${APIFY_TOKEN}` (formats: `csv`, `xlsx`, `jsonl`, `xml`).

**`fields` gotcha**: dot notation flattens keys — `fields:="crawl.httpStatusCode"` returns `{"crawl.httpStatusCode": 200}`, not `{"crawl": {"httpStatusCode": 200}}`. In jq use `."crawl.httpStatusCode"` (quoted).

**`get-actor-output` response** (`structuredContent`): `datasetId`, `items[]`, `itemCount`, `totalItemCount`, `offset`, `limit`.

If the run was async or is still in progress, check status first:

```bash
mcpc @apify tools-call get-actor-run runId:="<runId>" --json | jq '.structuredContent | {status, datasetId: .dataset.datasetId, itemCount: .dataset.itemCount}'
```

**`get-actor-run` response** (`structuredContent`): `runId`, `actorName`, `status`, `startedAt`, `finishedAt`, `stats` (runTimeSecs, computeUnits, memMaxBytes, ...), `dataset` (datasetId, itemCount, schema, previewItems[]).

Check the use-case file for suggested follow-up workflows after presenting results.

### Error handling — Feedback Loop

1. Read the error message
2. Non-obvious fixes:
   - "must have required property 'actor'" → parameter is `actor`, not `actorId`
   - "Authentication required" / 401 → OAuth expired, re-run `mcpc mcp.apify.com login` then `mcpc mcp.apify.com connect @apify`
   - "usage hard limit exceeded" → user's Apify plan limit, inform the user
3. Retry from the appropriate step
4. After 2 failed retries, tell the user what's happening and ask for guidance
5. **If the problem is structural** (skill docs wrong, mcpc behavior changed, Actor from use-case file missing/deprecated, response structure differs from documented) — this is a skill bug, not a user error. Read [issue-reporting.md](references/issue-reporting.md) and offer to report it. ALWAYS ask user for consent before creating an issue.

## How to Read Input Schema

The input schema from `fetch-actor-details` is standard JSON Schema, but with Apify-specific conventions:

- **`prefill`** — example values provided by the Actor author. Use as starting point for building input.
- **`sectionCaption` / `sectionDescription`** — group related fields; read these to understand field purpose in context.
- **`editor`** — hints at expected format (e.g., `"requestListSources"` means the field expects `[{"url": "..."}]` objects, not plain strings).

### Apify input gotchas

**startUrls pattern** — most scrapers use `startUrls`, which is NOT an array of strings. It's an array of request objects:
```json
// Correct
{"startUrls": [{"url": "https://example.com"}, {"url": "https://example.com/page2"}]}

// Wrong — plain strings don't work
{"startUrls": ["https://example.com"]}
```

**Field naming** — Actors don't follow a single convention. Common variants:
- Max results: `maxItems`, `maxResults`, `maxCrawlPages`, `resultsLimit` — always check the schema
- Search input: `searchTerms`, `queries`, `keywords`, `search` — varies per Actor

**Social handles** — typically without `@`: `"apify"` not `"@apify"`

**Proxy** — when needed: `"proxyConfiguration": {"useApifyProxy": true}`

## How to Compare Actors

`search-actors` already returns `stats`, `rating`, `pricing`, `isDeprecated`, `developer` per actor — enough for comparison without extra calls. Use `fetch-actor-details` only when you need `inputSchema`, `readme`, `outputSchema`, or `modifiedAt`.

**Decision order** (eliminate top-down):

1. `isDeprecated` = `true` → eliminate immediately
2. `successRate` < 80% → red flag, skip unless only option
3. `pricing.model` → prefer FREE > PAY_PER_EVENT > PRICE_PER_DATASET_ITEM (cheapest that works)
4. `totalUsers` > 100 → battle-tested, safe bet
5. Author `apify/` → official, often more reliable
6. `modifiedAt` more recent → maintained (only via `fetch-actor-details` with `metadata` flag)

**Selection rule**: Pick the cheapest Actor that meets functional requirements.

**Fallback chain**: If the chosen Actor fails (error, bad data, deprecated), try the next candidate. Don't ask the user before trying an alternative — just report what happened. Only stop and ask if all candidates fail.

## Docs-only Workflow

For Apify/Crawlee documentation questions without needing Actors:

```bash
mcpc @apify-docs tools-call search-apify-docs docSource:="apify" query:="proxy configuration" limit:=5
mcpc @apify-docs tools-call fetch-apify-docs url:="https://docs.apify.com/platform/proxy"
```

- `docSource`: `"apify"`, `"crawlee-js"`, or `"crawlee-py"`
- `query`: keywords only, not sentences
- `limit`: 1–20 (default 5), `offset`: for pagination (default 0)

## Filtering mcpc Output

mcpc responses can be large. Use these techniques to get only what you need:

### Server-side filtering (preferred — saves tokens)

| Situation | Technique |
|-----------|-----------|
| Need only input schema, not README/stats | `fetch-actor-details ... output:='{"inputSchema": true}'` |
| Need only README | `fetch-actor-details ... output:='{"readme": true}'` |
| Dataset has many fields, need only some | `get-actor-output ... fields:="title,url,price"` |
| Large dataset, need a sample | `get-actor-output ... limit:=10` |
| Paginate through results | `get-actor-output ... offset:=10 limit:=10` |

### Client-side filtering (`--json` + `jq`)

Use when server-side filtering isn't enough or for scripting:

```bash
# Extract actor names from search
mcpc @apify tools-call search-actors keywords:="instagram scraper" --json | \
  jq -r '.structuredContent.actors[].fullName'

# Quick comparison from search (no extra API calls)
mcpc @apify tools-call search-actors keywords:="instagram scraper" limit:=5 --json | \
  jq '[.structuredContent.actors[] | {fullName, successRate: .stats.successRate, rating: .rating.average, users: .stats.totalUsers, pricing: .pricing.model}]'

# Deep comparison with README (only when needed)
mcpc @apify tools-call search-actors keywords:="instagram scraper" --json | \
  jq -r '.structuredContent.actors[].fullName' | while read -r name; do
    mcpc @apify tools-call fetch-actor-details actor:="$name" output:='{"stats": true, "rating": true, "metadata": true, "pricing": true}' --json | \
      jq '.structuredContent'
  done
```

JSON responses wrap tool output in `{content, structuredContent}` — always access data via `.structuredContent`.

### When to filter

- **Step 2 (fetch details)**: First call without `output` to get overview. Follow-up calls with `output:='{"inputSchema": true}'` or `output:='{"readme": true}'` to dig deeper without re-fetching everything.
- **Step 7 (get results)**: Always use `fields` if you know which columns matter. Use `limit` for first peek, then fetch more if needed.
- **Comparing actors**: Use `--json` + `jq` to extract stats programmatically when comparing 3+ candidates.

## Recurring runs and scheduling

This skill handles one-shot runs. For recurring/scheduled scraping:

1. **Apify Schedules**: Guide user to set up schedules in Apify Console (UI) — Actors can be scheduled with cron syntax
2. **Webhook integration**: Actors support webhooks on completion — useful for pipelines
3. **Delta detection**: For "only new data" use cases, the agent must implement diffing — Apify doesn't track deltas automatically. Recommend: store previous run's dataset ID, compare on next run.

This skill does NOT create schedules programmatically — that requires Apify API, not mcpc.
