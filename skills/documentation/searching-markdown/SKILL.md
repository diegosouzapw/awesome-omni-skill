---
name: searching-markdown
description: "Local semantic search for markdown documents using qmd. Combines BM25 full-text, vector semantic, and LLM re-ranking—all on-device. Use when searching notes, documentation, meeting transcripts, or knowledge bases. Triggers on: search markdown, find in notes, query documents, semantic search, search my docs."
---

# qmd: Local Markdown Search

On-device semantic search for markdown. Combines BM25 full-text search, vector embeddings, and LLM re-ranking locally via GGUF models.

<installation>
**First-time setup?** Read `references/installation.md` for install and configuration steps.
</installation>

## Quick start

```bash
qmd search "keyword query"          # Fast BM25 full-text search
qmd vsearch "conceptual query"      # Vector semantic search
qmd query "complex question"        # Hybrid + LLM re-ranking (best quality)
qmd get notes/file.md               # Retrieve document (fuzzy matching)
```

## Search commands

```bash
qmd search "<query>"                # BM25 full-text (fast, keyword matching)
qmd vsearch "<query>"               # Vector semantic (conceptual similarity)
qmd query "<query>"                 # Hybrid + re-ranking (best results, slower)
```

### Search options

```bash
-n <num>              # Result count (default: 5; 20 with --files/--json)
-c, --collection      # Restrict to specific collection
--all                 # Return all matches (use with --min-score)
--min-score <num>     # Filter by relevance threshold (0.0-1.0)
--full                # Display complete document content
--line-numbers        # Add line annotations
```

### Output formats

```bash
--files               # TSV: docid, score, filepath, context
--json                # Structured JSON with snippets
--csv                 # Comma-separated values
--md                  # Markdown output
--xml                 # XML serialization
```

## Document retrieval

```bash
qmd get <filepath>                    # Get by path (fuzzy matching)
qmd get "#<docid>"                    # Get by 6-char document ID
qmd get <path>:<line> -l <count>      # Get from line, limit count
qmd multi-get "pattern/*.md"          # Multiple docs via glob
qmd multi-get "doc1.md, doc2.md"      # Comma-separated list
qmd multi-get "*.md" --max-bytes 20480  # Skip large files
```

## Collection management

```bash
qmd collection add <path> --name <name>   # Create indexed collection
qmd collection add <path> --mask "*.md"   # Custom glob pattern
qmd collection list                       # List all collections
qmd collection remove <name>              # Delete collection
qmd collection rename <old> <new>         # Rename collection
qmd ls <collection>[/<subfolder>]         # List files in collection
```

## Context & embeddings

```bash
qmd context add <path> "<description>"    # Add metadata to collection
qmd context list                          # View all contexts
qmd context rm <path>                     # Remove context
qmd embed                                 # Generate embeddings
qmd embed -f                              # Force re-embed everything
```

## Maintenance

```bash
qmd status                            # Index health & collection info
qmd update                            # Re-index all collections
qmd update --pull                     # Pull remote repos before indexing
qmd cleanup                           # Remove cache & orphaned data
```

## Score interpretation

| Score | Relevance |
|-------|-----------|
| 0.8–1.0 | Highly relevant |
| 0.5–0.8 | Moderately relevant |
| 0.2–0.5 | Somewhat relevant |
| 0.0–0.2 | Low relevance |

## Example: Setup and search workflow

```bash
# Index a collection
qmd collection add ~/notes --name notes
qmd context add qmd://notes "Personal notes and ideas"
qmd embed

# Search
qmd query "how does authentication work" -n 10 --min-score 0.3

# Get specific document
qmd get notes/auth-design.md
```

## Example: Agent-friendly JSON output

```bash
qmd search --json "configuration" -n 10
qmd query --json "API design patterns" --min-score 0.5
```
