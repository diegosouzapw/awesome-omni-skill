---
name: openclaw-starter-kit
description: "Replace 100+ API keys with one. Instant access to LLMs, Twitter, YouTube, LinkedIn, Finance, Tavily & Scholar data. Enterprise stability for your local agent."
homepage: https://openclaw.ai
metadata: {"openclaw":{"emoji":"🦞","requires":{"bins":["curl","python3"],"env":["AISA_API_KEY"]},"primaryEnv":"AISA_API_KEY"}}
---

# OpenClaw Starter Kit 🦞

**The definitive starting point for autonomous agents. Powered by AIsa.**

One API key. All the data sources your agent needs.

## 🔥 What Can You Do?

### Morning Briefing (Scheduled)
```
"Send me a daily briefing at 8am with:
- My portfolio performance (NVDA, TSLA, BTC)
- Twitter trends in AI
- Top news in my industry"
```

### Competitor Intelligence
```
"Monitor @OpenAI - alert me on new tweets, news mentions, and paper releases"
```

### Investment Research
```
"Full analysis on NVDA: price trends, insider trades, analyst estimates, 
SEC filings, and Twitter sentiment"
```

### Startup Validation
```
"Research the market for AI writing tools - find competitors, 
Twitter discussions, and academic papers on the topic"
```

### Crypto Whale Alerts
```
"Track large BTC movements and correlate with Twitter activity"
```

## AIsa vs bird

| Feature | AIsa ⚡ | bird 🐦 |
|---------|---------|---------|
| Auth method | API Key (simple) | Browser cookies (complex) |
| Read Twitter | ✅ | ✅ |
| Post/Like/Retweet | ✅ (via login) | ✅ |
| Web Search | ✅ | ❌ |
| Scholar Search | ✅ | ❌ |
| News/Financial | ✅ | ❌ |
| LLM Routing | ✅ | ❌ |
| Server-friendly | ✅ | ❌ |
| Cost | Pay-per-use | Free |

**Use AIsa when**: Server environment, need search/scholar APIs, prefer simple API key setup.
**Use bird when**: Local machine with browser, need free access, complex Twitter interactions.

## Quick Start

```bash
export AISA_API_KEY="your-key"
```

## Core Capabilities

### Twitter/X Data (Read)

```bash
# Get user info
curl "https://api.aisa.one/apis/v1/twitter/user/info?userName=elonmusk" \
  -H "Authorization: Bearer $AISA_API_KEY"

# Advanced tweet search
curl "https://api.aisa.one/apis/v1/twitter/tweet/advanced_search?query=AI+agents&queryType=Latest" \
  -H "Authorization: Bearer $AISA_API_KEY"

# Get trending topics (worldwide)
curl "https://api.aisa.one/apis/v1/twitter/trends?woeid=1" \
  -H "Authorization: Bearer $AISA_API_KEY"
```

### Twitter/X Post (Write)

> **Security Notice**: Posting requires account login. Credentials are read from environment variables to avoid exposure in shell history or logs. Set `TWITTER_EMAIL`, `TWITTER_PASSWORD`, and `TWITTER_PROXY` before using write operations.

> **Warning**: Use responsibly to avoid rate limits or account suspension.

```bash
# Set credentials via environment variables (never pass as CLI arguments)
export TWITTER_EMAIL="your-email"
export TWITTER_PASSWORD="your-password"
export TWITTER_PROXY="http://ip:port"

# Step 1: Login first (async, check status after)
curl -X POST "https://api.aisa.one/apis/v1/twitter/user_login_v3" \
  -H "Authorization: Bearer $AISA_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"user_name\":\"myaccount\",\"email\":\"$TWITTER_EMAIL\",\"password\":\"$TWITTER_PASSWORD\",\"proxy\":\"$TWITTER_PROXY\"}"

# Step 2: Send tweet
curl -X POST "https://api.aisa.one/apis/v1/twitter/send_tweet_v3" \
  -H "Authorization: Bearer $AISA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"user_name":"myaccount","text":"Hello from OpenClaw!"}'

# Like / Retweet
curl -X POST "https://api.aisa.one/apis/v1/twitter/like_tweet_v3" \
  -H "Authorization: Bearer $AISA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"user_name":"myaccount","tweet_id":"1234567890"}'
```

### Search (Web + Academic)

```bash
# Web search
curl -X POST "https://api.aisa.one/apis/v1/scholar/search/web?query=AI+frameworks&max_num_results=10" \
  -H "Authorization: Bearer $AISA_API_KEY"

# Academic/scholar search
curl -X POST "https://api.aisa.one/apis/v1/scholar/search/scholar?query=transformer+models&max_num_results=10" \
  -H "Authorization: Bearer $AISA_API_KEY"

# Smart search (web + academic combined)
curl -X POST "https://api.aisa.one/apis/v1/scholar/search/smart?query=machine+learning&max_num_results=10" \
  -H "Authorization: Bearer $AISA_API_KEY"
```

### Financial News

```bash
# Company news by ticker
curl "https://api.aisa.one/apis/v1/financial/news?ticker=AAPL&limit=10" \
  -H "Authorization: Bearer $AISA_API_KEY"
```

### LLM Routing (OpenAI Compatible)

```bash
curl -X POST "https://api.aisa.one/v1/chat/completions" \
  -H "Authorization: Bearer $AISA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "gpt-4", "messages": [{"role": "user", "content": "Hello"}]}'
```

Supported models: GPT-4, Claude-3, Gemini, Qwen, Deepseek, Grok, and more.

## Python Client

```bash
# Twitter Read
python3 {baseDir}/scripts/aisa_client.py twitter user-info --username elonmusk
python3 {baseDir}/scripts/aisa_client.py twitter search --query "AI agents"
python3 {baseDir}/scripts/aisa_client.py twitter trends --woeid 1

# Twitter Write (requires login first, credentials from env vars)
python3 {baseDir}/scripts/aisa_client.py twitter login --username myaccount
python3 {baseDir}/scripts/aisa_client.py twitter post --username myaccount --text "Hello!"
python3 {baseDir}/scripts/aisa_client.py twitter like --username myaccount --tweet-id 1234567890

# Search
python3 {baseDir}/scripts/aisa_client.py search web --query "latest AI news"
python3 {baseDir}/scripts/aisa_client.py search scholar --query "LLM research"
python3 {baseDir}/scripts/aisa_client.py search smart --query "machine learning"

# News
python3 {baseDir}/scripts/aisa_client.py news --ticker AAPL

# LLM
python3 {baseDir}/scripts/aisa_client.py llm complete --model gpt-4 --prompt "Explain quantum computing"
```

## Pricing

| API | Cost |
|-----|------|
| Twitter query | ~$0.0004 |
| Twitter post/like | ~$0.001 |
| Web search | ~$0.001 |
| Scholar search | ~$0.002 |
| News | ~$0.001 |
| LLM | Token-based |

Every response includes `usage.cost` and `usage.credits_remaining`.

## Error Handling

Errors return JSON with `error` field:

```json
{
  "error": "Invalid API key",
  "code": 401
}
```

Common error codes:
- `401` - Invalid or missing API key
- `402` - Insufficient credits
- `429` - Rate limit exceeded
- `500` - Server error

## Get Started

1. Sign up at [aisa.one](https://aisa.one)
2. Get your API key
3. Add credits (pay-as-you-go)
4. Set environment variable: `export AISA_API_KEY="your-key"`

## Full API Reference

See [API Reference](https://github.com/AIsa-team/Openclaw-Starter-Kit/blob/main/skills/aisa/references/api-reference.md) for complete endpoint documentation.
