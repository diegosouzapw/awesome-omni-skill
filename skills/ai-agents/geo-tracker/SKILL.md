---
name: geo-tracker
description: Track and optimize brand visibility across AI search engines (ChatGPT, Perplexity, Gemini, Google AI Overview, Claude). Use when monitoring brand mentions in AI answers, running GEO audits, comparing brand vs competitors in AI responses, or optimizing content for generative engine citation. Supports single queries, batch audits, and scheduled monitoring.
---

# GEO Tracker

Track how AI engines mention (or ignore) a brand. Query multiple AI-powered search engines, extract brand mentions, score visibility, and generate actionable optimization reports.

## Quick Start

### Single Query Check

```bash
python3 scripts/geo_query.py --brand "EZsite.ai" --query "best AI website builder" --engines chatgpt,perplexity,gemini
```

### Full Audit (batch prompts)

```bash
python3 scripts/geo_audit.py --brand "EZsite.ai" --prompts references/prompts.txt --engines all --output report.md
```

### Competitor Comparison

```bash
python3 scripts/geo_query.py --brand "EZsite.ai" --competitors "Wix,Squarespace,Framer" --query "best website builder for small business"
```

## Dependencies

Install required Python packages (one-time):

```bash
pip3 install openai anthropic google-generativeai
```

Or create a virtual environment:

```bash
python3 -m venv venv
source venv/bin/activate
pip install openai anthropic google-generativeai
```

Set API keys as environment variables:

```bash
export OPENAI_API_KEY="sk-..."
export PERPLEXITY_API_KEY="..."
export GOOGLE_API_KEY="..."
export ANTHROPIC_API_KEY="sk-ant-..."
```

## How It Works

1. Takes a brand name + search prompts
2. Queries AI engines via their APIs or web interfaces
3. Extracts: mentions, citations, sentiment, positioning
4. Scores visibility (0-100) per engine and overall
5. Generates optimization recommendations

## Engines Supported

| Engine | Method | API Key Env Var |
|--------|--------|-----------------|
| ChatGPT | OpenAI API | `OPENAI_API_KEY` |
| Perplexity | Perplexity API | `PERPLEXITY_API_KEY` |
| Gemini | Google AI API | `GOOGLE_API_KEY` |
| Claude | Anthropic API | `ANTHROPIC_API_KEY` |
| Google AI Overview | web_search tool | (none) |

At minimum, configure one API key. More engines = better coverage.

## Visibility Score

- 0-20: Invisible — AI doesn't know the brand
- 21-40: Low — occasional mentions, not recommended
- 41-60: Moderate — mentioned but not top choice
- 61-80: Strong — frequently cited/recommended
- 81-100: Dominant — top recommendation across engines

## Output Format

Reports include:
- Per-engine mention count and context
- Visibility score breakdown
- Competitor comparison matrix
- Top optimization recommendations
- Source prompts that triggered (or missed) mentions

## Prompt Library

Edit `references/prompts.txt` — one prompt per line. These are the queries sent to AI engines.

Example prompts for a website builder brand:
```
best AI website builder
how to build a website without coding
website builder comparison 2025
best website builder for small business
AI-powered web design tools
```

## Optimization Tips Reference

See `references/geo-optimization.md` for content optimization strategies to improve AI visibility.

## Scheduling

Use OpenClaw cron to run daily/weekly audits:
```
Schedule a daily GEO audit for EZsite.ai at 9am
```

The agent will run the audit and report findings.
