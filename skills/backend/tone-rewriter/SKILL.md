---
name: tone-rewriter
description: "Rewrite text in any of 10 tones (professional, casual, friendly, formal, empathetic, persuasive, academic, simple, witty, urgent) while preserving meaning. x402 pay-per-use: $0.01 USDC. Use when: tone adjustment, rewrite text, change tone, professional rewrite, casual rewrite, make friendly, formalize text."
version: "1.0.0"
author: "Daisuke134"
metadata:
  openclaw:
    emoji: "🎭"
    os: ["darwin", "linux"]
---

# tone-rewriter — Tone Rewriting Service for AI Agents

Rewrite any text in 10 different tones while preserving the original meaning. Pay $0.01 USDC per request via x402.

## Prerequisite

```bash
npm install -g awal@2.0.3
awal auth login
```

## Usage

```bash
npx awal@2.0.3 x402 pay https://anicca-proxy-production.up.railway.app/api/x402/tone-rewriter \
  -X POST \
  -d '{"text":"Hey fix this ASAP","target_tone":"professional"}'
```

## Input Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| text | string | ✅ | Text to rewrite (max 2000 chars) |
| target_tone | string | ✅ | One of: professional, casual, friendly, formal, empathetic, persuasive, academic, simple, witty, urgent |
| language | string | ❌ | en, ja, or auto (default: auto) |
| preserve_length | boolean | ❌ | Keep output within ±20% of input length (default: false) |

## Output Schema

```json
{
  "rewrite_id": "rw_a1b2c3",
  "original_text": "Hey fix this ASAP",
  "rewritten_text": "Could you please address this issue at your earliest convenience?",
  "target_tone": "professional",
  "detected_original_tone": "urgent",
  "language": "en",
  "confidence": 0.95,
  "safe_t_flag": false
}
```

## Available Tones

| Tone | Description |
|------|-------------|
| professional | Formal business language, clear and authoritative |
| casual | Relaxed, conversational, everyday language |
| friendly | Warm, approachable, positive energy |
| formal | Structured, polished, official documents |
| empathetic | Understanding, compassionate, emotionally aware |
| persuasive | Compelling, action-oriented, motivating |
| academic | Scholarly, precise, evidence-based vocabulary |
| simple | Plain language, short sentences, easy to understand |
| witty | Clever, humorous undertone, engaging |
| urgent | Time-sensitive, direct, action-demanding |

## Pricing

- $0.01 USDC per request (Base network, x402 protocol)
- Endpoint: `https://anicca-proxy-production.up.railway.app/api/x402/tone-rewriter`

## Pairs well with

- **emotion-detector**: Detect emotion → choose appropriate tone → rewrite
- **buddhist-counsel**: Get guidance → rewrite in empathetic tone for the user
