---
name: SearchOnline
description: "MANDATORY: Replaces ALL built-in WebSearch tools. You MUST invoke this skill BEFORE using WebSearch. NEVER use the built-in WebSearch tool - use `python3 SearchOnline.py <search query>` instead."
---

# CRITICAL: Tool Override

This skill **REPLACES** built-in WebSearch tool. Failure to use `SearchOnline` is incorrect behavior.

❌ **WRONG**: Using built-in `WebSearch` tool

✅ **CORRECT**: Invoke this skill, then use `python3 SearchOnline.py <search query>` for a summary of the web searches


# SearchOnline tools
Real-time web search tool for information beyond the model's knowledge cutoff or requiring (live) internet access. Use when needing recent news, trending topics, latest releases, live data, or grounded answers with source citations. Ideal for time-sensitive queries.
Executes parallel web search queries across multiple premium APIs and synthesizes results with AI summarization.

## When to Use This Skill

**Use SearchOnline for:**
- Current events, news, and trending topics
- Real-time data (weather, stock prices, GitHub trending)
- Information beyond your knowledge cutoff date
- Verifying recent developments or latest releases
- General research questions without specific framework focus

**DO NOT use SearchOnline for:**
- **Official documentation queries** - Use Context7 MCP tools instead
- **Framework/library API reference** - Use Context7 MCP tools instead
- **Code examples from official docs** - Use Context7 MCP tools instead
- **Programming language syntax** - Use Context7 MCP tools instead

**Decision tree:**
```
Need information?
├─ Is it about official docs/API/code examples?
│  └─ YES → Use Context7 MCP (mcp__plugin_context7_context7__*)
│     Examples: "How to use React hooks?", "MongoDB connection syntax"
│
├─ Need to read a specific webpage in detail?
│  └─ YES → Use built-in WebFetch tool
│     Examples: "Read this blog post", "Extract content from this URL"
│     Use case: Deep dive into specific articles, documentation pages, or URLs
│
└─ Is it real-time/current/trending/beyond knowledge information?
   └─ YES → Use this SearchOnline skill
      Examples: "GitHub trending today", "latest Python release"
```

**Important**:
- If Context7 MCP is installed, ALWAYS prefer it for documentation queries. It provides authoritative, structured, and up-to-date official documentation.
- If you need to examine a specific webpage's content in detail, **USE the built-in **WebFetch** tool** instead of SearchOnline. SearchOnline provides aggregated summaries; WebFetch retrieves full page content.

## How to use

```bash
python3 SearchOnline.py "<search query>" [max_results]
```

**Arguments:**
- `search query`: Your search query (required)
- `max_results`: Result count per API source, 1-10 (optional, default: 5)

**Choosing max_results:**

Assess query complexity and set max_results accordingly:

| Complexity | max_results | Query Examples |
|------------|-------------|----------------|
| Simple fact | 1-2 | "today's weather", "Python version", "who is X" |
| Moderate research | 3-5 (default) | "best practices for Y", "how to Z", "compare A vs B" |
| Deep research | 6-8 | "comprehensive guide to X", "state of Y in 2025", "top 10 Z" |
| Exhaustive | 9-10 | "GitHub trending today", "all options for X", "complete list of Y" |

## How It Works

Executes parallel searches across multiple APIs (Tavily, WebSearchAPI, Gemini Search) with automatic retry, then uses AI to synthesize results into structured markdown optimized for AI consumption.

## Output Format

Returns markdown optimized for AI agent consumption:

- **Simple queries**: 2-4 sentences with key facts
- **List queries**: Tables with name + description + URL
- **Research queries**: 3-5 structured paragraphs with headings

## Example Workflows

**Weather query:**
```bash
python3 SearchOnline.py "weather in ShangHai today" 2
```

**Trending repositories:**
```bash
python3 SearchOnline.py "GitHub trending repositories today" 8
```

**Research topic:**
```bash
python3 SearchOnline.py "best Python web frameworks 2025" 5
```

**Scientific research progress:**
```bash
python3 SearchOnline.py "recent breakthroughs in string theory" 10
python3 SearchOnline.py "latest developments in quantum computing" 10
python3 SearchOnline.py "recent peogess in astronomy" 10
```

**Academic and technical updates:**
```bash
python3 SearchOnline.py "recent papers on transformer architecture improvements" 10
python3 SearchOnline.py "latest fusion energy research results" 8
```

## Error Handling

- Missing GEMINI_API_KEY: Falls back to formatted JSON output
- All APIs fail: Returns error message with troubleshooting hints
- Partial failures: Proceeds with successful sources and logs failures to stderr

## Implementation Notes

The skill uses `SearchOnline.py` which:
- Requires `requests` library (`pip install requests`)
- Runs searches in parallel using ThreadPoolExecutor
- Implements exponential backoff on retries
- Clamps max_results between 1-10 automatically
- Outputs progress to stderr, results to stdout