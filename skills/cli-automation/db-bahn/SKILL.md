---
name: db-bahn
description: Query Deutsche Bahn train connections and prices.
homepage: https://github.com/timkrase/homebrew-tap
metadata: {"clawdis":{"emoji":"🚂","requires":{"bins":["dbrest"]},"install":[{"id":"brew","kind":"brew","formula":"timkrase/tap/dbrest","bins":["dbrest"],"label":"Install dbrest (brew)"}]}}
---

# DB Bahn Skill 🚂

Query train connections and prices via the Deutsche Bahn API.

## CLI: dbrest

**Binary:** `dbrest` (Homebrew: `timkrase/tap/dbrest`)

### Find stations
```bash
dbrest locations "station name"
```

Output: Tab-separated list with ID, Name, Type.

### Search connections
```bash
dbrest journeys --from <ID> --to <ID> [--departure ISO8601] [--results N]
```

**Important:** `--from` and `--to` require station IDs (IBNR), not names!

**Workflow:**
```bash
# 1. Find station IDs
dbrest locations "Köln Hbf"        # → 8000207
dbrest locations "München Hbf"     # → 8000261

# 2. Search journeys
dbrest journeys --from 8000207 --to 8000261 --results 5
```

### With prices (JSON)
```bash
dbrest --json journeys --from 8000207 --to 8000261 --results 5
```

Prices are at `journeys[].price.amount` and `journeys[].price.currency`.

### Live departures
```bash
dbrest departures --stop <ID> --results 10
```

### Live arrivals
```bash
dbrest arrivals --stop <ID> --results 10
```

## Notes

- **Prices only for long-distance trains** — S-Bahn/regional trains often have no price
- **International stations** (CH, AT, etc.) work ✅
- **Global flags before command:** `dbrest --json journeys ...`
