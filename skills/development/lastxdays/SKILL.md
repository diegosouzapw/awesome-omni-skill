---
name: lastXdays
description: Research a topic from the last X days on Reddit + X + Web. Specify number of days as first argument.
argument-hint: "<days> [topic]" or "[topic]" (defaults to 30 days)
context: fork
agent: Explore
disable-model-invocation: true
allowed-tools: Bash, Read, Write, AskUserQuestion, WebSearch
---

# lastXdays: Research Any Topic from a Custom Time Range

Research ANY topic across Reddit, X, and the web with a **configurable time window**. Specify how many days back to search.

## Usage

```
lastXdays 7 "AI tools"           # Last 7 days
lastXdays 14 "Claude Code"       # Last 2 weeks
lastXdays 3 "breaking news"      # Last 3 days
lastXdays "best prompts"         # Defaults to 30 days
```

## Time Range

- **Minimum:** 1 day
- **Maximum:** 365 days
- **Default:** 30 days (if no number specified)

## How It Works

1. Parse the first argument as number of days (or use default 30)
2. Research Reddit, X, and web for content from that time window
3. Synthesize findings with engagement metrics
4. Deliver expert-level prompts based on research

## Research Execution

Run the research script:
```bash
python3 ~/.claude/skills/lastXdays/scripts/lastXdays.py <days> "$TOPIC" --emit=compact 2>&1
```

Examples:
```bash
python3 ~/.claude/skills/lastXdays/scripts/lastXdays.py 7 "AI tools" --emit=compact
python3 ~/.claude/skills/lastXdays/scripts/lastXdays.py 14 "best practices" --quick
python3 ~/.claude/skills/lastXdays/scripts/lastXdays.py 3 "breaking news" --deep
```

## Options

- `--quick` — Faster, fewer sources (8-12 each)
- `--deep` — Comprehensive (50-70 Reddit, 40-60 X)
- `--emit=MODE` — Output: compact|json|md|context|path
- `--sources=MODE` — Source: auto|reddit|x|both
- `--debug` — Verbose logging

## Output

Same format as last30days — engagement metrics, synthesis, and copy-paste prompts tailored to the user's target tool.

---

*Based on last30days skill with configurable time range parameter.*
