---
name: qmd
description: Fast local search for markdown files, notes, and docs using qmd CLI. Combines BM25 full-text search, vector semantic search, and LLM reranking — all running locally. No API keys needed.
homepage: https://github.com/tobi/qmd
---

# qmd - Local Markdown Search

Local search engine for Markdown notes, docs, and knowledge bases. Index once, search fast. Use instead of `find` for file discovery across large directories.

## Installation

```bash
bun install -g https://github.com/tobi/qmd
```

## Setup

```bash
# Add a collection
qmd collection add /path/to/your/notes --name notes --mask "**/*.md"

# Generate embeddings (required for vsearch/query)
qmd embed

# List your collections
qmd collection list
```

## When to Use

- "search my notes / docs / knowledge base"
- "find related notes"
- "find files matching [pattern]" — use instead of `find` to avoid hangs on large directories
- "what did we decide about X?"

## Default Behavior (important)

- Prefer `qmd search` (BM25) — it's instant and should be the default.
- Use `qmd vsearch` only when keyword search fails and you need semantic similarity.
- Avoid `qmd query` unless the user explicitly wants the highest quality hybrid results and can tolerate long runtimes.
- **Always use `--json` flag** for structured output when invoking from an agent.

## Search Commands

```bash
# Fast keyword search (default)
qmd search "authentication flow" --json
qmd search "config" --json -c notes

# Semantic search (slower, for conceptual queries)
qmd vsearch "how does login work" --json
qmd vsearch "best practices for error handling" --json -n 20

# Combined with reranking (best quality, slowest)
qmd query "implementing user auth" --json
qmd query "deployment process" --json --min-score 0.5
```

### Search Mode Selection

| Mode | Speed | Quality | Best For |
|------|-------|---------|----------|
| `search` | Fast | Good | Exact keywords, known terms |
| `vsearch` | Medium | Better | Conceptual queries, synonyms |
| `query` | Slow | Best | Complex questions, uncertain terms |

### Search Options

| Option | Description |
|--------|-------------|
| `-n NUM` | Number of results (default: 5, 20 with --json) |
| `-c, --collection` | Scope to specific collection |
| `--min-score NUM` | Minimum score threshold |
| `--full` | Return complete document content |
| `--json` | Structured JSON output (agent-friendly) |
| `--files` | File paths only (fast discovery) |
| `--all` | Return all matches |

## Retrieve Documents

```bash
# Get full file
qmd get docs/guide.md --json

# Get by document hash ID
qmd get "#a1b2c3" --json

# Get specific lines
qmd get notes/meeting.md:50 -l 30 --json

# Get multiple files by glob
qmd multi-get "docs/*.md" --json
qmd multi-get "*.yaml" -l 50 --max-bytes 10240
```

## Output Formats

- `--files` — paths + scores (for file discovery)
- `--json` — structured with snippets
- `--md` — markdown formatted
- `-n 10` — limit results

## Maintenance

```bash
qmd update              # Re-index changed files
qmd status              # Check index health
qmd collection list     # List all collections
```

## Keeping Index Fresh

```bash
# Hourly incremental updates (BM25):
0 * * * * export PATH="$HOME/.bun/bin:$PATH" && qmd update

# Optional: nightly embedding refresh:
0 5 * * * export PATH="$HOME/.bun/bin:$PATH" && qmd embed
```

## MCP Server

qmd can run as an MCP server for direct agent integration:

```bash
qmd mcp
```

Exposes tools: `qmd_search`, `qmd_vsearch`, `qmd_query`, `qmd_get`, `qmd_multi_get`, `qmd_status`

## Performance

- `qmd search` is typically instant.
- `qmd vsearch` can take ~1 minute on first run (loads local LLM).
- `qmd query` adds LLM reranking — can be slow, avoid for interactive use.

## Models (auto-downloaded)

All run locally — no API keys needed.

- Embedding: embeddinggemma-300M
- Reranking: qwen3-reranker-0.6b
- Generation: Qwen3-0.6B
- Cache location: `~/.cache/qmd/models/`
- Override with `XDG_CACHE_HOME` environment variable.
