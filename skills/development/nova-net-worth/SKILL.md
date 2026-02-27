---
name: nova-net-worth
description: Query your Nova Net Worth financial data — net worth, accounts, goals, spending, transactions, AI insights, and health score. Use when the user asks about their finances, money, net worth, account balances, financial goals, spending habits, budget, savings, investments, debt, transactions, or financial health. Requires NOVA_API_KEY environment variable (API key from app.novanetworth.com → Settings → Integrations).
metadata:
  openclaw:
    requires:
      env:
        - NOVA_API_KEY
---

# Nova Net Worth API Skill

Query your complete financial picture from Nova Net Worth via the Agent API v1.

## Setup

Set the `NOVA_API_KEY` environment variable with your Nova API key:
```bash
export NOVA_API_KEY=nova_your_key_here
```

Generate your API key at: **app.novanetworth.com → Settings → Integrations**
Requires SuperNova ($19.99/mo) or Galaxy (enterprise) subscription.

## Quick Start

For any "how are my finances?" or daily briefing question, use the composite endpoint:
```bash
node scripts/nova-api.js briefing --pretty
```

## Available Commands

Run `scripts/nova-api.js` with a subcommand:

```bash
# Full financial briefing (RECOMMENDED — one call gets everything)
node scripts/nova-api.js briefing
node scripts/nova-api.js briefing --pretty    # Human-readable format

# Net worth summary
node scripts/nova-api.js summary

# All accounts with balances, grouped by type
node scripts/nova-api.js accounts

# Recent transactions with filtering
node scripts/nova-api.js transactions
node scripts/nova-api.js transactions --days 7 --limit 20
node scripts/nova-api.js transactions --category FOOD_AND_DRINK
node scripts/nova-api.js transactions --account acct_123
node scripts/nova-api.js transactions --since 2026-02-20T00:00:00Z  # Delta polling

# Financial goals with progress
node scripts/nova-api.js goals

# Monthly spending by category
node scripts/nova-api.js spending
node scripts/nova-api.js spending --months 3

# AI-generated financial insights
node scripts/nova-api.js insights

# Net worth trend over time
node scripts/nova-api.js history
node scripts/nova-api.js history --days 90

# Financial health score breakdown
node scripts/nova-api.js health
```

All commands support `--pretty` for human-readable output or `--json` (default) for raw JSON.

## When to Use Which Endpoint

| User Question | Command | Why |
|---|---|---|
| "How are my finances?" / "Financial update" | `briefing` | Everything in one call |
| "What's my net worth?" | `summary` | Quick headline number |
| "Show my accounts" / "How much in savings?" | `accounts` | All accounts with balances |
| "What did I spend on food?" / "Recent purchases" | `transactions --category FOOD_AND_DRINK` | Filterable transaction list |
| "Monthly spending breakdown" | `spending` | Categories with comparison |
| "Am I on track for my goals?" | `goals` | Progress tracking |
| "Any financial insights?" | `insights` | AI recommendations |
| "Net worth trend this year" | `history --days 365` | Historical snapshots |
| "How's my financial health?" | `health` | Score with recommendations |

## Response Format

All responses: `{ success: true, data: {...}, meta: { requestId, timestamp } }`

Money values are in **cents** (integer) with a `currency` field. Divide by 100 for display.
Example: `45840017` = `$458,400.17`

## Transaction Categories (Plaid)

Common categories for filtering: `FOOD_AND_DRINK`, `RENT_AND_UTILITIES`, `TRANSPORTATION`, `GENERAL_MERCHANDISE`, `TRANSFER_OUT`, `TRANSFER_IN`, `LOAN_PAYMENTS`, `ENTERTAINMENT`, `PERSONAL_CARE`, `MEDICAL`, `TRAVEL`, `INCOME`, `UNCATEGORIZED`

## Rate Limits

- SuperNova: 100 requests/hour
- Galaxy: 1,000 requests/hour
- Headers: `X-RateLimit-Remaining` shows remaining calls

## Delta Polling

For efficient monitoring, use `--since` with the timestamp of your last request:
```bash
node scripts/nova-api.js transactions --since 2026-02-25T12:00:00Z
```
This returns only new transactions since that time, minimizing data transfer.

## Environment

- `NOVA_API_KEY` (required) — Your Nova API key starting with `nova_`. Generate at app.novanetworth.com → Settings → Integrations.
- `NOVA_API_URL` (optional) — API base URL, defaults to `https://app.novanetworth.com`

## API Documentation

- OpenAPI spec: https://app.novanetworth.com/api-docs/openapi.yaml
- Interactive docs: https://novanetworth.com/api-docs
- AI plugin: https://novanetworth.com/.well-known/ai-plugin.json
