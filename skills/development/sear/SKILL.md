---
name: sear
description: Semantic search and RAG for documents. Use when user needs to index PDF/DOCX/text files, perform semantic search, extract relevant content from document corpuses, or build RAG applications. Supports multi-corpus search, GPU acceleration, line-level citations, and document conversion with OCR.
---

# SEAR: Semantic Enhanced Augmented Retrieval

## When to Use This Skill

Invoke SEAR when the user wants to:
- Search documents semantically (not just keyword matching)
- Build a RAG (Retrieval-Augmented Generation) application
- Convert PDF or DOCX files to searchable markdown
- Index code repositories, documentation, or knowledge bases
- Extract relevant content without LLM generation (pure retrieval)
- Search across multiple document collections (multi-corpus)
- Get line-level citations with exact source tracking

## Core Capabilities

### 1. Document Conversion
Convert PDF and DOCX files to LLM-optimized markdown:
```bash
# Basic conversion
sear convert document.pdf

# Custom output directory
sear convert report.docx --output-dir docs/

# OCR for scanned documents with language hints
sear convert scanned.pdf --force-ocr --lang heb+eng

# Keep original formatting (niqqud, etc.)
sear convert hebrew.pdf --no-normalize
```

**Features:**
- Smart OCR: Auto-detects text layer, falls back to OCR if needed
- Language support: Hebrew, English, mixed content (auto-detected)
- Token optimization: Removes niqqud, styling, formatting
- RAG-ready output: Metadata headers + page separators for citations

### 2. Indexing Documents
Create searchable FAISS indices from text files:
```bash
# Basic indexing
sear index document.txt my_corpus

# With GPU acceleration (5-10x faster on large datasets)
sear index large_doc.txt production_corpus --gpu

# Check GPU availability
sear gpu-info
```

**Index locations:** `faiss_indices/<corpus_name>/`

**Project Structure:** SEAR uses a standard Python src-layout:
- `src/sear/` - Main package directory
  - `cli.py` - CLI interface
  - `core.py` - Core library functions
  - `__init__.py` - Public API exports
- `src/doc_converter/` - Document conversion module
- `tests/` - Test suite
- `examples/` - Example code

### 3. Semantic Search with LLM
Search and get LLM-synthesized answers with citations:
```bash
# Basic search (uses local Ollama by default)
sear search "how does authentication work?" --corpus my_corpus

# With Anthropic Claude (higher quality)
export ANTHROPIC_API_KEY=sk-ant-xxx
sear search "explain the security model" --corpus my_corpus --provider anthropic

# Multi-corpus search
sear search "query" --corpus docs --corpus code --corpus wiki
```

**Output:** Synthesized answer with line-level citations: `[corpus_name] file.txt:42-45`

### 4. Content Extraction (No LLM)
Retrieve relevant chunks without generation (pure retrieval):
```bash
# Extract matching chunks
sear extract "security vulnerabilities" --corpus codebase

# Adjust similarity threshold (default: 0.30)
sear extract "query" --corpus docs --min-score 0.40

# Limit results
sear extract "query" --corpus docs --top-k 5
```

**Use case:** When you need raw content for further processing, not LLM answers.

## Typical Workflows

### Workflow 1: Process and Search PDFs
```bash
# Step 1: Convert PDF to markdown
sear convert research_paper.pdf

# Step 2: Index the converted markdown
sear index converted_md/research_paper.md research_corpus

# Step 3: Search with questions
sear search "what were the main findings?" --corpus research_corpus
```

### Workflow 2: Multi-Corpus Knowledge Base
```bash
# Index different sources
sear index documentation.txt docs_corpus
sear index codebase.txt code_corpus
sear index articles.txt articles_corpus

# Search across all corpuses
sear search "how to implement feature X?" \
  --corpus docs_corpus \
  --corpus code_corpus \
  --corpus articles_corpus
```

### Workflow 3: Extract Content for Analysis
```bash
# Extract relevant chunks for manual review
sear extract "security concerns" --corpus audit_corpus > security_findings.txt

# Use extracted content in further analysis
# (No LLM generation, just pure retrieval)
```

## Performance Optimization

### GPU Acceleration
- **Small corpuses (<500 chunks):** Use `--no-gpu` (CPU is faster)
- **Medium corpuses (500-10k chunks):** GPU provides 2-3x speedup
- **Large corpuses (>10k chunks):** GPU provides 5-10x speedup

```bash
# Let SEAR decide automatically (recommended)
sear index large.txt corpus

# Force GPU
sear index large.txt corpus --gpu

# Force CPU
sear index large.txt corpus --no-gpu
```

### Quality Filtering
SEAR uses empirical similarity thresholds (default: 0.30) to filter low-quality matches:
```bash
# Adjust threshold for stricter matching
sear search "query" --corpus docs --min-score 0.40

# Lower threshold for broader matching
sear search "query" --corpus docs --min-score 0.20
```

When results are insufficient (<2 matches), SEAR prompts for query refinement instead of generating answers from noise.

## LLM Provider Selection

### Local Ollama (Default - Zero Cost)
```bash
# Uses qwen2.5:0.5b by default
sear search "query" --corpus docs

# Fast (~5s), adequate quality, $0 cost
```

### Anthropic Claude (Higher Quality)
```bash
# Set API key
export ANTHROPIC_API_KEY=sk-ant-xxx

# Use Claude 3.5 Sonnet
sear search "query" --corpus docs --provider anthropic

# Better reasoning, structured output, ~10s, ~$0.01/query
```

## Corpus Management

```bash
# List all available corpuses
sear list

# Delete a corpus
sear delete corpus_name
```

## Best Practices

1. **Document Preparation:**
   - Convert PDFs/DOCX first: `sear convert` before indexing
   - For code repos: Use `gitingest` or concatenate files
   - Keep documents focused (better retrieval quality)

2. **Indexing Strategy:**
   - Use meaningful corpus names (e.g., `project_docs`, `codebase_v2`)
   - Separate different domains into different corpuses
   - Re-index when documents are updated

3. **Search Quality:**
   - Start with default threshold (0.30)
   - Use multi-corpus search for comprehensive coverage
   - Refine queries if results are insufficient

4. **GPU Usage:**
   - Don't force `--gpu` on small datasets
   - Let SEAR decide automatically
   - Monitor with `sear gpu-info`

5. **Cost Optimization:**
   - Use local Ollama for development/iteration
   - Use Anthropic Claude for production/critical analysis
   - Use `extract` command when you don't need LLM synthesis

## Architecture

```
Documents (PDF/DOCX/TXT)
    ↓
[src/doc_converter] ← PDF/DOCX → Markdown (with OCR)
    ↓
Text Files
    ↓
[Embedding: all-minilm via Ollama] ← 384-dimensional vectors
    ↓
[FAISS Index] ← CPU or GPU acceleration
    ↓
[Query Embedding]
    ↓
[Similarity Search] ← Quality filtering (threshold: 0.30)
    ↓
Top-k Relevant Chunks
    ↓
[LLM Synthesis: Ollama/Anthropic] ← Optional (skip for extract)
    ↓
Answer + Line-Level Citations
```

## Key Differentiators

**vs AWS NOVA/Titan Embeddings:**
- ✅ Zero cost (100% local)
- ✅ Complete document pipeline (convert → index → search)
- ✅ GPU acceleration (5-10x on large datasets)
- ✅ Line-level citations
- ✅ 100% offline/private
- ❌ Text-only (no multimodal)

**vs Traditional RAG:**
- ✅ Quality-aware filtering (empirical thresholds)
- ✅ Multi-corpus parallel search
- ✅ Content extraction without LLM
- ✅ 99% token reduction (retrieval-first)
- ✅ Deterministic retrieval (100%)

## Installation Requirements

SEAR must be installed in the user's environment:

```bash
# Basic installation
pip install -e .

# With document conversion (PDF/DOCX)
pip install -e ".[converter]"

# With GPU support
pip install -e ".[gpu]"

# With Anthropic Claude
pip install -e ".[anthropic]"

# Install everything
pip install -e ".[all]"

# Install Ollama models
ollama pull all-minilm
ollama pull qwen2.5:0.5b
```

## Common Issues

**Issue:** "ModuleNotFoundError: No module named 'doc_converter'"
**Solution:** Install converter dependencies: `pip install -e ".[converter]"`

**Issue:** GPU not detected
**Solution:** Check CUDA toolkit: `sear gpu-info`, install `faiss-gpu` if needed

**Issue:** Low-quality results
**Solution:** Adjust threshold: `--min-score 0.40` or refine query

**Issue:** Slow search on small corpus
**Solution:** Use CPU mode: `--no-gpu`

## Example Use Cases

1. **Legal Document Review:** Convert contracts to markdown, index, extract clauses
2. **Code Documentation:** Index codebase + docs, semantic search for implementations
3. **Research Analysis:** Convert papers to markdown, multi-paper semantic search
4. **Knowledge Management:** Index company docs, wiki, policies - searchable knowledge base
5. **Compliance Auditing:** Extract policy-relevant sections without reading everything

## Additional Resources

- Repository: https://github.com/Guard8-ai/SEAR
- Documentation: See README.md, INSTALL.md, GPU_SUPPORT.md
- Benchmarks: See BENCHMARK_RESULTS.md
- Examples: See examples/ directory
