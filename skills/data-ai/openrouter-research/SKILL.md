---
name: openrouter-research
description: Research OpenRouter API docs, available Grok model IDs, vision capability for the judge service, and integration patterns. Use when implementing openrouter_tool.py, when checking which Grok model supports vision/image input for judge_service.py, when OpenRouter returns unexpected errors, or when verifying model availability and context limits.
tools: WebSearch, WebFetch, Read
---

# OpenRouter Research

Research current OpenRouter API specifications and Grok model availability for SFUMATO.

## Context

All text LLM calls in SFUMATO go through `app/tools/openrouter_tool.py`:
- `prompt_service.generate_prompt()` — Grok, text-only input
- `prompt_service.revise_prompt()` — Grok, text-only input
- `judge_service.evaluate()` — **Grok with vision** (must analyze generated image)

ENV vars: `OPENROUTER_API_KEY`, `OPENROUTER_BASE_URL=https://openrouter.ai/api/v1`
Client: `httpx.AsyncClient` — OpenAI-compatible API

Read existing file first if it exists:
- `app/tools/openrouter_tool.py`
- `config.py`

## Step 1: OpenRouter API Fundamentals

1. Fetch: `https://openrouter.ai/docs/api-reference/overview`
2. Fetch: `https://openrouter.ai/docs/requests`
3. Search: `OpenRouter API python httpx async chat completions 2025`

Capture:
- Base URL: `https://openrouter.ai/api/v1`
- Auth header format (`Authorization: Bearer` vs `API-Key`)
- Required request headers: `HTTP-Referer`, `X-Title`
- Chat completions endpoint: `POST /v1/chat/completions`
- Request body schema: `model`, `messages`, `temperature`, `max_tokens`, `stream`
- Response schema: `choices[0].message.content`
- Error response format and status codes (429, 402, 503, 500)

## Step 2: Available Grok Models

1. Fetch: `https://openrouter.ai/models?q=grok`
2. Search: `OpenRouter xAI Grok models list 2025`
3. Fetch: `https://openrouter.ai/x-ai` if available

For each available Grok model, capture:
- Full model ID string (e.g. `x-ai/grok-beta`, `x-ai/grok-vision-beta`)
- Context window size (tokens)
- **Supports image input (vision)?** — critical for judge_service
- Cost per 1M input/output tokens

Determine the best model for each use case:
- `prompt_gen`: text-only, large context
- `prompt_revise`: text-only
- `judge`: **MUST support vision/image input**

## Step 3: Image Input Format for Vision Models

Since `judge_service.py` sends a generated image to Grok for evaluation:

1. Search: `OpenRouter vision model image input base64 format 2025`
2. Fetch: `https://openrouter.ai/docs/features/vision`
3. Search: `OpenAI-compatible vision API image_url content type format`

Capture:
- Message content format for image:
  ```json
  {"type": "image_url", "image_url": {"url": "data:image/jpeg;base64,..."}}
  ```
  vs URL-based image reference
- Max image size limits
- Supported formats (JPEG, PNG, WebP)
- Whether OpenRouter passes base64 to xAI directly or requires URL

## Step 4: Error Handling and Retry Strategy

1. Search: `OpenRouter API error handling retry 429 503 2025`
2. Fetch: `https://openrouter.ai/docs/api-reference/errors` if available

Capture:
- 429 rate limit: `retry-after` header? fixed backoff interval?
- 402 payment/balance: raise immediately, no retry
- 503 model unavailable: retry with backoff
- Recommended timeout values for text calls vs vision calls
- Max retries pattern

## Output Format

### Section 1: openrouter_tool.py Implementation Blueprint

```python
# Async httpx client structure
BASE_URL = "https://openrouter.ai/api/v1"

# Required headers
headers = {
    "Authorization": f"Bearer {api_key}",
    "HTTP-Referer": "...",
    "X-Title": "SFUMATO",
    "Content-Type": "application/json",
}

# chat_completion(model, messages, temperature, max_tokens) -> str
# vision_completion(model, text_messages_plus_image_message) -> str
# Retry pattern for 429/503 (show how many retries, backoff)
# Error handling: which codes to retry vs raise immediately
```

### Section 2: Model Selection Table

| Use Case | Recommended Model ID | Context | Vision | Notes |
|----------|---------------------|---------|--------|-------|
| prompt_gen | x-ai/grok-... | N | No | |
| prompt_revise | x-ai/grok-... | N | No | |
| judge | x-ai/grok-... | N | **YES** | Required |

### Section 3: Image Message Format

Exact Python dict structure to use when sending image to judge:
```python
{
    "role": "user",
    "content": [
        {"type": "text", "text": "...judge prompt..."},
        {"type": "image_url", "image_url": {"url": "data:image/jpeg;base64,..."}}
    ]
}
```

### Section 4: Recommended Constants for config.py

```python
OPENROUTER_BASE_URL = "https://openrouter.ai/api/v1"
OPENROUTER_MODEL_PROMPT_GEN = "x-ai/grok-..."
OPENROUTER_MODEL_PROMPT_REVISE = "x-ai/grok-..."
OPENROUTER_MODEL_JUDGE = "x-ai/grok-vision-..."  # must support vision
OPENROUTER_TIMEOUT_TEXT = 30
OPENROUTER_TIMEOUT_VISION = 45
OPENROUTER_MAX_RETRIES = 2
```

## Notes

- Never log API key, response.text for large vision calls, or base64 image data
- Image for judge: read from `data/sessions/<id>/iter_<n>.jpg`, encode as base64
- The judge call is the only one that uses vision; wrap it separately in `vision_completion()`
- `HTTP-Referer` should be `"http://localhost:5000"` for local dev
