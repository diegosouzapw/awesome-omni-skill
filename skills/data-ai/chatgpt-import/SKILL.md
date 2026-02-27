---
name: chatgpt-import
description: Import ChatGPT conversation history into OpenClaw's memory search. Use when migrating from ChatGPT, giving OpenClaw access to old conversations, or building a searchable archive of past chats.
---

# ChatGPT History Import

Import your ChatGPT conversations into OpenClaw so they're searchable via memory search.

## Workflow

### 1. Export from ChatGPT

Follow [references/export-guide.md](references/export-guide.md) to download your `conversations.json`.

### 2. Convert to Markdown

```bash
python3 scripts/convert_chatgpt.py \
  --input /path/to/conversations.json \
  --output /path/to/chatgpt-history
```

Options: `--min-messages N` to skip trivial conversations (default: 2).

### 3. Embed into SQLite

```bash
export OPENAI_API_KEY=sk-...
python3 scripts/bulk_embed.py \
  --history-dir /path/to/chatgpt-history \
  --db /path/to/chatgpt-memory.sqlite
```

Options: `--model`, `--batch-size`, `--max-workers`, `--chunk-size`, `--api-key`.

### 4. Configure OpenClaw

Add as an extra search path in your OpenClaw config:

```yaml
memorySearch:
  extraPaths:
    - /path/to/chatgpt-memory.sqlite
```

Then restart the gateway:

```bash
openclaw gateway restart
```

## Important Notes

- **OpenAI API key required.** The embed script sends conversation text to `api.openai.com` for embedding. If your conversations contain secrets, consider filtering them out first or using a scoped API key.
- **No key material stored.** The generated DB does not store your API key.
- **Back up first.** The embed script will refuse to overwrite an existing output DB.
- **Embeddings cost money** — but it's cheap. ~2,400 conversations cost ~$0.15 with `text-embedding-3-small`.
