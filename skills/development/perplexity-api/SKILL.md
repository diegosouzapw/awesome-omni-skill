---
name: perplexity-api
description: "Perplexity API integration for building AI-powered applications with real-time web search and citations. Use when working with Perplexity Sonar models (sonar, sonar-pro, sonar-reasoning-pro, sonar-deep-research), Chat Completions (/chat/completions and /async/chat/completions), Agentic Research (/responses and presets like fast-search/pro-search/deep-research), Search API (/search), web_search options (search_domain_filter, search_recency_filter, search_mode, search_context_size, user_location), file/image attachments (file_url, image_url), returning images/videos, structured outputs (json_schema or regex), streaming (SSE), OpenAI-compatible usage via base URL https://api.perplexity.ai/v2, or Perplexity SDKs (Python: pip install perplexityai; TypeScript: @perplexity-ai/perplexity_ai). Triggers on Perplexity, Sonar, citations, web-grounded chat, pro-search, deep research, /responses, /chat/completions, or PERPLEXITY_API_KEY."
---

# Perplexity API

Integrate Perplexity's APIs (search-grounded chat, agentic research, and web search) without leaking API keys to the browser.

## Choose an API (fast routing)

- Use **Chat Completions** for "answer with citations" Q&A and lightweight research: `POST https://api.perplexity.ai/v2/chat/completions`.
- Use **Async Chat Completions** for long-running deep research with `sonar-deep-research`: `POST https://api.perplexity.ai/v2/async/chat/completions`.
- Use **Agentic Research (Responses)** for multi-step tool-using research flows: `POST https://api.perplexity.ai/v2/responses`.
- Use **Search API** when you want ranked results without synthesis: `POST https://api.perplexity.ai/v2/search`.

## Quick start (SDK-first)

### Installation

```bash
# Python
pip install perplexityai

# TypeScript/Node.js
npm install @perplexity-ai/perplexity_ai
```

### Client Setup

**TypeScript (Node.js):**
```ts
import Perplexity from "@perplexity-ai/perplexity_ai";

const apiKey = process.env["PERPLEXITY_API_KEY"];
if (!apiKey) throw new Error("Missing PERPLEXITY_API_KEY");

const client = new Perplexity({ apiKey });
```

**Python:**
```py
from perplexity import Perplexity

client = Perplexity()  # uses PERPLEXITY_API_KEY env var
```

## Chat Completions (Sonar + citations)

Basic "answer with sources" request:

**Python:**
```py
completion = client.chat.completions.create(
    model="sonar-pro",
    messages=[
        {"role": "system", "content": "Be precise and concise."},
        {"role": "user", "content": "What happened in tech news today?"}
    ],
    web_search_options={"search_context_size": "medium"},
)
print(completion.choices[0].message.content)

# Access citations (URLs of sources used)
for citation in completion.citations:
    print(f"Source: {citation}")
```

**TypeScript:**
```ts
const chunk = await client.chat.completions.create({
  model: "sonar-pro",
  messages: [
    { role: "system", content: "Be precise and concise." },
    { role: "user", content: "What happened in tech news today?" },
  ],
  web_search_options: { searchContextSize: "medium" },
});
console.log(chunk.content);
```

## OpenAI compatibility (when you already use OpenAI SDKs)

- Set the OpenAI SDK base URL to `https://api.perplexity.ai/v2`.
- Use **Chat Completions** (`/chat/completions`) for Sonar models and **Responses** (`/responses`) for Agentic Research.
- Avoid TypeScript `any` casts; prefer the Perplexity SDKs or a typed `fetch` wrapper when you need Perplexity-only request fields.

## References

- Chat Completions (models, web_search_options, structured outputs, streaming, attachments, return_images/videos): `references/chat-completions.md`
- Agentic Research / Responses (presets, tools web_search/fetch_url, streaming events): `references/agentic-research.md`
- Search API: `references/search-api.md`
- API key management (auth tokens/groups): `references/key-management.md`
