---
name: manifold-analysis
description: Analyze Manifold Markets prediction market data. Use when processing HTML exports or trade history from manifold.markets to create visualizations of trading volume, trader leaderboards, probability movements, and market dynamics. Triggers on requests involving Manifold Markets data, prediction market analysis, or when user uploads Manifold HTML files.
---

# Manifold Markets Analysis

Analyze prediction market data from Manifold Markets to create interactive visualizations and trader analytics.

## Overview

Manifold Markets is a play-money prediction market platform. Key concepts:
- **Mana (Ṁ)** - Play-money currency (not convertible to cash, ~Ṁ100 = $1 purchase price)
- **Markets** - Questions with multiple answer buckets (e.g., "$5-10B", ">$25B")
- **Trading** - Users buy YES/NO shares on answers; prices reflect probability

## Data Sources

### Manifold API (Preferred)
Fetch data directly from the Manifold Markets API:

1. **Find market ID** via search:
```bash
curl "https://api.manifold.markets/v0/search-markets?term=your+search+term"
```

2. **Fetch all bets** with pagination:
```bash
curl "https://api.manifold.markets/v0/bets?contractId=MARKET_ID&limit=1000"
# Use &before=LAST_BET_ID for pagination
```

3. **Resolve usernames** for top traders:
```bash
curl "https://api.manifold.markets/v0/user/by-id/USER_ID"
```

**Rate Limiting**: Be conservative - 1 second between paginated requests, longer for user lookups. Skip bulk user lookups if possible.

Use `scripts/fetch_market_data.py` for automated fetching:
```bash
python3 scripts/fetch_market_data.py --market-id MARKET_ID --output all > market_data.json
```

### HTML Export
Users may upload saved HTML from manifold.markets pages. Extract data from:
- Market title and metadata in page header
- Trade history in comments/activity sections (look for patterns like "bought Ṁ50 of YES")
- Current probabilities displayed for each answer

### Trade History Text
Users may paste trade history directly. Common format:
```
Username,action,amount,answer,outcome,time_ago
JoshYou,bought,350,>$25B,YES,1y
Bayesian,sold,100,$5-10B,NO,3mo
```

Time formats: `23d` (days), `1mo`/`3mo` (months), `1y` (year ago)

## Analysis Workflow

### 1. Parse Trade Data
Use `scripts/parse_trades.py` to extract trades from text:
```bash
python3 scripts/parse_trades.py < trades.txt > trades.json
```

### 2. Aggregate by Trader
For each trader compute:
- Total volume (sum of all trade amounts)
- Trade count
- Buy/sell ratio
- YES vs NO volume breakdown
- Top answer buckets traded

### 3. Aggregate by Time
Convert relative timestamps to approximate dates:
- Reference: current date or market close date
- Map "1y" → ~12 months ago, "3mo" → ~3 months ago, etc.
- Group by month for time series

### 4. Create Visualization
Build an HTML visualization with Chart.js (preferred for reliability):

```html
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
```

Include:
- Cumulative stacked area chart by answer over time
- Trader leaderboard table with volume, trades, YES/NO breakdown
- Answer breakdown legend with colors
- Stats cards showing probability, total volume, trades, unique traders

See `references/visualization_template.md` for React/Recharts approach (less reliable CDN loading).

**Example output**: `iran_market_viz_chartjs.html` - full standalone visualization

## Color Scheme for Answers

### Binary Markets (YES/NO)
```javascript
const colors = {
  YES: '#10b981',  // Green - teal
  NO: '#ef4444'    // Red
};
```

### Multi-Answer Markets
Use consistent colors across visualizations:
```javascript
const colors = {
  "<$5B": "#99DDFF",
  "$5-10B": "#FFDD99",
  "$10.1-12.5B": "#FFAABB",
  "$12.6-15B": "#77F299",
  "$15.1-17.5B": "#CD46EA",
  "$17.6-20B": "#F23542",
  "$20.1-25B": "#FF8C00",
  ">$25B": "#44BB99"
};
```

Adapt color keys to match actual answer labels in the market.

## Key Metrics to Surface

### Market Level
- Total volume traded
- Number of unique traders
- Peak trading month
- Current leading answer and probability

### Trader Level
- Rank by total volume ("whales")
- Rank by trade count ("most active")
- YES vs NO ratio (bullish/bearish tendency)
- Top 2-3 answers traded per user

### Insights to Highlight
- **Biggest whale** - Highest total volume
- **Most active** - Highest trade count
- **Top bull** - Highest % YES volume
- **Top bear** - Highest % NO volume

## Context Notes

When presenting analysis, note:
1. Mana is play money with no cash value
2. Large positions may represent accumulated winnings, not money invested
3. New users get Ṁ1,000 free; active traders earn daily bonuses
4. Someone with Ṁ40k may have spent $0-400 actual dollars
