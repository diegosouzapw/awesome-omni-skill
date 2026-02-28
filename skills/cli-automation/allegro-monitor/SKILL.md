---
name: allegro-monitor
description: Monitor Allegro.pl prices and get alerts when items drop below your threshold.
homepage: https://github.com/nexty5870/clawdis-allegro-skill
metadata: {"clawdis":{"emoji":"🛒","requires":{"tools":["clawdis_browser"]}}}
---

# Allegro Price Monitor

Monitor Allegro.pl listings for specific items and track price changes.

## Overview

This skill uses browser automation to scrape Allegro search results and track prices over time. It's designed for hunting deals on specific items (like GPUs).

Since Allegro's API registration is currently closed, this skill uses `clawdis_browser` to:

1. Open Allegro search page with the query
2. Handle cookie consent if needed
3. Sort by price (lowest first)
4. Extract listing data: title, price, condition, seller, URL
5. Filter and return results

## Agent Instructions

When asked to check Allegro prices:

1. Use `clawdis_browser` to open: `https://allegro.pl/listing?string={query}&order=p`
   - `order=p` sorts by price ascending
   
2. If you see a cookie consent dialog ("Dbamy o Twoją prywatność"), click "Ok, zgadzam się"

3. Take a snapshot and extract listings from `article` elements:
   - **Title**: `heading[level=2]` text
   - **Price**: Look for `XXX,XX zł` pattern
   - **Condition**: After `Stan:` label (Nowy, Używany, Uszkodzony, etc.)
   - **URL**: Link in the heading
   - **Seller**: "Super Sprzedawca", "Firma", or "Prywatny sprzedawca"

4. Filter results:
   - Skip `Uszkodzony` (damaged) items unless specifically requested
   - Apply max price filter if specified
   - Apply condition filter if specified

5. Report findings with: title, price, condition, and URL

## Condition Reference

| Polish | English | Working? |
|--------|---------|----------|
| Nowy | New | ✅ |
| Jak nowe | Like new | ✅ |
| Używany | Used | ✅ |
| Powystawowy | Ex-display | ✅ |
| Po zwrocie | After return | ⚠️ Check |
| Uszkodzony | Damaged | ❌ |

## Example Queries

- "Check Allegro for RTX 3090 under 3000 PLN"
- "Find used PS5 controllers on Allegro"
- "Monitor Allegro for iPhone 15 Pro deals"

## Cron Setup

For automated monitoring, create a cron job:

```json
{
  "name": "Item Price Monitor",
  "schedule": { "kind": "cron", "expr": "0 9,14,19 * * *", "tz": "Europe/Warsaw" },
  "sessionTarget": "isolated",
  "wakeMode": "now",
  "payload": {
    "kind": "agentTurn",
    "message": "Check Allegro for [ITEM]. Only working condition. Alert if under [PRICE] PLN.",
    "deliver": false
  }
}
```

## Notes

- Allegro is Poland's largest marketplace (like eBay for Poland)
- Prices are in PLN (Polish Złoty)
- Best deals often come from "Super Sprzedawca" (Super Seller) accounts
- "Allegro Lokalnie" listings are local pickup / classifieds style
