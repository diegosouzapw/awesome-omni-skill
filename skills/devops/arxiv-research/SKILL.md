---
name: arxiv-research
description: Download and analyze academic papers from arXiv. Use when users want to download a specific paper by ID (e.g., "download paper arxiv:1234.5678") or read/analyze papers they've already downloaded.
---

# arXiv Research

Download and analyze academic papers from arXiv by paper ID.

## Scripts

All scripts in `scripts/` directory. Run with `uv run --with pymupdf python <script>` (pymupdf only needed for read_paper.py).

### download_paper.py
```bash
python scripts/download_paper.py PAPER_ID [--json]
```
Downloads PDF to `~/.arxiv-papers/`

### list_papers.py
```bash
python scripts/list_papers.py [--json]
```
Lists all downloaded papers.

### read_paper.py
```bash
uv run --with pymupdf python scripts/read_paper.py PAPER_ID [-p MAX_PAGES] [--json]
```
Extracts text from downloaded PDF. Requires pymupdf.

## Workflow

1. Download paper by ID: `python scripts/download_paper.py ID`
2. Read: `uv run --with pymupdf python scripts/read_paper.py ID`

Storage: `~/.arxiv-papers/` (override with `ARXIV_STORAGE_PATH` env var)
