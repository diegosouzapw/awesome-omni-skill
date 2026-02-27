---
name: browser-fetch
description: Delegate browser automation to a lightweight subagent (Haiku) to reduce context consumption. Also provides web clipping (HTML→Markdown) via clipper.
---

# browser-fetch

Delegate browser automation to a cost-effective subagent, preserving the main agent's context window.

## Why This Skill?

- **Problem**: `agent-browser snapshot` can consume 10,000+ tokens on complex pages
- **Solution**: Haiku subagent processes snapshots and returns only summaries (~100 tokens)
- **Result**: 90%+ context reduction while maintaining full browser capabilities

## Prerequisites

### Install agent-browser

```bash
npm install -g agent-browser
agent-browser install
```

### Install clipper

```bash
npm install -g @philschmid/clipper
```

Uses [Mozilla Readability](https://github.com/mozilla/readability) + [Turndown](https://github.com/mixmark-io/turndown) internally — same stack as Obsidian Web Clipper.

### Windows Workaround

On Windows (Git Bash), the daemon may not auto-start. Run manually:

```bash
node "$(npm root -g)/agent-browser/dist/daemon.js" &
sleep 3
```

## Basic Flow

```
Main Agent                        Haiku Subagent
    │                                   │
    │  Task: "Open URL, summarize"      │
    ├──────────────────────────────────►│
    │                                   │  1. agent-browser open
    │                                   │  2. snapshot → save to file
    │                                   │  3. build summary
    │   "Page summary + key refs"       │
    │◄──────────────────────────────────┤
    │                                   │
    │  Task: "Click @e5, summarize"     │
    ├──────────────────────────────────►│
    │                                  ...
```

## Snapshot Options

| Option | Use Case |
|--------|----------|
| `-i` | Interactive elements only (default) |
| `-i -c` | Compact output (recommended for complex pages) |
| `-i -d 2` | Limit depth (deeply nested pages) |
| `-i -s "main"` | Scope to CSS selector |

## Summary Format (Critical)

Subagent must return **structured summaries**, not raw element lists.

### Good Example

```
Product listing page (127 items displayed)

Structure:
- Filters: category (@e12), price range (@e18), search (@e24)
- Sort: date (@e30), popularity (@e31)

Sample items:
- "Product A" $29.99 (@e45)
- "Product B" $49.99 (@e52)
- "Product C" $19.99 (@e58)

Ready for instructions.
(debug: 001.txt)
```

### Bad Example

```
- @e1 Logo
- @e2 Login button
- @e3 Menu
- @e4 Search
- @e5 Footer
```

→ No context. Main agent cannot make informed decisions.

## Invocation Template

### Opening a Page

```
Task(
  model: haiku,
  subagent_type: general-purpose,
  prompt: """
  Use agent-browser to open {URL}.

  Steps:
  1. agent-browser open {URL}
  2. agent-browser snapshot -i -c > scratchpad/browser-session/001.txt
  3. Append to index.log: 001 | {time} | {URL} | {summary}

  Return ONLY a structured summary:
  - Page type and overview (e.g., "Login page", "Product list with 50 items")
  - Key interactive elements with @refs
  - Do NOT include raw snapshot data

  Format:
  {Page description}

  Structure:
  - {feature}: {description} (@ref)

  Key elements:
  - {element description} (@ref)

  Ready for instructions.
  (debug: 001.txt)
  """
)
```

### Performing Actions

```
Task(
  model: haiku,
  subagent_type: general-purpose,
  prompt: """
  Use agent-browser:
  1. agent-browser fill @e1 "value"
  2. agent-browser click @e3
  3. agent-browser snapshot -i -c > scratchpad/browser-session/002.txt
  4. Update index.log

  Return structured summary only.
  """
)
```

### Complex Pages

If snapshot exceeds limits, use additional constraints:

```
agent-browser snapshot -i -c -d 2 > file.txt   # Depth limit
agent-browser snapshot -i -c -s "#content"     # Scope to selector
```

## Rules

### Main Agent

- Trust subagent summaries
- Do NOT read raw files (scratchpad/browser-session/*.txt) unless debugging
- Raw data is for troubleshooting only

### Subagent

- Always save snapshots to files (never return raw data)
- Append to index.log for history
- Use `--session` flag to maintain browser state
- Return concise, structured summaries

## File Structure

```
scratchpad/browser-session/
├── index.log    # History (debug reference only)
├── 001.txt      # Raw snapshot data
├── 002.txt
└── ...
```

### index.log Format

```
# browser-session index (debug only - do not read unless troubleshooting)

001 | 14:23:45 | https://example.com/login     | Login form
002 | 14:24:12 | https://example.com/dashboard | Dashboard with 5 widgets
```

## Ending Session

```
Task(
  model: haiku,
  subagent_type: general-purpose,
  prompt: "Run: agent-browser close"
)
```

---

## Web Clip (HTML → Markdown)

Convert web pages to clean Markdown using `clipper`. No subagent needed — runs directly.

### Basic Usage

```bash
clipper clip -u "https://example.com/article" -o scratchpad/browser-session/clip.md
```

This is the **preferred method**. Quality is equivalent to Obsidian Web Clipper (same Readability + Turndown stack).

### Output Formats

| Flag | Format |
|------|--------|
| `-f md` | Markdown (default) |
| `-f json` | JSON (title, content, url) |

### Batch Clipping

```bash
clipper crawl -u "https://docs.example.com" -g "https://docs.example.com/**/*" -o dataset.jsonl
```

### When to Use What

| Scenario | Tool |
|----------|------|
| Read an article → Markdown | `clipper clip -u` |
| Interact with a page (click, fill, navigate) | agent-browser + Haiku subagent |
