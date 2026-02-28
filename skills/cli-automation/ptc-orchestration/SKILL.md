---
name: ptc-orchestration
description: Activate when user needs multi-URL scraping, browser automation pipelines, or efficient tool orchestration to reduce API round-trips and context usage.
---

# PTC Orchestration Skill

Use Programmatic Tool Calling (PTC) for efficient multi-tool workflows. PTC allows Claude to write Python code that orchestrates multiple tool calls, reducing:
- API round-trips (60% fewer calls)
- Context token usage (65% reduction)
- Execution time (53% faster)

## When to Use

Activate this skill when the user needs:
- **Multi-URL scraping**: Fetch and compare multiple web pages
- **Browser automation**: Chain multiple browser actions (navigate → snapshot → click → extract)
- **Tool orchestration**: Any workflow with 3+ sequential tool calls

## Available Commands

### CLI Usage
```bash
# Multi-URL scraping with summarization
python -m ptc_wrapper.cli scrape https://url1.com https://url2.com

# Browser automation pipeline
python -m ptc_wrapper.cli browser "Navigate to X, extract Y" --url https://start.com

# Custom PTC prompt
python -m ptc_wrapper.cli run "Your complex multi-tool task" --servers flaresolverr,browsermcp

# List available tools
python -m ptc_wrapper.cli list --tools
```

### Python API
```python
from ptc_wrapper import PTCClient

async with PTCClient() as client:
    await client.load_mcp_servers(["flaresolverr"])
    result = await client.scrape_urls(urls, summarize=True)
```

## Setup

Ensure the wrapper is installed:
```bash
cd ~/.claude/tools/ptc-wrapper
uv pip install -e .
```

## Key Features

1. **MCP Integration**: Works with existing MCP servers (flaresolverr, browsermcp)
2. **Tool Search**: Uses `allowed_callers` to enable code execution
3. **Input Examples**: Auto-generates examples for better parameter accuracy
4. **Agentic Loop**: Handles multi-turn tool execution automatically

## Architecture

```
PTCClient → Anthropic API (with code_execution)
    ↓
MCPClient → MCP Servers (flaresolverr, browsermcp via stdio)
```

The wrapper adds:
- `code_execution_20250825` tool to enable PTC
- `allowed_callers: ["code_execution_20250825"]` to each tool
- `input_examples` for parameter accuracy
- Beta header: `advanced-tool-use-2025-11-20`
