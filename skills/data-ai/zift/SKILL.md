---
name: zift
description: Fast, semantic, and hybrid code search tool. Use when you need to find specific code patterns, understand architectural flows, or locate symbols across a large codebase using natural language, exact strings, or regular expressions.
---

# zift Skill

`zift` is a high-performance, local-first code search tool that combines semantic (vector) and lexical (FTS5) search.

## Core Search Modes

### 1. Semantic / Hybrid (Default)
Uses natural language to find code by "meaning" or "intent".
```bash
zift "how do I handle database connections" .
```
*Best for: High-level architectural questions, finding unfamiliar logic.*

### 2. Exact Match (`-e`, `--exact`)
Performs a literal, byte-perfect substring search.
```bash
zift -e "SearchResult { file_path" .
```
*Best for: Finding specific variable names, error strings, or boilerplate.*

### 3. Regex Search (`-r`, `--regex`)
Uses Rust-powered regular expressions for pattern matching.
```bash
zift -r "pub (async )?fn" .
```
*Best for: Finding all instances of a pattern (e.g., all public functions).*

## Workflow

### Indexing
Before searching, you must index the project. `zift` uses incremental indexing and will auto-refresh stale files on query, but a full initial index is recommended:
```bash
zift add .
```

### Filtering
Use the `-l` or `--local` flag to restrict results to the current working directory (useful in monorepos).
```bash
zift "auth logic" . --local
```

## Architecture & Performance
- **Local-first**: All embeddings and indices stay on your machine (`~/.cache/zift`).
- **GPU Accelerated**: Uses Metal/GPU for embedding generation via `llama-cpp-2`.
- **Hybrid RRF**: Fuses semantic and lexical results using Reciprocal Rank Fusion (k=60) with Power-Law scaling (γ=2.5) for intuitive percentages.
- **Speed**: Exact and Regex searches skip the embedding phase and are near-instant (<50ms).

## Troubleshooting
- **Stale Index**: If results seem old, run `zift add .` again or `zift forget .` to reset.
- **Model Issues**: `zift` defaults to `nomic-embed-text-v1.5`. Ensure the model is downloaded to the cache directory.
