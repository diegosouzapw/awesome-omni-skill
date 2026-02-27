---
name: gpt-researcher
description: Run GPT-Researcher multi-agent deep research framework locally using OpenAI GPT-5.2. Replaces ChatGPT Deep Research with local control. Researches 100+ sources in parallel, provides comprehensive citations. Use for Phase 3 industry/technical research or comprehensive synthesis. Takes 6-20 min depending on report type. Supports multiple LLM providers.
---

# GPT-Researcher Skill

Use this skill to run GPT-Researcher's multi-agent deep research framework locally with OpenAI's GPT-5.2 model.

## What is GPT-Researcher?

GPT-Researcher is an autonomous multi-agent research framework that:
- Uses **parallel agent execution** for faster research
- **Researches 100+ sources** across the web
- Provides **comprehensive citations** and source validation
- Benchmarks **competitively with ChatGPT Deep Research and Claude Research**
- Runs **locally** with full control over configuration

**Default Model:** OpenAI GPT-5.2 (latest flagship model, 2025)

**GPT-5.2 Highlights:**
- Best general-purpose model for complex reasoning and agentic tasks
- Improved instruction following and accuracy over GPT-5.1
- Enhanced code generation and tool calling
- Better context management and token efficiency
- Knowledge cutoff: August 2025

**Carnegie Mellon Benchmark (DeepResearchGym, May 2025):**
GPT-Researcher **outperformed** Perplexity, OpenAI Deep Research, and other tools on:
- Citation quality
- Report quality
- Information coverage

## When to Use This Skill

Use GPT-Researcher for deep research tasks in the podcast episode workflow:

1. **Phase 3: Industry & Technical Research** (replaces ChatGPT Deep Research browser automation)
2. **Phase 3: Comprehensive Synthesis** (alternative to Claude Deep Research)
3. **Any multi-dimensional research** requiring parallel information gathering

**Advantages over browser automation:**
- No Chrome/browser required
- Fully scriptable and reproducible
- Choose any LLM provider (OpenAI, Anthropic, etc.)
- Run in background or CI/CD pipelines
- Complete control over configuration

## Installation

This skill requires `uv`, a fast Python package manager:

```bash
# Install uv (if not already installed)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install dependencies
cd /Users/valorengels/src/research/podcast/tools
uv pip install gpt-researcher langchain-openai python-dotenv
```

## Configuration

API keys are stored in `/Users/valorengels/.env` and auto-loaded via `~/.zshenv` for all shells.

**Required for default:**
- **OPENAI_API_KEY** - For GPT-5.2, GPT-5.2-Pro, etc.

**Optional providers:**
- **OPENROUTER_API_KEY** - Unified access to 400+ models
- **ANTHROPIC_API_KEY** - Claude Opus, Sonnet
- **XAI_API_KEY** - Grok models

## Usage

### Basic Usage (GPT-5.2)

```bash
cd /Users/valorengels/src/research/podcast/tools
uv run python gpt_researcher_run.py "Your research prompt here"
```

This uses **GPT-5.2** by default - OpenAI's latest and most capable general-purpose model.

### Read Prompt from File

```bash
cd /Users/valorengels/src/research/podcast/tools
uv run python gpt_researcher_run.py --file ../episodes/YYYY-MM-DD-slug/prompt.txt
```

### Save to File

```bash
uv run python gpt_researcher_run.py "prompt" --output results.md
```

### Specify Different Model

```bash
# Use GPT-5.2-Pro for harder thinking (more compute)
uv run python gpt_researcher_run.py "prompt" --model openai:gpt-5.2-pro

# Use GPT-5-Mini for cost-optimized research
uv run python gpt_researcher_run.py "prompt" --model openai:gpt-5-mini

# Use Anthropic Claude Opus 4
uv run python gpt_researcher_run.py "prompt" --model anthropic:claude-opus-4

# Use OpenRouter for any model
uv run python gpt_researcher_run.py "prompt" --model openrouter/anthropic/claude-opus-4.5
```

### Report Types

```bash
# Standard research report (default, 6-10 min)
uv run python gpt_researcher_run.py "prompt" --report-type research_report

# Detailed comprehensive report (10-20 min)
uv run python gpt_researcher_run.py "prompt" --report-type detailed_report

# Quick report (3-5 min, fewer sources)
uv run python gpt_researcher_run.py "prompt" --report-type quick_report
```

## Integration with Podcast Workflow

### Phase 3: Industry & Technical Research

**Replaces:** ChatGPT Deep Research browser automation

**Use Case:** Industry reports, technical documentation, case studies

```bash
cd podcast/tools
uv run python gpt_researcher_run.py --file ../episodes/YYYY-MM-DD-slug/phase3_prompt.txt \
    --model openai:gpt-5.2 \
    --report-type research_report \
    --output ../episodes/YYYY-MM-DD-slug/research-results-industry.md
```

**Expected time:** 6-10 minutes
**Output:** Research report with 50-100+ sources, industry and technical focus

### Phase 3: Comprehensive Synthesis

**Use Case:** Deep multi-dimensional research with comprehensive synthesis

```bash
cd podcast/tools
uv run python gpt_researcher_run.py --file ../episodes/YYYY-MM-DD-slug/phase3_prompt.txt \
    --model openai:gpt-5.2 \
    --report-type detailed_report \
    --output ../episodes/YYYY-MM-DD-slug/research-results-comprehensive.md
```

**Expected time:** 10-20 minutes
**Output:** Comprehensive report with 100+ sources, multi-agent synthesis

### Using GPT-5.2-Pro for Complex Problems

For particularly challenging research that requires deeper thinking:

```bash
cd podcast/tools
uv run python gpt_researcher_run.py --file ../episodes/YYYY-MM-DD-slug/prompt.txt \
    --model openai:gpt-5.2-pro \
    --report-type detailed_report \
    --output ../episodes/YYYY-MM-DD-slug/research-results-pro.md
```

**Expected time:** 15-25 minutes
**Output:** Highest quality research with extended reasoning

## Output Format

The script outputs markdown-formatted research with:
- **Header:** Date, model, prompt
- **Research report:** Comprehensive findings with structure
- **Citations:** Inline citations with source URLs
- **Sources:** List of sources researched

Example output structure:
```markdown
# GPT-Researcher Results

**Date:** 2025-12-14 14:30

**Model:** openai:gpt-5.2

**Prompt:** Research early childhood educator burnout interventions

---

## Executive Summary
[Comprehensive overview]

## Key Findings
[Detailed findings with citations]

## Methodology Considerations
[Study quality notes]

## Sources
[List of 100+ sources with URLs]
```

## Why GPT-5.2 for Research?

OpenAI's GPT-5.2 is their latest flagship model optimized for:
- **Complex reasoning** - Multi-step analysis and synthesis
- **Research tasks** - Information gathering and validation
- **Agentic workflows** - Tool calling and context management
- **Accuracy** - Improved instruction following and token efficiency
- **Code generation** - Especially front-end UI creation
- **Multimodality** - Enhanced vision capabilities

This makes it ideal for deep research compared to previous models.

**Model comparison:**
- **gpt-5.2:** Best for complex reasoning and comprehensive research
- **gpt-5.2-pro:** Best for hardest problems requiring extended thinking
- **gpt-5-mini:** Best for cost-optimized research
- **claude-opus-4:** Best for synthesis and writing quality

## Comparison: GPT-Researcher vs ChatGPT Deep Research

| Feature | GPT-Researcher (Local) | ChatGPT Deep Research (Browser) |
|---------|------------------------|--------------------------------|
| **Model** | GPT-5.2 (latest) | ChatGPT (whatever's enabled) |
| **Control** | Full local control | Browser automation |
| **Setup** | API key only | Chrome + auth + browser automation |
| **Reliability** | High (API) | Medium (UI changes) |
| **Sources analyzed** | 100+ | 25-50 |
| **Processing time** | 6-20 min | 5-10 min |
| **Cost** | Pay-per-use (~$0.27-2) | $200/mo subscription |
| **Headless** | Yes | No (needs browser) |
| **Maintenance** | Low | High (UI changes) |
| **Benchmark** | CMU winner | Commercial |

**Decision:** GPT-Researcher with GPT-5.2 replaces ChatGPT Deep Research browser automation.

## Advanced Usage

### Environment Variables

GPT-Researcher uses these environment variables (set in `.env`):

```bash
# Required: At least one API key
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
OPENROUTER_API_KEY=sk-or-...
XAI_API_KEY=...

# Optional: Override via --model flag
FAST_LLM=openai:gpt-5.2          # Quick tasks
SMART_LLM=openai:gpt-5.2         # Deep analysis
STRATEGIC_LLM=openai:gpt-5.2     # Planning

# Optional: Search provider
RETRIEVER=tavily  # Default (best quality)
# or: duckduckgo (free fallback)
```

### Custom Model Selection

```bash
# Latest OpenAI GPT-5 family (2025)
uv run python gpt_researcher_run.py "prompt" --model openai:gpt-5.2          # Best for research
uv run python gpt_researcher_run.py "prompt" --model openai:gpt-5.2-pro      # Harder thinking
uv run python gpt_researcher_run.py "prompt" --model openai:gpt-5-mini       # Cost-optimized
uv run python gpt_researcher_run.py "prompt" --model openai:gpt-5-nano       # High-throughput

# Legacy OpenAI models
uv run python gpt_researcher_run.py "prompt" --model openai:o1               # Legacy reasoning
uv run python gpt_researcher_run.py "prompt" --model openai:gpt-4o           # Legacy multimodal

# Anthropic Claude
uv run python gpt_researcher_run.py "prompt" --model anthropic:claude-opus-4
uv run python gpt_researcher_run.py "prompt" --model anthropic:claude-sonnet-4

# Via OpenRouter (single API key for all)
uv run python gpt_researcher_run.py "prompt" --model openrouter/openai/gpt-5.2
uv run python gpt_researcher_run.py "prompt" --model openrouter/anthropic/claude-opus-4.5
uv run python gpt_researcher_run.py "prompt" --model openrouter/x-ai/grok-4
```

## Troubleshooting

### Error: "No API keys found"
- Check `.env` files exist in root or `podcast/tools/`
- Ensure `OPENAI_API_KEY` is set for default GPT-5.2 model
- Verify `.env` format: `KEY=value` (no spaces around `=`)

### Error: "gpt-researcher not installed"
- Run: `cd podcast/tools && uv pip install gpt-researcher langchain-openai python-dotenv`
- Or ensure you're using: `uv run python gpt_researcher_run.py` (auto-installs dependencies)

### Research times out or fails
- Try `--report-type quick_report` for faster results
- Check API key has sufficient credits
- Verify OpenAI API key is valid
- Use `--model openai:gpt-5-mini` for faster/cheaper alternative

### Model not found
- For OpenRouter models, use format: `openrouter/provider/model`
- Check model names at https://openrouter.ai/models
- For native providers, use format: `provider:model`

### GPT-5.2 model errors
- Ensure you have access to GPT-5.2 in your OpenAI account
- Fallback to `--model openai:gpt-5-mini` if GPT-5.2 unavailable
- Check OpenAI API status page

## Example Commands

**Basic research with GPT-5.2:**
```bash
uv run python gpt_researcher_run.py "Research quantum computing applications in healthcare"
```

**From file with output:**
```bash
uv run python gpt_researcher_run.py \
  --file research-prompt.txt \
  --output results.md
```

**Industry research (typical Phase 3):**
```bash
uv run python gpt_researcher_run.py \
  --file ../episodes/episode-dir/prompt.txt \
  --model openai:gpt-5.2 \
  --report-type research_report \
  --output ../episodes/episode-dir/research-industry.md
```

**Hardest problems with GPT-5.2-Pro:**
```bash
uv run python gpt_researcher_run.py \
  --file prompt.txt \
  --model openai:gpt-5.2-pro \
  --report-type detailed_report \
  --output results-pro.md
```

**Cost-optimized with GPT-5-Mini:**
```bash
uv run python gpt_researcher_run.py \
  --file prompt.txt \
  --model openai:gpt-5-mini \
  --report-type quick_report \
  --output results-mini.md
```

**Comprehensive with Claude:**
```bash
uv run python gpt_researcher_run.py \
  --file prompt.txt \
  --model anthropic:claude-opus-4 \
  --report-type detailed_report \
  --output results-comprehensive.md
```

## Notes

- **Default model:** OpenAI GPT-5.2 (latest flagship, 2025)
- **Processing time:** Budget 6-20 minutes for comprehensive research
- **API costs:** Typically $0.27-2 per research session (varies by model and sources)
- **Quality:** Competitive with ChatGPT Deep Research on benchmarks
- **Local execution:** Runs on your machine, full control over configuration
- **No browser required:** Pure API-based, works in any environment
- **Replaces:** ChatGPT Deep Research browser automation (deprecated)
- **Knowledge cutoff:** GPT-5.2 has August 2025 cutoff (most current)

## Further Reading

- [OpenAI GPT-5.2 Documentation](https://platform.openai.com/docs/guides/latest-model)
- [GPT-Researcher Framework](https://docs.gptr.dev/)
- [Carnegie Mellon Benchmark Results](https://github.com/assafelovic/gpt-researcher)
