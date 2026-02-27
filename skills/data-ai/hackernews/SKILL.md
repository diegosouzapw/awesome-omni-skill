---
name: hackernews
description: Comprehensive toolkit for fetching, searching, analyzing, and monitoring Hacker News content. Use when Claude needs to interact with Hacker News for (1) Fetching top/new/best/ask/show/job stories, (2) Searching for specific topics or keywords, (3) Monitoring specific users or tracking their activity, (4) Analyzing trending topics and patterns, (5) Getting story details, comments, or user profiles, or (6) Any other task involving Hacker News data retrieval or analysis.
---

# Hacker News Skill

## Overview

This skill provides comprehensive access to Hacker News through both the official Firebase API and Algolia Search API. Use the provided scripts to fetch stories, search content, monitor users, and analyze trending topics.

## Core Capabilities

### 1. Fetch Stories

Use `scripts/fetch_stories.py` to retrieve stories from different categories.

**Available story types:**
- `top` - Top ranked stories (front page)
- `new` - Newest stories
- `best` - Best stories (highest quality)
- `ask` - Ask HN posts
- `show` - Show HN posts
- `job` - Job postings

**Usage:**
```bash
cd /home/claude/hackernews/scripts
python3 fetch_stories.py --type top --limit 10
python3 fetch_stories.py --type ask --limit 20 --detailed
```

**Common patterns:**
- For current front page: Use `--type top`
- For latest posts: Use `--type new`
- For highest quality: Use `--type best`
- For detailed info: Add `--detailed` flag

### 2. Search Content

Use `scripts/search_hn.py` to search across all HN content using Algolia's search API.

**Search types:**
- `all` - Search everything
- `story` - Stories only
- `comment` - Comments only
- `ask` - Ask HN posts only
- `show` - Show HN posts only

**Sort options:**
- `relevance` - Most relevant results (default)
- `date` - Most recent results

**Usage:**
```bash
cd /home/claude/hackernews/scripts
python3 search_hn.py "AI security" --type story --limit 20
python3 search_hn.py "Rust" --sort date --limit 15
python3 search_hn.py "" --author pg --limit 10
```

**Common patterns:**
- For topic search: `search_hn.py "topic keywords" --type story`
- For recent discussions: Add `--sort date`
- For user activity: Use `--author username`

### 3. Monitor Users

Use `scripts/user_monitor.py` to track specific users and view their activity.

**Usage:**
```bash
cd /home/claude/hackernews/scripts
python3 user_monitor.py pjmlp --limit 10
python3 user_monitor.py pg sama dang --limit 5
python3 user_monitor.py username --type story --activity-only
```

**Features:**
- View user profile (karma, account age, about)
- See recent submissions (stories, comments, etc.)
- Filter by item type
- Monitor multiple users at once

**Options:**
- `--limit` - Number of recent items per user
- `--type` - Filter by story/comment/poll/job
- `--activity-only` - Skip profile, show only activity
- `--profile-only` - Show only profile, skip activity

### 4. Analyze Trends

Use `scripts/analyze_trends.py` to identify trending topics and patterns.

**Usage:**
```bash
cd /home/claude/hackernews/scripts
python3 analyze_trends.py --limit 100 --type top
python3 analyze_trends.py --compare
```

**Provides:**
- Top trending keywords from story titles
- Most submitted domains
- Most active authors
- Overall statistics (avg score, comments)
- Top stories by score and engagement
- Comparison across story types (when using `--compare`)

**Typical workflow:**
1. Run with `--limit 50-200` for good sample size
2. Use `--type top` for current trends
3. Use `--type new` for emerging topics
4. Use `--compare` for comprehensive analysis

### 5. Programmatic Access

For custom workflows, import the API modules directly:

```python
import sys
sys.path.append('/home/claude/hackernews/scripts')

from hn_api import HNClient, format_item
from search_hn import HNSearchClient

# Use Firebase API
client = HNClient()
top_ids = client.get_top_stories(limit=5)
for sid in top_ids:
    item = client.get_item(sid)
    print(format_item(item))

# Use Search API
search = HNSearchClient()
results = search.search_stories("machine learning", num_results=10)
for hit in results['hits']:
    print(hit['title'])
```

## Quick Decision Guide

**User asks for current/top stories?**
→ Use `fetch_stories.py --type top`

**User asks to search for a topic?**
→ Use `search_hn.py "topic" --type story`

**User asks about a specific user?**
→ Use `user_monitor.py username`

**User asks about trending topics?**
→ Use `analyze_trends.py --limit 100`

**User asks for Ask/Show HN?**
→ Use `fetch_stories.py --type ask` or `--type show`

**User wants detailed analysis?**
→ Use `analyze_trends.py --compare`

## Important Notes

1. **Always use full paths** - Scripts are in `/home/claude/hackernews/scripts`
2. **Check for requests dependency** - Install with `pip install requests` if needed
3. **Handle missing data** - HN items can be deleted; check for None/null
4. **Respect rate limits** - Algolia allows 10,000 requests/hour per IP
5. **Cache when possible** - HN data updates slowly; cache responses

## Additional Resources

- **API Reference**: See `references/api_reference.md` for complete API documentation
- **Usage Examples**: See `references/examples.md` for detailed workflow examples

## Scripts Reference

All scripts located in `/home/claude/hackernews/scripts/`:

- **hn_api.py** - Core Firebase API client (can be imported)
- **fetch_stories.py** - Fetch stories by type (top/new/best/ask/show/job)
- **search_hn.py** - Search HN using Algolia API
- **user_monitor.py** - Monitor user profiles and activity
- **analyze_trends.py** - Analyze trending topics and patterns
