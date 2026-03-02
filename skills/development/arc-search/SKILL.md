---
name: arc-search
description: Semantic and full-text search across indexed collections. Use when user mentions searching a collection, semantic search, vector search, knowledge base, or finding content in indexed code, markdown, or PDFs.
allowed-tools: Bash(arc:*), Read
---

```bash
arc collection list                                       # list collections
arc search semantic "query" --corpus NAME --limit 10      # conceptual queries
arc search semantic "query" --corpus A --corpus B         # multi-corpus search
arc search text "query" --corpus NAME --limit 10          # exact terms
```

Options: `--json` for structured output, `--filter "key=value"` for metadata filtering.
