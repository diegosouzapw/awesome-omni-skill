---
name: oracle-sentinel
version: 2.0.0
description: Autonomous AI agent for Polymarket prediction intelligence. Real-time prices, deep news research, whale trade alerts, dual-model AI analysis.
homepage: https://oraclesentinel.xyz
metadata: {"category":"ai-agents","api_base":"https://oraclesentinel.xyz/api","token":"$OSAI","features":["gamma-api","news-research","whale-alerts","dashboard-ai-agent"]}
---

# Oracle Sentinel

Autonomous AI agent for Polymarket prediction intelligence.

**Scans 190+ markets every 4 hours.** Real-time prices via Polymarket Gamma API. Deep news research from multiple sources. Whale trade alerts for $5K+ transactions. Dual-model AI analysis with radical transparency.

## Quick Start
```bash
# Get active trading signals
curl -s https://oraclesentinel.xyz/api/signals

# Get all monitored markets  
curl -s https://oraclesentinel.xyz/api/markets

# Get prediction tracking data
curl -s https://oraclesentinel.xyz/api/predictions

# Get full dashboard data
curl -s https://oraclesentinel.xyz/api/dashboard

# Check system health
curl -s https://oraclesentinel.xyz/api/health
```

## Dashboard AI Agent

Chat directly with Oracle Sentinel on the web dashboard: **https://oraclesentinel.xyz/app**

Click the chat button and ask:
- "Analyze this market: [Polymarket URL]"
- "Research and analyze: [Polymarket URL]" (includes deep news research)
- "What are the current signals?"
- "Show me your accuracy stats"

The AI agent fetches real-time market data, searches news from multiple sources, and reads full article content for deeper analysis.

## Telegram Bot

For interactive market analysis, message: **@oraclesentinel_pm_bot**

Send any Polymarket URL and the bot will:
1. Fetch real-time prices via Gamma API
2. Read resolution rules carefully
3. Check for loopholes in criteria
4. Search and read full news articles
5. Calculate AI probability
6. Compare vs market price
7. Return signal with full reasoning

## Whale Trade Alerts

Real-time monitoring of large trades on Polymarket.

- Scans every 5 minutes for trades > $5,000
- Each trade alerted only once (no duplicates)
- Includes full transaction hash for on-chain verification
- Alerts sent to Telegram automatically

## How It Works
```
┌─────────────────────────────────────────────────────────────┐
│                    ORACLE SENTINEL                          │
├─────────────────────────────────────────────────────────────┤
│  1. INGEST   → Fetch live prices via Polymarket Gamma API   │
│  2. RESEARCH → Search news (DuckDuckGo + Google News RSS)   │
│  3. EXTRACT  → Claude Haiku extracts facts (no opinions)    │
│  4. ASSESS   → Claude Sonnet computes AI probability        │
│  5. SIGNAL   → Edge calculator generates BUY/NO_TRADE       │
│  6. TRACK    → Accuracy tracker records every outcome       │
│  7. ALERT    → Whale trades monitor detects $5K+ moves      │
└─────────────────────────────────────────────────────────────┘
```

## Monitoring Systems

| System | Frequency | Purpose |
|--------|-----------|---------|
| Market Scanner | Every 4 hours | Analyze top markets, generate signals |
| Whale Monitor | Every 5 minutes | Detect $5,000+ trades with TX hash |
| Price Updater | Every scan | Track market movements |
| Accuracy Tracker | Continuous | Verify predictions against outcomes |

## API Reference

**Base URL:** `https://oraclesentinel.xyz/api`

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/signals` | Active trading signals with AI analysis |
| GET | `/markets` | All monitored Polymarket markets |
| GET | `/predictions` | Tracked predictions with accuracy data |
| GET | `/dashboard` | Complete dashboard data |
| GET | `/health` | System health status |

### Response: /api/signals
```json
[
  {
    "question": "Will X happen by Y date?",
    "signal_type": "BUY_YES",
    "ai_probability": 0.75,
    "market_price": 0.60,
    "edge": 15.0,
    "confidence": "HIGH",
    "reasoning": "AI explanation..."
  }
]
```

## Signal Types

| Signal | Meaning |
|--------|---------|
| `BUY_YES` | AI probability > market price by 5%+ |
| `BUY_NO` | AI probability < market price by 5%+ |
| `NO_TRADE` | Edge too small or low confidence |
| `SKIP` | Insufficient data or coin-flip market |

## Confidence Levels

| Level | Description |
|-------|-------------|
| `HIGH` | Strong data support, clear reasoning, required for signals |
| `MEDIUM` | Moderate data, requires 15%+ edge for signal |
| `LOW` | Limited data, never generates signal |

## Safety Overrides

Oracle Sentinel has hardcoded safety rules:
- Only HIGH confidence generates BUY signals
- MEDIUM confidence requires 15%+ edge
- Market 45-55% (coin-flip zone) = automatic skip
- Market >97% or <3% = automatic skip
- Low confidence = never generates signal

The system rejects its own overconfidence.

## Example: Fetch and Filter Signals
```python
import requests

signals = requests.get("https://oraclesentinel.xyz/api/signals").json()

for s in signals:
    if s["edge"] > 10 and s["confidence"] == "HIGH":
        print(f"{s['signal_type']}: {s['question']}")
        print(f"  Edge: +{s['edge']}%")
        print(f"  AI: {s['ai_probability']*100:.1f}%")
        print(f"  Market: {s['market_price']*100:.1f}%")
```

## Links

| Resource | URL |
|----------|-----|
| Dashboard | https://oraclesentinel.xyz/app |
| Documentation | https://oraclesentinel.xyz/docs |
| API Base | https://oraclesentinel.xyz/api |
| Skill File | https://oraclesentinel.xyz/skill.md |
| Telegram Bot | https://t.me/oraclesentinel_pm_bot |
| X (Twitter) | https://x.com/oracle_sentinel |
| GitHub | https://github.com/oraclesentinel/oracle-sentinel |
| Token ($OSAI) | https://solscan.io/token/HuDBwWRsa4bu8ueaCb7PPgJrqBeZDkcyFqMW5bbXpump |

## Powered By

- **Claude Sonnet 4.5** — Probability assessment
- **Claude Haiku 3.5** — Fact extraction
- **OpenClaw** — Autonomous agent gateway
- **Polymarket Gamma API** — Real-time market data

## About

Oracle Sentinel runs autonomously 24/7. Every 4 hours, the system triggers a full scan cycle — no human intervention required.

Every signal has a number. Every prediction is recorded. Every outcome is verified.