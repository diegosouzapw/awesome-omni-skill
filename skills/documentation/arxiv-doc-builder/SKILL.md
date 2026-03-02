---
name: arxiv-doc-builder
description: Automatically convert arXiv papers to well-structured Markdown documentation. Invoke with an arXiv ID to fetch materials (LaTeX source or PDF), convert to Markdown, and generate implementation-ready reference documentation with preserved mathematics and section structure.
---

# arXiv Document Builder

Automatically converts arXiv papers into structured Markdown documentation for implementation reference.

## Capabilities

This skill automatically:

1. **Fetches paper materials from arXiv**
   - Attempts to download LaTeX source first (preferred for accuracy)
   - Falls back to PDF if source is unavailable
   - Handles all HTTP requests, extraction, and directory setup

2. **Converts to structured Markdown**
   - LaTeX source → Markdown via pandoc (preserves all math and structure)
   - PDF → Markdown via text extraction with multiple conversion modes:
     - Simple single-column conversion (default)
     - Full double-column conversion for academic papers
     - Page-wise extraction with mixed column support
   - Preserves mathematical formulas in MathJax/LaTeX format (`$...$`, `$$...$$`)
   - Maintains section hierarchy and document structure
   - Includes abstracts, figures, and references

3. **Generates implementation-ready documentation**
   - Output saved to `papers/{ARXIV_ID}/{ARXIV_ID}.md`
   - Easy to reference during code implementation
   - Optimized for Claude to read and understand

## When to Use This Skill

Invoke this skill when the user requests:
- "Convert arXiv paper {ID} to markdown"
- "Fetch and process paper {ID}"
- "Create documentation for arXiv:{ID}"
- "I need to read/reference paper {ID}"

## How It Works

### Single Entry Point

Use the main orchestrator script which handles everything automatically:

```bash
python scripts/convert_paper.py ARXIV_ID [--output-dir DIR]
```

The orchestrator:
1. Calls `fetch_paper.py` to download materials (with automatic source→PDF fallback)
2. Detects available format (LaTeX source or PDF)
3. Calls the appropriate converter (`convert_latex.py` or `convert_pdf_simple.py`)
4. Outputs structured Markdown to `papers/{ARXIV_ID}/{ARXIV_ID}.md`

All HTTP requests (curl), file extraction (tar), and directory creation (mkdir) are handled automatically.

### Automatic Source Detection and Fallback

The fetcher tries LaTeX source first, then PDF:
- **LaTeX source available**: Downloads `.tar.gz`, extracts to `papers/{ID}/source/`, converts with pandoc
- **PDF only**: Downloads PDF to `papers/{ID}/pdf/`, extracts text with pdfplumber

No manual intervention needed—the skill handles format detection and fallback automatically.

## Output Structure

Generated Markdown includes:
- Title, authors, and abstract
- Full paper content with section hierarchy
- Inline math: `$f(x) = x^2$`
- Display math: `$$\int_0^\infty e^{-x} dx = 1$$`
- Preserved LaTeX commands for complex formulas
- References section

Output location: `papers/{ARXIV_ID}/{ARXIV_ID}.md`

## PDF Conversion Scripts

Three specialized scripts for direct PDF conversion:

### convert_pdf_simple.py

Convert all pages as single-column layout.

```bash
uv run convert_pdf_simple.py paper.pdf -o output.md
```

### convert_pdf_double_column.py

Convert all pages as double-column layout (for academic papers).

```bash
uv run convert_pdf_double_column.py paper.pdf -o output.md
```

### convert_pdf_extract.py

Extract specific pages with optional double-column processing.

```bash
# Extract specific pages
uv run convert_pdf_extract.py paper.pdf --pages 1-5,10 -o output.md

# Extract with mixed column layouts
uv run convert_pdf_extract.py paper.pdf --pages 1-10 --double-column-pages 3-7 -o output.md
```

**Note:** `--double-column-pages` must be a subset of `--pages`. Invalid page ranges cause immediate error.

### Architecture

All three scripts share common conversion logic through `pdf_converter_lib.py`, ensuring consistent behavior while keeping each script focused on its specific use case.

## Advanced: Vision-Based PDF Conversion

For papers with complex mathematical formulas where text extraction fails, a vision-based approach is available as a manual fallback:

```bash
# Generate high-resolution images from PDF
python scripts/convert_pdf_with_vision.py paper.pdf --dpi 300 --columns 2
```

This creates page images (with optional column splitting) that can be read manually with Claude's vision capabilities for maximum accuracy. This is NOT part of the automatic workflow—use it only when automatic conversion produces poor results.

See [references/pdf-conversion.md](references/pdf-conversion.md) for details on vision-based conversion.

## Directory Structure

```
papers/
└── {ARXIV_ID}/
    ├── source/           # LaTeX source files (if available)
    ├── pdf/              # PDF file
    ├── {ARXIV_ID}.md     # Generated Markdown output
    └── figures/          # Extracted figures (if any)
```
