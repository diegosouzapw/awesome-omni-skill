---
name: web-research
description: Perform web research using OpenAI APIs. Fast mode uses gpt-5-search-api for quick lookups. Normal/deep modes use o3-deep-research model for comprehensive multi-step research with code interpreter. Invoke when user needs current web information or thorough research on a topic.
---

# Web Research Skill

Perform web research at three depth levels using OpenAI's APIs.

## Choosing the Right Depth

The key question: **Are you retrieving information or exploring a topic?**

### Fast (10-60 sec) — Retrieval

Use when you'd normally Google something, open a few links, and get your answer. You generally know what you're looking for; you just need current data or quick verification.

**Good for:**
- Current facts (prices, dates, events)
- Quick verifications ("Does X support Y?")
- Simple lookups where you know the answer exists
- Low-stakes decisions

**Examples:**
- "What is the current price of Bitcoin?"
- "What version of Python does Django 5.0 require?"
- "When is the next Apple event?"

### Normal (2-6 min) — Moderate Research

Use when you need more than a quick lookup but don't need exhaustive coverage. Good for comparisons, how-to questions, and understanding a topic at a moderate depth.

**Good for:**
- Feature comparisons (without needing every detail)
- How-to guides and best practices
- Understanding a topic you're somewhat familiar with
- Questions where you want synthesized information, not just raw facts

**Examples:**
- "What are the best practices for Python async programming in 2026?"
- "Compare Tailwind CSS vs vanilla CSS for a small project"
- "How do I set up GitHub Actions for a Python project?"

### Deep (6-14 min) — Exploratory Research

Use when you're genuinely exploring—you don't have certainty, the topic is niche, or you need the model to follow leads and check multiple sources. Also use when your research might require data analysis (reading PDFs, spreadsheets, doing calculations).

**Good for:**
- Niche or specialized topics
- Multi-faceted questions requiring synthesis
- Research that needs data analysis (trends, comparisons over time)
- Critical decisions where you want thorough source-checking
- Topics where information might be in PDFs or require calculations

**Examples:**
- "Compare US-SK105 Midea Wi-Fi dongle vs ESPHome for Carrier mini-split Home Assistant integration. Include compatibility, setup reliability, and reported issues."
- "Analyze electricity price trends with Peco Electric over the last 10 years"
- "Research the economic impact of semaglutide on global healthcare systems with specific figures and statistics"

## Structuring Your Query

Unlike ChatGPT's Deep Research (which asks clarifying questions), the API expects **fully-formed prompts**. The model won't ask for clarification—it just starts researching.

**Tips for better results:**
- State your goal explicitly ("I'm trying to decide between X and Y for Z use case")
- Include what you already know or have tried
- Specify constraints (budget, timeline, technical requirements)
- Ask for specific deliverables ("Include a comparison table", "List pros and cons")
- For deep research, mention if you need data analysis or source verification

## Configuration

| Depth | Model | Time | Max Tool Calls |
|-------|-------|------|----------------|
| **fast** | gpt-5-search-api | 10-60 sec | — |
| **normal** | o3-deep-research | 2-6 min | 25 |
| **deep** | o3-deep-research | 6-14 min | unlimited |

**Note:** `o4-mini-deep-research` is available as a faster/cheaper alternative to o3, but produces lower quality output.

## Usage

**Important:** Always quote paths containing backslashes or spaces.

```bash
# Fast lookup (default) — a sentence or two
cd "<skill-directory>" && uv run research.py "What is the current price of Bitcoin?"

# Normal research — roughly a paragraph of context
cd "<skill-directory>" && uv run research.py -d normal "I'm building a FastAPI app and trying to decide on an async database approach. What are the current best practices for async database connections with SQLAlchemy 2.0? I'm particularly interested in connection pooling, session management, and whether to use encode/databases or native SQLAlchemy async."

# Deep research — two paragraphs of detailed context
cd "<skill-directory>" && uv run research.py -d deep "I have a Carrier 40MHHQ09 mini-split (which is a rebadged Midea unit) and want to integrate it with Home Assistant. I've seen mentions of the US-SK105 Midea Wi-Fi dongle and ESPHome-based solutions but I'm not sure which approach is more reliable or if they even work with Carrier-branded units. [...]

Compare these options and include: (1) compatibility confirmation for Carrier 40MHH series, (2) Midea AC LAN HACS integration setup and reliability, (3) ESPHome alternatives, (4) USB port location, and (5) whether the solution reads actual unit state vs just sending commands. [...]"
```

## Arguments

- `--depth` / `-d`: Research depth - `fast`, `normal`, or `deep` (default: `fast`)
- `--no-save`: Don't save results to `~/research/`
- `query` (required): The research question or topic

## Output

Returns markdown-formatted results with:
- Main content answering the query
- Inline citations where applicable
- Source URLs listed at the end

## Saved Research

All research results are automatically saved to `~/research/` as markdown files. Each file contains YAML frontmatter (working directory, date, depth, model, metrics) followed by the original query and the full response with sources.

**Before running new research**, check if relevant past research already exists — especially for normal/deep depths which are expensive. Search by keyword or by project directory:

```bash
# Find past research containing a keyword
grep -rl "keyword" ~/research/

# Find past research done from the current project directory
grep -rl "directory: <current-working-directory>" ~/research/

# Read a past result
cat ~/research/2026-02-04_some-query.md
```

If a relevant past result exists, read it and present it to the user instead of re-running the research.

## Credentials

The script looks for `OPENAI_API_KEY` in:
1. Local `.env` file in current directory
2. User home `~/.env` file
3. Skill directory `.env` file
4. System environment variables
