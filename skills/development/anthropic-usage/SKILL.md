---
name: anthropic-usage
description: Check Anthropic API usage and costs for any time period. Use when the user asks about API costs, usage, spending, or billing for their Anthropic account. Supports natural language periods like "last week", "yesterday", "january 2025", specific dates, or date ranges.
---

# Anthropic API Usage

Fetch cost reports from the Anthropic Admin API.

## Prerequisites

- `ANTHROPIC_ADMIN_KEY` must be set in `~/.env`

## Usage

Run the script with a time period:

```bash
~/.claude/skills/anthropic-usage/scripts/get_cost.py "<period>"
```

## Supported Period Formats

| Format | Example | Description |
|--------|---------|-------------|
| Relative week | `last week`, `this week` | 7-day periods |
| Relative month | `last month`, `this month` | Calendar months |
| Relative day | `yesterday`, `today` | Single days |
| Last N days | `last 3 days`, `last 30 days` | Rolling window |
| Specific date | `2026-02-01` | Single UTC day |
| Date range | `2026-01-01 to 2026-01-31` | Inclusive range |
| Month name | `january`, `feb 2025` | Full calendar month |

## Options

- `--format table` (default): Human-readable table with totals
- `--format json`: Raw API response

## Notes

- Costs are returned in USD
- Data is reported in UTC day buckets
- The API returns amounts in cents; the script converts to dollars
