---
name: summarize
description: Summarize or extract text/transcripts from URLs, podcasts, and local files.
emoji: 🧾
requires:
  bins: ["summarize"]
install:
  - id: brew
    kind: brew
    formula: steipete/tap/summarize
    bins: ["summarize"]
    label: Install summarize (brew)
---

# Summarize

Fast CLI to summarize URLs, local files, and YouTube links using the `summarize` CLI tool.

## When to use (trigger phrases)

Use this skill immediately when the user asks any of:
- "use summarize.sh"
- "what's this link/video about?"
- "summarize this URL/article"
- "transcribe this YouTube/video"
- "what does this article say?"

## Prerequisites

Install the summarize CLI:

```bash
brew install steipete/tap/summarize
```

## Quick start

```bash
# Summarize a URL
summarize "https://example.com" --model google/gemini-3-flash-preview

# Summarize a local file
summarize "/path/to/file.pdf" --model google/gemini-3-flash-preview

# Summarize a YouTube video
summarize "https://youtu.be/dQw4w9WgXcQ" --youtube auto
```

## YouTube: summary vs transcript

For best-effort transcript extraction (URLs only):

```bash
summarize "https://youtu.be/dQw4w9WgXcQ" --youtube auto --extract-only
```

If the user asked for a transcript but it's huge, return a tight summary first, then ask which section/time range to expand.

## Model + API keys

Set the API key for your chosen provider:
- OpenAI: `OPENAI_API_KEY`
- Anthropic: `ANTHROPIC_API_KEY`
- xAI: `XAI_API_KEY`
- Google: `GEMINI_API_KEY` (aliases: `GOOGLE_GENERATIVE_AI_API_KEY`, `GOOGLE_API_KEY`)

Default model is `google/gemini-3-flash-preview` if none is set.

## Useful flags

| Flag | Description |
|------|-------------|
| `--length short\|medium\|long\|xl\|xxl\|<chars>` | Summary length |
| `--max-output-tokens <count>` | Max output tokens |
| `--extract-only` | Extract text only (URLs) |
| `--json` | Machine readable output |
| `--firecrawl auto\|off\|always` | Fallback extraction |
| `--youtube auto` | Apify fallback for YouTube |

## Fallback without CLI

If the `summarize` CLI is not installed, you can fallback to using the web tool:

```bash
# Use fetch_url to get content, then have the LLM summarize
curl -s "https://example.com" | head -c 50000
```

Then ask the LLM to summarize the fetched content.

## Config

Optional config file: `~/.summarize/config.json`

```json
{ "model": "openai/gpt-4o" }
```

Optional services:
- `FIRECRAWL_API_KEY` for blocked sites
- `APIFY_API_TOKEN` for YouTube fallback
