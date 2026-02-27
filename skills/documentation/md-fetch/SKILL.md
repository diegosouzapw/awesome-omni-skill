---
name: md-fetch
description: Fetch and read markdown files from the internet with support for partial content extraction and intelligent section ranking. Use when Claude needs to retrieve markdown documentation from URLs and extract specific portions by line numbers, section headers, search patterns, or query-based relevance ranking (BM25). Triggers include requests to fetch/download/read markdown from URLs, extract specific sections from online docs, find relevant sections by query, or get partial content from remote markdown files.
---

# Markdown Fetch Skill

Efficiently fetch and extract partial content from markdown files on the internet with intelligent section ranking.

## Quick Start

### Fetch Full File

```bash
scripts/fetch_md.sh <url> [output_file]
```

Examples:
```bash
# Print to stdout
scripts/fetch_md.sh https://bun.com/docs/runtime/sql.md

# Save to file
scripts/fetch_md.sh https://bun.com/docs/runtime/sql.md output.md
```

### Structured Retrieval (Recommended)

**Use BM25 ranking to find most relevant sections by query:**

```bash
scripts/fetch_md_structured.py <url> <query> [OPTIONS]
```

Examples:
```bash
# Find sections about "connection pooling"
scripts/fetch_md_structured.py https://bun.com/docs/runtime/sql.md "connection pooling"

# Get top 5 sections about "authentication" with scores
scripts/fetch_md_structured.py https://example.com/docs.md "authentication" --top 5 --show-scores

# Save results to file
scripts/fetch_md_structured.py https://example.com/docs.md "query hooks" --output result.md
```

**When to use:**
- Finding relevant information without knowing exact section names
- Exploring documentation by topic or concept
- Getting the most relevant sections ranked by query intent

### Fetch Partial Content

```bash
scripts/fetch_md_partial.sh <url> [OPTIONS]
```

## Partial Reading Methods

### By Line Numbers

Extract specific line ranges:

```bash
# Get lines 10-50
scripts/fetch_md_partial.sh https://bun.com/docs/runtime/sql.md --lines 10:50

# Get first 100 lines
scripts/fetch_md_partial.sh https://example.com/docs.md --lines 1:100
```

### By Section Headers

Extract specific markdown sections by header:

```bash
# Get "Installation" section
scripts/fetch_md_partial.sh https://ai-sdk.dev/docs/getting-started/expo.md --section "## Installation"

# Get "API Reference" section
scripts/fetch_md_partial.sh https://example.com/docs.md --section "# API Reference"
```

**Note:** Section extraction includes the header and all content until the next same-level header.

### By Search Pattern

Find and extract lines matching a pattern:

```bash
# Search for "query" keyword
scripts/fetch_md_partial.sh https://raw.githubusercontent.com/tanstack/query/main/docs/framework/react/overview.md --pattern "query"

# Search with context (3 lines before/after)
scripts/fetch_md_partial.sh https://example.com/docs.md --pattern "API" --context 3
```

## Saving Output

Add `--output` to save results to a file:

```bash
scripts/fetch_md_partial.sh https://example.com/docs.md --lines 1:100 --output result.md
scripts/fetch_md_partial.sh https://example.com/docs.md --section "## Setup" --output setup.md
```

## Common Use Cases

**Find relevant sections by topic (Structured Retrieval):**
```bash
# Most relevant approach - uses BM25 ranking
scripts/fetch_md_structured.py <url> "your search query" --top 3
```

**Preview documentation before full read:**
```bash
# Get first 50 lines to understand structure
scripts/fetch_md_partial.sh <url> --lines 1:50
```

**Extract specific documentation sections:**
```bash
# Get installation instructions only
scripts/fetch_md_partial.sh <url> --section "## Installation"
```

**Search for specific topics:**
```bash
# Find all mentions of "authentication" with context
scripts/fetch_md_partial.sh <url> --pattern "authentication" --context 5
```

## Structured Retrieval Details

The `fetch_md_structured.py` script uses **BM25 ranking** to intelligently rank sections by relevance:

**How it works:**
1. Extracts all markdown sections by headers
2. Tokenizes query and section content
3. Calculates BM25 scores with term frequency saturation
4. Boosts sections with query terms in headers (2x weight)
5. Returns top N most relevant sections

**Parameters:**
- `k1=1.2`: Term frequency saturation (higher = more weight to term frequency)
- `b=0.75`: Length normalization (higher = more penalty for long documents)

**Benefits:**
- No need to know exact section names
- Finds relevant content even with different terminology
- Ranks by semantic relevance, not just keyword matching
- In-memory processing (no persistent index needed)

## Supported URL Types

- Direct markdown URLs: `https://example.com/docs.md`
- GitHub raw URLs: `https://raw.githubusercontent.com/user/repo/main/README.md`
- Documentation sites: `https://bun.com/docs/runtime/sql.md`
- Any publicly accessible markdown file over HTTP/HTTPS

## Error Handling

Scripts use `set -e` and will exit on errors. Common issues:

- **404 Not Found**: URL is incorrect or file doesn't exist
- **Network errors**: Check internet connection
- **Invalid line range**: Ensure START:END format is correct
- **Section not found**: Check header format matches exactly (including `#` symbols)
