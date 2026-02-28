---
name: manifold-markets-skill
description: "Manifold Markets prediction market API guide. Use when: (1) Fetching/searching markets or market data, (2) Placing bets, limit orders, or multi-bets, (3) Selling shares or canceling orders, (4) Analyzing positions, portfolios, or profit, (5) Building trading bots, (6) WebSocket real-time updates, (7) Bulk data via Supabase, (8) Creating/editing/resolving markets, (9) Comments, reactions, follows, managrams, or DMs, (10) Querying transactions or bet history, (11) AMM math/simulation."
---

# Manifold Markets API

## Quick Reference

| Item | Value |
|------|-------|
| Base URL | `https://api.manifold.markets/v0` |
| Root URL | `https://api.manifold.markets` (some endpoints) |
| Auth Header | `Authorization: Key {api_key}` |
| Rate Limit | 500 req/min per IP |
| Timestamps | JavaScript milliseconds (NOT seconds) |

---

## Authentication

**Get your API key at:** https://manifold.markets/profile

### For scripts (Python)

```python
import os
import requests

API_KEY = os.getenv("MANIFOLD_API_KEY")
BASE = "https://api.manifold.markets/v0"
headers = {"Authorization": f"Key {API_KEY}"}

r = requests.get(f"{BASE}/me", headers=headers)
```

### For shell commands (curl)

If an authenticated request is needed, ask the user to export their API key first:

```bash
export MANIFOLD_API_KEY="your-key-here"
```

The user can get their key from https://manifold.markets/profile or their `.env` file.

Then use in curl commands:
```bash
curl -H "Authorization: Key $MANIFOLD_API_KEY" https://api.manifold.markets/v0/me
```

The export lasts for the terminal session (until the window is closed).

### Which endpoints need auth?

- **No auth needed:** Reading public markets, users, bets, comments
- **Auth required:** Placing bets, posting comments, creating markets, accessing own portfolio/notifications

---

## Common Operations

### Get a Market

```python
r = requests.get(f"{BASE}/market/{market_id}")
r = requests.get(f"{BASE}/slug/{market_slug}")
```

### Search Markets

```python
r = requests.get(f"{BASE}/search-markets", params={
    "term": "AI",
    "filter": "open",      # all, open, closed, resolved, closing-this-month, closing-next-month
    "sort": "liquidity",   # See sort options below
    "limit": 50,           # Max 100
})

# Valid sort options (from schema.ts):
# score (default), newest, daily-score, freshness-score, 24-hour-vol,
# most-popular, liquidity, subsidy, last-updated, close-date,
# start-time, resolve-date, random, bounty-amount, prob-descending, prob-ascending
#
# NOTE: There is NO "volume" sort. Use Supabase for all-time volume ranking.
```

### Place a Bet

```python
# Market order
r = requests.post(f"{BASE}/bet", headers=headers, json={
    "contractId": market_id,
    "amount": 100,
    "outcome": "YES",
})

# Limit order (IOC - expires quickly)
r = requests.post(f"{BASE}/bet", headers=headers, json={
    "contractId": market_id,
    "amount": 100,
    "outcome": "YES",
    "limitProb": 0.60,
    "expiresMillisAfter": 10,
})

# Multiple choice (requires answerId)
r = requests.post(f"{BASE}/bet", headers=headers, json={
    "contractId": market_id,
    "answerId": answer_id,
    "amount": 100,
    "outcome": "YES",
})
```

### Sell Shares

```python
r = requests.post(f"{BASE}/market/{market_id}/sell", headers=headers, json={
    "outcome": "YES",
    "shares": 50,          # Optional, sells all if omitted
    "answerId": answer_id, # Required for MC
})
```

### Get User & Positions

```python
r = requests.get(f"{BASE}/user/{username}")
r = requests.get(f"{BASE}/market/{market_id}/positions", params={"userId": user_id})
```

---

## Market Types

| Type | Mechanism | Description |
|------|-----------|-------------|
| `BINARY` | `cpmm-1` | YES/NO, variable `p` parameter |
| `MULTIPLE_CHOICE` | `cpmm-multi-1` | Multiple answers, always `p=0.5` |
| `PSEUDO_NUMERIC` | `cpmm-1` | Numeric range mapped to 0-1 |
| `POLL` | `none` | Non-tradeable voting |
| `BOUNTIED_QUESTION` | `none` | Bounty for best answer |

### Linked vs Unlinked Multi-Choice

**`shouldAnswersSumToOne`** determines behavior:
- `false` → Independent answers (each is its own YES/NO)
- `true` → Mutually exclusive (probabilities sum to 1)

---

## Market State

A market can be bet on only if:
```python
trading_allowed = (
    not market.get("isResolved") and
    (not market.get("closeTime") or market["closeTime"] > time.time() * 1000) and
    market.get("mechanism") != "none" and
    (answer is None or not answer.get("resolution"))  # For MC
)
```

---

## AMM Probability Formula

```python
# General formula
prob = pool_no**(1-p) / (pool_yes**p + pool_no**(1-p))

# Multi-choice (always p=0.5)
prob = pool_no / (pool_yes + pool_no)
```

---

## Common Gotchas

| Issue | Solution |
|-------|----------|
| Timestamps are milliseconds | `close_time_sec = market["closeTime"] / 1000` |
| Query param booleans are strings | `params={"perAnswer": "true"}` in URL params |
| Market descriptions: markdown images stripped | Use `descriptionHtml` or TipTap JSON for images |
| MC markets need `answerId` | Always include for multi-choice bets |
| Search excludes fields | Fetch full market if you need `answers`, `description` |
| Answer pools are nested | `answer["pool"]["YES"]`, `answer["pool"]["NO"]` |
| Root vs /v0/ paths | Some endpoints at root (no `/v0/`), see reference files |
| No all-time volume sort | REST API has no `sort=volume`. Use Supabase (see recipes below) |
| `profitByTopic` is a list | Not a dict. Iterate with `for item in profitByTopic` |
| `winRate` is a percentage | Value of 62.5 means 62.5%, not 0.625 |
| `maxDrawdown` is a percentage | Value of 25 means 25%, not 0.25 |
| Supabase timestamps need ISO | Use `datetime.now(timezone.utc).isoformat()`, not milliseconds |

---

## Common Recipes

### Top Markets by All-Time Volume

The REST API cannot sort by all-time volume. Use Supabase instead:

```python
from supabase import create_client

SUPABASE_URL = "https://pxidrgkatumlvfqaxcll.supabase.co"
SUPABASE_ANON_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InB4aWRyZ2thdHVtbHZmcWF4Y2xsIiwicm9sZSI6ImFub24iLCJpYXQiOjE2Njg5OTUzOTgsImV4cCI6MTk4NDU3MTM5OH0.d_yYtASLzAoIIGdXUBIgRAGLBnNow7JG2SoaNMQ8ySg"

sb = create_client(SUPABASE_URL, SUPABASE_ANON_KEY)

# Top 10 by all-time volume (volume is in data->volume JSON path)
result = (
    sb.table("contracts")
    .select("id, question, slug, visibility, data")
    .eq("visibility", "public")
    .order("data->volume", desc=True)
    .limit(10)
    .execute()
)

for m in result.data:
    vol = m["data"].get("volume", 0)
    creator = m["data"].get("creatorUsername", "")
    print(f"{int(vol):,} M - {m['question']}")
    print(f"  https://manifold.markets/{creator}/{m['slug']}")
```

### Top Markets by 24-Hour Volume

Use the REST API with `sort=24-hour-vol`:

```python
r = requests.get(f"{BASE}/search-markets", params={
    "sort": "24-hour-vol",
    "filter": "open",
    "limit": 10,
})
```

---

## Error Responses

```python
{"message": "Contract not found"}
{"message": "Insufficient balance"}
{"message": "Market is closed"}
```

Errors return HTTP 4xx/5xx with a `message` field.

---

## Reference Files

Organized by domain:

| File | Contents |
|------|----------|
| [markets.md](references/markets.md) | Create, fetch, search, edit, resolve markets, groups/topics |
| [betting.md](references/betting.md) | Place bets, limit orders, multi-bet, sell, cancel |
| [users.md](references/users.md) | Profiles, positions, portfolios, activity, loans, performance stats |
| [social.md](references/social.md) | Comments, reactions, follows, manalinks, DMs, transactions |
| [data-structures.md](references/data-structures.md) | Response schemas, **endpoint comparison** (which endpoint returns what), sort options |
| [amm.md](references/amm.md) | AMM math formulas, bet sizing, arbitrage |
| [websocket.md](references/websocket.md) | Real-time updates via WebSocket |
| [supabase.md](references/supabase.md) | Bulk data access, **sorting by volume** and other JSON fields |
| [endpoint-audit.md](references/endpoint-audit.md) | Complete endpoint inventory with documentation status |

---

## Parallel Fetching

```python
from concurrent.futures import ThreadPoolExecutor

def fetch_markets_parallel(market_ids, max_workers=10):
    def fetch_one(mid):
        return requests.get(f"{BASE}/market/{mid}").json()

    with ThreadPoolExecutor(max_workers=max_workers) as ex:
        return list(ex.map(fetch_one, market_ids))

# Batch probability fetch (100x faster, up to 100 markets)
r = requests.get(f"{BASE}/market-probs",
    params=[("ids[]", mid) for mid in market_ids[:100]])

# Batch market fetch (root path) - note bracket notation
r = requests.get("https://api.manifold.markets/markets-by-ids",
    params=[("ids[]", mid) for mid in market_ids])
```

---

## Caching Bulk Data

For long-running operations (scanning users, bulk fetches, etc.), cache to `~/.cache/manifold-markets-skill/` to avoid polluting the user's repo. Use checkpointing with `processed_ids` for resumable operations.

---

## Admin / Internal Endpoints

The following endpoints exist but require special privileges or are for internal use:

- **Admin**: `admin-*`, `super-ban-user`, `recover-user`, `anonymize-user`, `toggle-system-trading-status`
- **Moderation**: `get-mod-reports`, `update-mod-report`, `dismiss-user-report`
- **Identity verification (GIDX)**: `*-gidx` endpoints for KYC/sweepstakes compliance
- **Analytics tracking**: `record-contract-view`, `record-comment-view`, `record-contract-interaction`

Don't use these unless you have appropriate permissions.

---

## Official Resources

- **API Docs**: https://docs.manifold.markets/api
- **API Schema**: https://github.com/manifoldmarkets/manifold/blob/main/common/src/api/schema.ts
- **Market Types**: https://github.com/manifoldmarkets/manifold/blob/main/common/src/contract.ts
