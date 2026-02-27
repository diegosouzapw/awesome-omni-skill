---
name: moltpulse
description: Generate industry intelligence reports with MoltPulse. Run when user says "generate report", "daily brief", "weekly digest", "industry analysis", "market intelligence", or "moltpulse run".
homepage: https://github.com/moltpulse/moltpulse
user-invocable: true
allowed-tools: Bash, Read
metadata: {"openclaw":{"emoji":"📊","requires":{"bins":["moltpulse"]},"primaryEnv":"ALPHA_VANTAGE_API_KEY","install":[{"id":"uv","kind":"uv","package":"moltpulse","bins":["moltpulse"],"label":"Install MoltPulse (uv)"}]}}
---

# MoltPulse - Industry Intelligence Framework

Generate daily, weekly, and specialized reports for monitored industries. MoltPulse collects data from financial APIs, news sources, RSS feeds, and social media, then generates formatted intelligence reports.

## Quick Start

```bash
# Generate daily brief for advertising industry
moltpulse run --domain=advertising --profile=ricki daily

# Generate weekly digest with deep collection
moltpulse run --domain=advertising --profile=ricki --deep weekly

# Generate fundraising outlook
moltpulse run --domain=advertising --profile=ricki fundraising
```

## Commands

### Report Generation

```bash
moltpulse run --domain=DOMAIN --profile=PROFILE REPORT_TYPE [options]
```

**Report Types:**
- `daily` - Compact morning brief (stock performance, top 5 news, thought leader highlights)
- `weekly` - Comprehensive weekly analysis (15 news items, trends, M&A activity)
- `fundraising` - Nonprofit-focused outlook (timing recommendations, 6-month/1-year/3-year outlook)

**Options:**
- `--quick` - Fast, shallow collection
- `--deep` - Thorough, comprehensive collection
- `--deliver` - Send via profile's delivery settings
- `--no-deliver` - Output to stdout only (for OpenClaw orchestration)
- `--output FORMAT` - Format: `compact`, `json`, `markdown`, `full`
- `--dry-run` - Show configuration without running

### Configuration Management

```bash
# List all configuration
moltpulse config list

# Set an API key
moltpulse config set ALPHA_VANTAGE_API_KEY your_key

# Get a specific value
moltpulse config get ALPHA_VANTAGE_API_KEY

# Test API key validity
moltpulse config test ALPHA_VANTAGE_API_KEY
```

### Domain Management

```bash
# List available domains
moltpulse domain list

# Show domain details
moltpulse domain show advertising

# Create new domain
moltpulse domain create healthcare --display-name "Healthcare Intelligence"

# Add entities
moltpulse domain update healthcare --add-entity "hospital_systems:HCA:HCA Healthcare"
```

### Profile Management

```bash
# List profiles in a domain
moltpulse profile list advertising

# Show profile details
moltpulse profile show advertising ricki

# Create new profile
moltpulse profile create advertising sarah \
  --thought-leader "Seth Godin:ThisIsSethsBlog:1" \
  --publications "Ad Age,AdWeek" \
  --delivery-channel email \
  --delivery-email "sarah@example.com"
```

### Cron Integration (OpenClaw)

```bash
# List available cron templates
moltpulse cron list

# Show template details
moltpulse cron show daily-brief

# Install cron jobs to OpenClaw
moltpulse cron install --all

# Install specific job
moltpulse cron install daily-brief

# Preview without installing
moltpulse cron install --all --dry-run
```

## OpenClaw Orchestration

When running under OpenClaw orchestration, use `--no-deliver` so OpenClaw handles delivery:

```bash
# OpenClaw runs this and delivers output to configured channel
moltpulse run --domain=advertising --profile=ricki --output=markdown --no-deliver daily
```

### Bundled Cron Templates

| Template | Schedule | Description |
|----------|----------|-------------|
| `daily-brief` | Weekdays 7am PT | Morning intelligence brief |
| `weekly-digest` | Mondays 8am PT | Comprehensive weekly analysis |
| `fundraising-outlook` | 1st of month 9am PT | Monthly fundraising outlook |

## Available Domains

Run `moltpulse domain list` to see configured domains.

**Current domains:**
- `advertising` - Advertising industry monitoring (holding companies, agencies, M&A)

## Delivery Channels

Reports can be delivered via:
- `console` - Print to terminal (default)
- `file` - Save to local file
- `email` - Send via SMTP

Configure delivery in profile YAML or via CLI:

```bash
moltpulse profile update advertising ricki \
  --set-delivery-channel email \
  --set-delivery-email "user@example.com"
```

## API Keys Required

Configure in `~/.config/moltpulse/.env` or via `moltpulse config set`:

| Key | Purpose |
|-----|---------|
| `ALPHA_VANTAGE_API_KEY` | Stock data (required) |
| `NEWSDATA_API_KEY` | News aggregation (required) |
| `XAI_API_KEY` | X/Twitter search (optional) |

## Examples

### Morning Brief Workflow

```bash
# Check what will be collected
moltpulse run --domain=advertising --profile=ricki --dry-run daily

# Generate and deliver
moltpulse run --domain=advertising --profile=ricki --deliver daily
```

### Setup New Industry Domain

```bash
# Create domain
moltpulse domain create fintech --display-name "Fintech Intelligence"

# Add entities
moltpulse domain update fintech \
  --add-entity "banks:JPM:JPMorgan Chase" \
  --add-entity "banks:BAC:Bank of America"

# Create profile
moltpulse profile create fintech analyst1 \
  --thought-leader "Cathie Wood:CathieDWood:1" \
  --publications "Bloomberg,WSJ"

# Generate report
moltpulse run --domain=fintech --profile=analyst1 daily
```

### Output Formats

```bash
# Compact (default) - brief summary
moltpulse run --domain=advertising --profile=ricki daily

# Markdown - formatted for reading
moltpulse run --domain=advertising --profile=ricki --output=markdown daily

# JSON - for programmatic use
moltpulse run --domain=advertising --profile=ricki --output=json daily

# Full - all collected data
moltpulse run --domain=advertising --profile=ricki --output=full daily
```
