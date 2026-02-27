---
name: qmd
description: Search Tim's Obsidian vault with semantic and keyword search via qmd.
homepage: https://github.com/tobi/qmd
metadata: {"clawdis":{"emoji":"🔍","requires":{"bins":["qmd"]}}}
---

# QMD - Vault Search

Search Tim's Obsidian vault (Second Brain) using qmd — combines BM25 keyword search, vector semantic search, and LLM reranking.

## When to Use

- Tim asks about something that might be in his notes
- Looking up personal preferences, past decisions, documentation
- Finding specific notes or topics in the vault
- Any question about "what did I write about X"

## Quick Commands

```bash
# Set PATH first (required!)
export PATH="$HOME/.bun/bin:$PATH"

# Fast keyword search
qmd search "query" -n 5

# Semantic search (understands meaning)
qmd vsearch "how to deploy" -n 5

# Best quality: hybrid + reranking
qmd query "quarterly planning" -n 5

# Search in vault collection specifically
qmd search "API" -c vault

# Get a specific document
qmd get "vault/path/to/note.md"

# Get by docid (from search results)
qmd get "#abc123"

# List all files in vault
qmd ls vault
```

## Output Formats

```bash
# Default: human readable snippets
qmd search "query"

# JSON for parsing
qmd search "query" --json

# Just file paths with scores
qmd search "query" --files

# Full document content
qmd search "query" --full
```

## Collections

Currently indexed:
- **vault** — Tim's Obsidian vault (`/home/clawdis/clawd/obsidian-vault`)

## Updating the Index

```bash
export PATH="$HOME/.bun/bin:$PATH"

# Re-index after vault changes
qmd update

# Update with git pull first
qmd update --pull

# Regenerate embeddings
qmd embed
```

## Tips

1. **Start with `search`** for exact terms, use `vsearch` or `query` for concepts
2. **Use `-n 10`** if you need more results
3. **Use `--full`** to get complete document content
4. **Check `qmd status`** to see index health

## Example Workflow

```bash
export PATH="$HOME/.bun/bin:$PATH"

# Tim asks "what series do I like?"
qmd search "Streaming Serien" -n 3

# Tim asks "how do I deploy to production?"
qmd query "deployment process production" -n 5 --full
```
