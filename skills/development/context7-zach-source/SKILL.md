---
name: context7
description: "Library documentation lookup with tokenization and reranking. Use when user mentions external libraries, frameworks, or packages (e.g., how do I use React hooks, FastAPI authentication, pandas DataFrame). Triggers on library names, package imports, framework questions. Returns top 5 most relevant documentation chunks."
---

# Context7 - Library Documentation

Fetch up-to-date documentation for any library with intelligent chunking and reranking.

## Workflow

### Step 1: Resolve Library ID

First, resolve the library name to a Context7-compatible ID:

```
mcp__context7__resolve-library-id({ libraryName: "<package name>" })
```

Example: `libraryName: "react"` → returns `/facebook/react`

### Step 2: Get Documentation

Fetch docs with topic focus and token budget:

```
mcp__context7__get-library-docs({
  context7CompatibleLibraryID: "<library_id>",
  topic: "<specific topic>",
  tokens: 5000
})
```

### Step 3: Rerank Results

After receiving documentation, apply this mental reranking:

1. **Score by relevance** - Prioritize chunks containing query terms
2. **Consider recency** - Newer API patterns over deprecated ones
3. **Focus on examples** - Code snippets > prose descriptions
4. **Limit to top 5** - Present only the 5 most relevant chunks

## Quick Reference

| Library Type | Example Query | Topic |
|-------------|---------------|-------|
| React/Vue/Angular | "React hooks for state" | "hooks" or "useState" |
| FastAPI/Flask/Django | "FastAPI authentication" | "auth" or "security" |
| Pandas/NumPy | "pandas merge dataframes" | "merge" or "join" |
| SQLAlchemy/Prisma | "SQLAlchemy relationships" | "relationships" |
| Testing libs | "pytest fixtures" | "fixtures" |

## Token Budget Guidelines

- **Quick lookup**: 2000-3000 tokens
- **Deep dive**: 5000-8000 tokens
- **Comprehensive**: 10000+ tokens

## Reranking Script

For programmatic reranking, use the wrapper script:

```bash
# Process context7 output and rerank
python ~/.config/context7/context7_wrapper.py process \
  --query "authentication middleware" \
  --top 5 \
  --input docs.md
```

## Error Handling

If library not found:
1. Try alternative names (e.g., "nextjs" vs "next.js")
2. Check for organization prefix (e.g., "/vercel/next.js")
3. Search for related libraries

## Examples

### React Hooks
```
1. resolve-library-id({ libraryName: "react" })
2. get-library-docs({
     context7CompatibleLibraryID: "/facebook/react",
     topic: "hooks",
     tokens: 5000
   })
```

### FastAPI Auth
```
1. resolve-library-id({ libraryName: "fastapi" })
2. get-library-docs({
     context7CompatibleLibraryID: "/tiangolo/fastapi",
     topic: "security oauth2",
     tokens: 5000
   })
```
