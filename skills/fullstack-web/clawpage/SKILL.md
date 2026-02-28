---
name: clawpage
description: Extract and structure any web page into clean JSON. Returns text, tables, prices, contacts, hours, ratings, dates, links, and images. Handles JavaScript-rendered SPAs and bot-blocked sites. Free tier: 10 extractions/day with API key. Also supports x402 micropayments and prepaid USDC.
license: Proprietary
metadata:
  author: clawpage
  version: "0.2.0"
  category: web-extraction
  api_base: https://api.clawpage.xyz
  pricing: "Free tier: 10/day. Prepaid: deposit USDC. x402: 0.01 USDC per URL. Cache hits: free."
  network: base
---

# ClawPage

Web extraction API for agents. Converts any URL into structured JSON.

**Base URL:** `https://api.clawpage.xyz`

## When to Use This Skill

Use ClawPage when you need to:
- Extract structured data from a web page (prices, contacts, hours, ratings)
- Get clean text from a page that blocks bots or requires JavaScript
- Parse HTML tables into structured JSON
- Get metadata (title, description, language) from any URL

## Quick Start

### 1. Register (Free)

```bash
curl -X POST https://api.clawpage.xyz/register \
  -H "Content-Type: application/json" \
  -d '{"email": "you@example.com"}'
```

Response:
```json
{"api_key": "cpk_...", "free_extractions": 10}
```

### 2. Extract Any URL

```bash
curl -X POST https://api.clawpage.xyz/extract \
  -H "Content-Type: application/json" \
  -H "X-API-Key: cpk_..." \
  -d '{"url": "https://example.com", "sync": true}'
```

### 3. Get Structured JSON

```json
{
  "status": "done",
  "data": {
    "url": "https://example.com",
    "title": "Example Domain",
    "description": "...",
    "language": "en",
    "content": {
      "main_text": "...",
      "sections": [{"heading": "...", "text": "..."}]
    },
    "structured": {
      "tables": [{"headers": [...], "rows": [[...]]}],
      "prices": [{"item": "...", "amount": 9.99, "currency": "USD"}],
      "contacts": {"phones": [...], "emails": [...], "addresses": [...]},
      "hours": [{"day": "Monday", "open": "09:00", "close": "17:00"}],
      "ratings": {"score": 4.5, "max": 5, "count": 120, "source": "Google"},
      "dates": [{"label": "Event date", "iso": "2026-03-15"}],
      "links": [{"text": "...", "url": "..."}],
      "images": [{"alt": "...", "url": "..."}],
      "metadata": {}
    },
    "fetched_at": "2026-02-06T08:00:00Z",
    "cache_ttl_seconds": 86400,
    "render_method": "raw_fetch"
  }
}
```

## Authentication

Three options (choose one):

### API Key (Recommended)
1. `POST /register` with email to get API key
2. Pass `X-API-Key` header with every request
3. 10 free extractions per day (rolling 24h recharge)
4. Need more? Deposit USDC via `POST /account/deposit`

### x402 Micropayments (No Account Needed)
1. Send request without auth → get 402 with payment details
2. Send 0.01 USDC on Base to the recipient address
3. Retry with `X-Payment-Proof: <tx_hash>` header

### Cache Hits (Always Free)
- If another agent already extracted the URL in the last 24h, you get it free
- No auth required for cache hits

## Modes

### Sync Mode (Recommended)

Pass `"sync": true` to wait for the result inline.

### Async Mode

Omit `sync` or set `"sync": false`. You get a job ID to poll:

```json
{"status": "processing", "job_id": "abc123", "poll_url": "/result/abc123", "estimated_seconds": 15}
```

Poll: `GET /result/abc123`

## What Gets Extracted

| Field | Description |
|-------|-------------|
| `content.main_text` | Clean page text |
| `content.sections` | Text split by headings |
| `structured.tables` | HTML tables (headers + rows) |
| `structured.prices` | Products/services with amounts and currency |
| `structured.contacts` | Phone numbers, emails, physical addresses |
| `structured.hours` | Business hours by day |
| `structured.ratings` | Review scores, counts, source |
| `structured.dates` | Dates with labels in ISO format |
| `structured.links` | Up to 50 hyperlinks |
| `structured.images` | Up to 20 images with alt text |
| `title` | Page title |
| `description` | Meta description |
| `language` | Page language |

## Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `POST` | `/register` | None | Create account, get API key |
| `GET` | `/account` | API Key | Account info (balance, free remaining, wallets) |
| `POST` | `/account/wallets` | API Key | Associate wallet address |
| `POST` | `/account/deposit` | API Key | Credit USDC from on-chain tx |
| `POST` | `/extract` | API Key or x402 | Extract and structure a URL |
| `GET` | `/result/:job_id` | None | Poll for async results |
| `GET` | `/openapi.json` | None | OpenAPI specification |
| `GET` | `/.well-known/agent-service.json` | None | Agent discovery manifest |
| `GET` | `/health` | None | Health check |

## Pricing

- **Free tier:** 10 extractions per day (rolling 24h recharge)
- **Prepaid:** Deposit USDC, deducts $0.01 per extraction
- **x402:** 0.01 USDC per URL on Base (no account needed)
- **Cache hits:** Always free (24h TTL)

## Feedback

clawpage@fastmail.com
