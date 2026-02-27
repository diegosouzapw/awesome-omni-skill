---
name: prediction-markets-gina
displayName: Polymarket via Gina
description: Search, Trade, and Automate any strategy on Polymarket with your own agent.
version: 0.2.0
metadata:
  tags: polymarket, prediction-markets, trading, betting, gina, mcp, crypto, agent, agentic, bot, automated
authors:
  - Gina (@askginadotai)
---

# Polymarket via Gina

Trade Polymarket with AI — search, bet, track positions, and automate strategies from any MCP client.

**Server URL:** `https://askgina.ai/ai/predictions/mcp`

## Use This Skill For

### Searching & discovery

- Search prediction markets by topic, sport, keyword, or date
- Analyze market data with SQL (filter by volume, liquidity, end time, category)
- Find expiring markets about to resolve (last-minute opportunities)
- Discover recurring markets to set up automated strategies for (BTC, ETH, SOL — 5 min to monthly)
- Browse stock and index prediction markets (AAPL, S&P 500, Gold)

### Trading

- Place market orders and limit orders on Polymarket
- Track your positions, P&L, and win rate
- View and cancel open orders
- Redeem winnings from resolved markets

### Automating

- Set up Recipes — scheduled automations that trade or alert on your behalf.
- Daily market briefings, odds swing alerts, portfolio summaries.
- Set up fully automated trading strategies that scan, filter, trade, and journal for you.

## What You Can Do

Just type natural language prompts — no special syntax needed.

| Feature | Example Prompts |
|---------|----------------|
| **Search markets** | `"NBA markets tomorrow"` `"Fed rate decision odds"` |
| **Trending** | `"what's trending on Polymarket?"` `"markets with the most trading volume and liquidity"` |
| **Crypto prices** | `"BTC 15 minute up or down"` `"ETH current hourly up/down market"` |
| **Stocks & indices** | `"AAPL daily up or down"` `"S&P 500 daily"` |
| **Expiring markets** | `"markets ending in 2 hours"` `"NBA games ending tonight"` |
| **Place trades** | `"bet $10 on lakers to win"` `"buy $50 of Yes"` |
| **Limit orders** | `"buy Yes at 0.40 or better"` `"avoid slippage"` |
| **View positions** | `"show my Polymarket positions"` `"what's my P&L?"` |
| **Performance** | `"what's my win rate?"` `"show my trade history"` |
| **Manage orders** | `"show my open orders"` `"cancel all pending limit orders"` |
| **Redeem winnings** | `"redeem my winnings"` `"what can I redeem?"` |
| **Data analysis** | `"fetch crypto markets into SQL"` `"run a query"` |
| **Automations** | `"daily market briefing at 9am"` `"alert me on odds swings"` |

## Quick Start

1. Add the server URL to your MCP client (Claude Code, Codex, Cursor, Windsurf, etc.)
2. Your client opens a browser for sign-in — log in to Gina and approve access
3. Start prompting

For detailed client setup instructions, see the [Quick Start guide](https://docs.askgina.ai/predictions-mcp/quick-start).

## How It Works

- **Auth**: OAuth 2.1 with PKCE — your client handles it automatically. No API keys to manage.
- **Wallets**: Self-custodial via [Privy](https://privy.io). You own your keys.
- **Trades**: Execute on-chain on Polymarket (Polygon / USDC).
- **Gas**: Gina provides gas sponsorship to help cover transaction fees.
- **Safety**: Large trades require explicit confirmation before executing.
- **Automations**: Create scheduled jobs (market briefings, alerts) via natural language. Manage anytime.

## Safety

- Trading uses real money (USDC on Polygon) — always review before confirming
- Start with read-only prompts (search, trending) before trading
- Large trades require explicit confirmation
- If the auth flow asks for private keys, **do not proceed**

## Links

- **App**: https://askgina.ai
- **Docs**: https://docs.askgina.ai
- **Features**: https://docs.askgina.ai/predictions-mcp/features
- **Client setup**: https://docs.askgina.ai/predictions-mcp/client-setup
- **Troubleshooting**: https://docs.askgina.ai/predictions-mcp/troubleshooting
- **Terms**: https://askgina.ai/terms-and-conditions
- **Twitter**: https://x.com/askginadotai

