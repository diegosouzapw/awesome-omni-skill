---
name: web-search
description: Web search and content extraction toolkit. Use for searching documentation, facts, current information, or extracting readable content from URLs. Supports multiple providers (ddgs keyless, brave_api with key), caching, and safe defaults. Prefer this over browser-tools when no interaction is needed.
---

# Web Search Toolkit

Search the web and extract readable content. Stable CLI with JSON output for agents.

## Setup

```bash
cd {baseDir}
uv sync  # Install dependencies (once)
```

Optional: Set `BRAVE_API_KEY` for better search reliability (ddgs is keyless but flaky).

## Commands

### Search

```bash
{baseDir}/.venv/bin/wstk search "query"                    # Default (10 results)
{baseDir}/.venv/bin/wstk search "query" -n 5 --plain       # URLs only, one per line
{baseDir}/.venv/bin/wstk search "query" --json             # Machine-readable
{baseDir}/.venv/bin/wstk search "query" --time-range w     # Last week
{baseDir}/.venv/bin/wstk search "site:docs.python.org asyncio"  # Site-scoped
```

Key flags:
- `-n, --max-results <N>` — Number of results (default: 10)
- `--time-range <d|w|m|y>` — Filter by recency
- `--provider <ddgs|brave_api|auto>` — Search provider
- `--plain` — Output URLs only (for piping)
- `--json` — Structured output

### Pipeline (search → extract)

```bash
{baseDir}/.venv/bin/wstk pipeline "python asyncio tutorial" --json
{baseDir}/.venv/bin/wstk pipeline "python asyncio tutorial" --plan --plain
```

Key flags:
- `--top-k <N>` — Search results to consider
- `--extract-k <N>` — Number of results to extract
- `--plan` — Return candidates without fetching
- `--method <http|browser|auto>` — Extraction method (default: http)

### Extract (fetch + extract readable content)

```bash
{baseDir}/.venv/bin/wstk extract https://example.com --plain     # Markdown output
{baseDir}/.venv/bin/wstk extract https://example.com --text      # Plain text
{baseDir}/.venv/bin/wstk extract https://example.com --json      # Full metadata
{baseDir}/.venv/bin/wstk extract ./local-file.html --plain       # From file
```

Key flags:
- `--markdown` / `--text` / `--both` — Output format
- `--strategy <auto|readability|docs>` — Extraction strategy
- `--max-chars <N>` — Truncate output
- `--allow-domain <domain>` — Restrict to specific domains (safety)

### Fetch (raw HTTP, no extraction)

```bash
{baseDir}/.venv/bin/wstk fetch https://example.com --json        # Metadata + status
{baseDir}/.venv/bin/wstk fetch https://example.com --plain       # Path to cached body
```

### List providers

```bash
{baseDir}/.venv/bin/wstk providers --plain
```

## Decision Guide

- `search` when you need discovery or candidate URLs.
- `pipeline` when you want a one-shot search → extract bundle.
- `fetch` when you need HTTP metadata or the cached body path (no extraction).
- `extract` when you want readable content from a URL or local HTML.
- `render` when a page is JS-only or blocked (or use `extract --method browser` for one-step extraction).

## Common Patterns

**Search → extract top result:**
```bash
url=$({baseDir}/.venv/bin/wstk search "python asyncio tutorial" --plain | head -1)
{baseDir}/.venv/bin/wstk extract "$url" --plain --max-chars 8000
```

**Search with JSON for programmatic use:**
```bash
{baseDir}/.venv/bin/wstk search "openai api reference" --json | jq '.data.results[0].url'
```

**Safe extraction (restrict domains):**
```bash
{baseDir}/.venv/bin/wstk extract https://docs.python.org/3/library/asyncio.html \
  --allow-domain docs.python.org --plain
```

## Output Formats

- `--plain` — Stable text for piping (URLs for search, content for extract)
- `--json` — Structured envelope: `{ "ok": bool, "data": {...}, "error": {...} }`
- Default — Human-readable with colors

## Agent Defaults

- Default to `--json` in agent wrappers; parse `ok`, `error.code`, and `warnings`.
- Surface concise diagnostics by relaying `error.message` and `error.details.reason` when `ok=false`.
- Use `--plain` only for piping, and add `--no-input` for non-interactive runs.
- Consider `--redact` when handling sensitive URLs or content.

## Exit Codes

- `0` — Success
- `1` — Runtime failure (network, provider error)
- `2` — Invalid usage
- `3` — Not found / empty result
- `4` — Blocked / access denied
- `5` — Needs JS rendering (page is JS-only)

## Global Flags

- `--timeout <seconds>` — Network timeout
- `--no-cache` — Disable caching
- `--fresh` — Bypass cache reads (still writes)
- `--quiet` — Minimal output
- `--verbose` — Debug diagnostics to stderr
- `--policy <standard|strict|permissive>` — Safety defaults

## References

- `references/troubleshooting.md` — 403/JS-only guidance and advanced fetch flags.
- `references/providers.md` — Provider selection and privacy notes.
- `docs/claude-code.md` — Claude Code wrapper usage.

## When to Use

- Searching for documentation or API references
- Looking up facts or current information
- Extracting content from known URLs
- Any task requiring web search without interactive browsing

Prefer `browser-tools` when you need: JS interaction, form filling, clicking, or visual inspection.
