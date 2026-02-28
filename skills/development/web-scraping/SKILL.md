---
name: web-scraping
description: Web scraping best practices for AI coding agents. Covers tmux session management for long-running scrapes, Crawl4AI integration, parallel pipeline orchestration, resume-friendly architecture, and rate limit handling. Use this skill when building scrapers, running data extraction jobs, or managing lead generation pipelines.
license: MIT
metadata:
  author: Top of Funnel
  version: "1.0.0"
  repository: https://github.com/bcharleson/webscraping-skill
---

# Web Scraping Skill for AI Agents

This skill provides best practices and operational patterns for building and running web scrapers. It covers session management, parallel execution, error recovery, and integration with AI-powered extraction tools.

## When to Use This Skill

- **Building a web scraper** — structuring code for reliability and resumability
- **Running long scrapes** — protecting processes with tmux
- **Lead generation pipelines** — orchestrating multi-stage data extraction
- **Data enrichment** — email finding, validation, and waterfall patterns
- **AI-powered extraction** — using Crawl4AI or LLM-based parsing
- **Remote server scraping** — SSH + tmux for location-independent execution

## Core Principles

### 1. Always Protect Long-Running Processes

Any script that runs longer than 5 minutes MUST run inside a tmux session.

```bash
# Standard pattern — always follow this
tmux new -s descriptive-name
python my_scraper.py --resume
# Ctrl+B, D to detach
```

For AI agents executing scripts autonomously, use detached mode:

```bash
# Launch detached (agent can continue other work immediately)
tmux new -d -s job-name 'python my_scraper.py --resume'

# Check status without attaching
tmux capture-pane -t job-name -p | tail -10

# Check if session is still alive
tmux has-session -t job-name 2>/dev/null && echo "running" || echo "done"
```

### 2. Always Build Resume Support

Every scraper should survive interruption. Implement checkpointing:

```python
import json
import os

PROGRESS_FILE = "output/progress.json"

def load_progress():
    if os.path.exists(PROGRESS_FILE):
        with open(PROGRESS_FILE) as f:
            return json.load(f)
    return {"completed": [], "last_index": 0}

def save_progress(progress):
    # Atomic write — write to temp file, then rename
    tmp = PROGRESS_FILE + ".tmp"
    with open(tmp, "w") as f:
        json.dump(progress, f)
    os.rename(tmp, PROGRESS_FILE)
```

### 3. Always Log to Files

Never rely solely on terminal output:

```bash
# Combine terminal output AND file logging
python scraper.py --resume 2>&1 | tee output/scrape.log
```

In Python, use dual logging:

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
    handlers=[
        logging.FileHandler("output/scrape.log"),
        logging.StreamHandler(),  # Also print to terminal
    ],
)
```

### 4. Respect Rate Limits

Always add delays between requests. When scraping directories or registries:

```python
import asyncio
import random

async def scrape_with_rate_limit(urls, delay_range=(1, 3)):
    for i, url in enumerate(urls):
        result = await fetch(url)
        process(result)

        # Random delay to avoid detection
        delay = random.uniform(*delay_range)
        await asyncio.sleep(delay)

        if i % 100 == 0:
            logging.info(f"Progress: {i}/{len(urls)}")
            save_progress({"last_index": i})
```

### 5. Name Sessions Descriptively

```bash
# Bad — you'll forget what's running
tmux new

# Good — instantly identifiable
tmux new -s email-enrichment-batch-3
tmux new -s directory-scrape-q1-2026
tmux new -s validation-run-feb06
```

## Scraping Patterns

### Pattern 1: Simple Sequential Scrape

For straightforward data collection from a single source:

```bash
tmux new -s my-scrape
python scraper.py --source listings --resume --output data/results.csv
# Ctrl+B, D
```

### Pattern 2: Parallel Pipeline

For multi-stage pipelines where stages can overlap:

```bash
# Stage 1: Discovery (2 hours)
tmux new -d -s discovery 'python scrape_directory.py --resume'

# Stage 2: Enrichment (runs on completed records from stage 1)
tmux new -d -s enrich 'python enrich_contacts.py --watch-input data/raw.csv'

# Stage 3: Validation
tmux new -d -s validate 'python validate_emails.py --watch-input data/enriched.csv'

# Monitor all stages
tmux ls
```

### Pattern 3: Multi-Source Aggregation

Scrape multiple sources in parallel, deduplicate later:

```bash
tmux new -d -s source-directory-a 'python scrape_source_a.py --resume'
tmux new -d -s source-directory-b 'python scrape_source_b.py --resume'
tmux new -d -s source-registry   'python scrape_registry.py --resume'

# When all finish, merge and deduplicate
python merge_sources.py --dedup --output data/combined.csv
```

### Pattern 4: Crawl4AI Integration

For AI-powered extraction from JavaScript-rendered pages:

```python
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, CacheMode

async def crawl_and_extract(urls, output_dir="output/pages"):
    config = CrawlerRunConfig(
        cache_mode=CacheMode.BYPASS,
        page_timeout=30000,
    )
    async with AsyncWebCrawler() as crawler:
        for i, url in enumerate(urls):
            result = await crawler.arun(url=url, config=config)
            if result.success:
                with open(f"{output_dir}/{i}.md", "w") as f:
                    f.write(result.markdown)
            print(f"[{i+1}/{len(urls)}] {url} - {'OK' if result.success else 'FAIL'}")
            await asyncio.sleep(2)

asyncio.run(crawl_and_extract(urls))
```

Always run Crawl4AI jobs in tmux:

```bash
tmux new -s crawl-sites
python crawl_extract.py --input data/urls.txt --resume
# Ctrl+B, D
```

### Pattern 5: Email Enrichment Waterfall

Try multiple methods in sequence, each operating on leftovers from the previous:

```bash
# Phase 1: Catch-all detection
tmux new -s enrich-catchall
python email_waterfall.py --phase catchall
# Ctrl+B, D

# Phase 2: API-based finders (after catchall completes)
tmux new -s enrich-finders
python email_waterfall.py --phase finders
# Ctrl+B, D

# Phase 3: Pattern permutation + validation
tmux new -s enrich-permutations
python email_waterfall.py --phase permutations
# Ctrl+B, D
```

### Pattern 6: Friday Night Deploy

Launch a full pipeline before the weekend:

```bash
# Friday evening
tmux new -d -s weekend-pipeline 'python full_pipeline.py --source all --enrich --validate --export'

# Saturday (check from phone via SSH)
ssh work-machine
tmux capture-pane -t weekend-pipeline -p | tail -5
# "Processing record 8,432 / 12,257..."

# Monday morning
tmux attach -t weekend-pipeline
# "Pipeline complete. 12,257 records. 3,847 validated emails."
```

## tmux Quick Reference

### Session Management

```
tmux new -s NAME              Create named session
tmux new -d -s NAME 'CMD'     Create detached session running CMD
tmux attach -t NAME           Reattach to session
tmux ls                       List all sessions
tmux kill-session -t NAME     Kill a session
```

### Inside a tmux Session

```
Ctrl+B, D                     Detach (session keeps running)
Ctrl+B, [                     Scroll mode (q to exit)
Ctrl+B, "                     Split pane horizontally
Ctrl+B, %                     Split pane vertically
Ctrl+B, arrow keys            Switch between panes
Ctrl+B, c                     New window
Ctrl+B, n / p                 Next / previous window
```

### Remote Monitoring (Without Attaching)

```bash
# Read last N lines of output
tmux capture-pane -t NAME -p | tail -20

# Check if session is alive
tmux has-session -t NAME 2>/dev/null && echo "running"

# Send Ctrl+C for graceful shutdown
tmux send-keys -t NAME C-c

# Send arbitrary input
tmux send-keys -t NAME 'q' Enter
```

## Agent-Specific Guidelines

When an AI agent is running scraping tasks:

1. **Always use `tmux new -d`** — the `-d` flag starts detached so the agent can continue working
2. **Check back periodically** — use `tmux capture-pane -t NAME -p | tail -10` to read output without attaching
3. **Run independent stages in parallel** — each in its own tmux session
4. **Never run long processes in the foreground** — always wrap in tmux
5. **Verify completion** — check `tmux has-session` or look for output files before proceeding to the next stage

## Scraper Architecture Checklist

When building a new scraper, ensure it has:

- [ ] `--resume` flag that loads progress from a checkpoint file
- [ ] Atomic checkpoint writes (write temp file, then rename)
- [ ] File-based logging (not just stdout)
- [ ] Configurable rate limiting (delay between requests)
- [ ] Graceful Ctrl+C handling (save progress before exit)
- [ ] Output to structured format (CSV or JSON)
- [ ] Error logging with failed URLs for retry
- [ ] User-agent rotation (if needed)
- [ ] Proxy support (if needed)

## Common Gotchas

| Problem | Solution |
|---------|----------|
| "Already attached" when reattaching | `tmux attach -t NAME -d` (force detach other viewer) |
| Session gone after reboot | Sessions don't survive restarts — use `--resume` |
| SSH drops mid-scrape | This is why you use tmux — just SSH back and reattach |
| Copy text from tmux scroll | `Ctrl+B, [` → navigate → `Space` to start → `Enter` to copy → `Ctrl+B, ]` to paste |
| Need to stop scraper gracefully | `tmux send-keys -t NAME C-c` from outside |

## Further Reading

- Full guide: [tmux for Web Scraping](https://topoffunnel.com/resources/tmux-web-scraping)
- Crawl4AI docs: [github.com/unclecode/crawl4ai](https://github.com/unclecode/crawl4ai)
- tmux cheat sheet: `man tmux`
