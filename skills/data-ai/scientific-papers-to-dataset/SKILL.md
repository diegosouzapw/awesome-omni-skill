---
name: scientific-papers-to-dataset
description: Build structured datasets from academic papers. Use when the user wants to extract structured data from scientific literature, traverse citation graphs, search OpenAlex for papers, or create datasets from PDFs for research purposes.
---

# scientific-papers-to-dataset

Build datasets by extracting structured data from academic papers and traversing citation graphs.

## When to Use This Skill

Use this skill when the user wants to:

- Create a dataset from academic papers
- Extract structured information from PDFs
- Search for papers on a topic using OpenAlex
- Traverse citation graphs to find related papers

## Architecture: Subagent Pattern

> [!IMPORTANT]
> Use **subagents** for PDF download, relevance checking, data extraction, and citation traversal to keep the main context clean.

### Recommended Subagents

1. **pdf-downloader** - Downloads PDF for a paper ID
2. **relevance-checker** - Evaluates paper relevance from title/abstract
3. **data-extractor** - Reads PDF and extracts structured data (use thinking model)
4. **citation-traverser** - Fetches related/cited/citing papers from OpenAlex

## Workflow

### Step 1: Project Setup

From user's description, generate project assets. User should provide:

- **Goal**: What dataset they want to create
- **Domain**: Research area and key terminology
- **Data fields**: What information to extract from papers

Create project directory with these files:

```
projects/<project_name>/
├── prompt.txt           # Data extraction instructions
├── relevance_prompt.txt # Relevance criteria for papers
├── search_query.txt     # OpenAlex search terms
├── bfs_queue.json       # BFS queue state (see assets)
├── pdfs/                # Downloaded PDFs
└── data/                # Extracted JSON files
```

**Generate assets by creating:**

1. **prompt.txt**: Detailed instructions for extracting data from PDFs
   - What fields to extract
   - Domain context and terminology
   - Output format (JSON structure)
   - Guidelines for handling missing/ambiguous data

2. **relevance_prompt.txt**: Criteria for filtering papers
   - What makes a paper relevant
   - Template: `{title}` and `{abstract}` placeholders

3. **search_query.txt**: OpenAlex search query
   - Domain-specific terms
   - Broad enough for coverage, specific enough for relevance

### Step 2: Initial Paper Search

Search OpenAlex to populate the BFS queue:

```
GET https://api.openalex.org/works?search=<query>&per-page=25&mailto=email
```

Extract OpenAlex IDs (e.g., `W2741809807`) from results and add to `bfs_queue.json`.

**Options:**

- Use [search_openalex.py](scripts/search_openalex.py) script
- Write equivalent code in preferred language
- Install uv (`curl -LsSf https://astral.sh/uv/install.sh | sh`) and use Python directly

See [bfs_queue.py](references/bfs_queue.py) for queue implementation reference.

### Step 3: Process Queue (Loop)

Pop paper ID from queue and process with subagents:

#### 3a. Download PDF (subagent: pdf-downloader)

```
Download PDF for OpenAlex ID: <id>
Save to: projects/<name>/pdfs/<id>.pdf
Return: success/failure
```

If failed → mark as `failed: no_pdf` in queue, continue to next paper from queue.

#### 3b. Check Relevance (subagent: relevance-checker)

```
Given title and abstract from OpenAlex metadata,
evaluate using: [relevance_prompt.txt]
Return: {is_relevant: bool, reason: string}
```

If not relevant → mark as `skipped: <reason>` in queue, continue to next paper from queue.

#### 3c. Extract Data (subagent: data-extractor with thinking model)

```
Read PDF: projects/<name>/pdfs/<id>.pdf
Extract data following: [prompt.txt]
Return: structured JSON
```

Save result to `projects/<name>/data/<id>.json`.

#### 3d. Traverse Citations (subagent: citation-traverser)

```
For OpenAlex ID: <id>
Fetch: referenced_works, related_works, citing works
Return: list of new paper IDs
```

Add new IDs to queue (skip already processed/skipped/failed).
Mark current paper as `processed`.

### Step 4: Continue Until Done

Repeat Step 3 until:

- User stops the process
- Queue is empty (all papers in processed/skipped/failed state)
- User provides new seed papers or search queries

## BFS Queue Format

Use `bfs_queue.json` for stop/resume:

```json
{
  "queue": ["W123", "W456"],
  "processed": ["W789"],
  "skipped": {"W111": "review article, no experimental data"},
  "failed": {"W222": "pdf not available"}
}
```

## Key Principles

1. **Use subagents** for each processing step to preserve main context
2. **Use thinking model** for data extraction (complex reasoning needed)
3. **Handle failures gracefully** - ~30-50% of papers won't have accessible PDFs
4. **Track everything** - queue.json enables stop/resume at any point
5. **Rate limit OpenAlex** - 10 req/sec with email, 1 req/sec without

## References

- [OPENALEX.md](references/OPENALEX.md) - OpenAlex API reference
- [WORKFLOW.md](references/WORKFLOW.md) - Detailed workflow steps
- [bfs_queue.py](references/bfs_queue.py) - Queue implementation reference
- [download_pdf.py](references/download_pdf.py) - PDF download reference with some of the logic for downloading PDFs
