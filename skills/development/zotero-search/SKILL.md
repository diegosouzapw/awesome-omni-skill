---
name: zotero-search
description: Search the user's local Zotero library, find related papers via Semantic Scholar, and discover citations/references. **Claude Code only** - this skill requires a local Zotero instance and access to a port on localhost. Use when the user asks to search their Zotero library, find papers on a topic, explore citations of a paper, or find related literature. Supports cross-referencing Semantic Scholar results against the local library.
---

# Zotero Search Skill

Search and retrieve documents from a local Zotero library using the pyzotero CLI. Extends to Semantic Scholar for discovering related papers, citations, and references beyond the local collection.

## Prerequisites

- Zotero desktop application running with local API enabled
- pyzotero CLI installed (`pip install pyzotero[cli]` or `uv tool install "pyzotero[cli]@latest"` if they have uv in their $PATH). The Pyzotero version must be >=1.9.0
- Test connection: `pyzotero test`

## Quick Start
If the user has uvx in their path, you can prefix these commands with `uvx --from "pyzotero[cli]@latest"`

```bash
# Basic search
pyzotero search -q "climate change" --json

# Search specific collection
pyzotero search -q "neural" --collection ABC123 --json

# Full-text search (searches PDF contents)
pyzotero search -q "methodology" --fulltext --json

# Filter by item type
pyzotero search -q "adaptation" --itemtype journalArticle --json

# Paginate through results
pyzotero search -q "climate" --limit 20 --offset 20 --json

# Filter by tag
pyzotero search -q "review" --tag "to-read" --json
```

## Local Search Options

| Option | Description | Example |
|--------|-------------|---------|
| `-q` | Search query (required) | `-q "machine learning"` |
| `--collection` | Limit to collection key | `--collection ABC123` |
| `--itemtype` | Filter by type | `--itemtype journalArticle` |
| `--fulltext` | Search PDF contents | `--fulltext` |
| `--limit` | Max results | `--limit 50` |
| `--offset` | Skip N results (for pagination) | `--offset 20` |
| `--tag` | Filter by tag | `--tag "to-read"` |
| `--json` | JSON output (required for local search) | `--json` |

### Item Types

Common types: `journalArticle`, `book`, `bookSection`, `conferencePaper`, `report`, `thesis`, `preprint`

Run `pyzotero itemtypes` for the full list.

### Collections

Run `pyzotero listcollections` to see all collections with their keys and names.

## Direct Item Access

Retrieve specific items directly by key:

```bash
# Get a single item by key
pyzotero item ABC123 --json

# Get child items (attachments, notes) for a parent item
pyzotero children ABC123 --json

# Get multiple items at once (batch lookup)
pyzotero subset ABC123 DEF456 GHI789 --json
```

The `subset` command is more efficient than multiple `item` calls when fetching several items.

## Tags

List and filter by tags in your library:

```bash
# List all tags in library
pyzotero tags

# List tags with usage counts (JSON output)
pyzotero tags --json

# Search filtered by tag
pyzotero search -q "methodology" --tag "important" --json
```

## Full-Text Content

Extract indexed full-text content from attachments:

```bash
# Get full-text content from an attachment
pyzotero fulltext ATTACHMENT_KEY
```

**Note:** The `fulltext` command requires the key of an attachment item (PDF, etc.), not the parent item. Use `pyzotero children PARENT_KEY --json` first to find attachment keys.

## Output Format

Search results include:

```json
{
  "count": 12,
  "items": [
    {
      "key": "ABC123",
      "itemType": "journalArticle",
      "title": "Paper Title",
      "creators": ["Author One", "Author Two"],
      "date": "2023-06-15",
      "publication": "Journal Name",
      "doi": "10.1234/example",
      "pdfAttachments": ["file:///path/to/paper.pdf"]
    }
  ]
}
```

Key fields: `title`, `creators`, `date`, `publication`, `doi`, `abstractNote`, `pdfAttachments`

## Semantic Scholar Integration

Extend beyond local library with Semantic Scholar commands:

| Command | Purpose | Example |
|---------|---------|---------|
| `related` | Find similar papers | `pyzotero related --doi "10.1234/..."` |
| `citations` | Find citing papers | `pyzotero citations --doi "10.1234/..."` |
| `references` | Find referenced papers | `pyzotero references --doi "10.1234/..."` |
| `s2search` | Search S2 by keyword | `pyzotero s2search -q "topic"` |

**Note:** Semantic Scholar commands output JSON by default and do not require (or support) the `--json` flag.

### Key Options

- `--min-citations N`: Filter to papers with N+ citations
- `--sort citations`: Sort by citation count (most cited first)
- `--check-library/--no-check-library`: Cross-reference with local Zotero

### Finding Influential Work

```bash
# Search for highly-cited papers on a topic
pyzotero s2search -q "deep learning" --sort citations --min-citations 100

# Find seminal works related to a paper you have
pyzotero related --doi "10.1234/example" --min-citations 200

# Find influential papers citing a foundational work
pyzotero citations --doi "10.1234/example" --min-citations 50
```

### Cross-Referencing

All Semantic Scholar results include `inLibrary: true/false` indicating whether each paper exists in local Zotero (matched by DOI).

**Performance tip:** Use `--no-check-library` for faster results when you don't need cross-referencing, or use the DOI index caching pattern (see `references/semantic-scholar.md`) for efficient repeated lookups.

For detailed Semantic Scholar documentation, see **`references/semantic-scholar.md`**.

## Research Assistant Behaviour

After presenting search results:

1. **Analyse patterns**: Identify themes, temporal patterns, coverage gaps
2. **Suggest next steps**: Offer to cluster results, find gaps, create reading lists
3. **Respect preferences**: Act like a diligent postdoc, not overeager

For detailed output formatting and proactive analysis patterns, see **`references/output-guidelines.md`**.

## Error Handling

| Error | Solution |
|-------|----------|
| Connection error | Ensure Zotero desktop is running |
| Empty results | Try broader terms, remove filters, use `--fulltext` |
| Paper not found (S2) | DOI may not be indexed; try `s2search` by title |
| Rate limit exceeded (S2) | Wait a moment and retry |

## Reference Files

For detailed guidance beyond this quick reference:

### Semantic Scholar
- **`references/semantic-scholar.md`** - Complete S2 command reference, workflows for finding seminal works, literature expansion patterns

### Research Analysis
- **`references/research-patterns.md`** - Thematic clustering, gap analysis, trajectory analysis, reading list prioritisation, methodology surveys

### Output and Presentation
- **`references/output-guidelines.md`** - Result formatting by count, proactive analysis patterns, common usage examples

### Technical Recipes
- **`references/jq-recipes.md`** - Set operations (intersection, union, difference), temporal filtering, processing recipes for grouping, counting, extracting
