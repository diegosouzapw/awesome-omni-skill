---
name: anydocs
description: Generic Documentation Indexing & Search. Index any documentation site (SPA/static) and search it instantly.
tools:
  - name: anydocs_search
    description: Search indexed documentation profiles. Returns ranked results with snippets.
    parameters:
      type: object
      properties:
        query:
          type: string
          description: Search query (keyword or phrase)
        profile:
          type: string
          description: Profile name (e.g. 'discord', 'openclaw')
        limit:
          type: number
          description: Max results to return (default 5)
      required: [query]
  - name: anydocs_index
    description: Build or update the search index for a documentation profile.
    parameters:
      type: object
      properties:
        profile:
          type: string
          description: Profile name to index
        use_browser:
          type: boolean
          description: Use browser rendering for SPAs (requires gateway token)
      required: [profile]
  - name: anydocs_config
    description: Configure a new documentation profile.
    parameters:
      type: object
      properties:
        profile:
          type: string
          description: Profile name
        base_url:
          type: string
          description: Base URL of the docs
        sitemap_url:
          type: string
          description: URL to sitemap.xml
      required: [profile, base_url, sitemap_url]
---

# anydocs - Generic Documentation Indexing & Search

A powerful, reusable skill for indexing and searching **ANY** documentation site.

## What It Does

`anydocs` solves a real problem: accessing documentation from code or CLI. Instead of opening a browser every time, you can:

- **Index** any documentation site (Discord, OpenClaw, internal docs, etc.)
- **Search** instantly from the command line or Python API
- **Cache** pages locally to avoid repeated network calls
- **Configure** multiple profiles for different doc sites

## When to Use It

Use `anydocs` when you need to:
- Quickly look up API documentation without leaving the terminal
- Build agents that need to reference docs
- Extract specific information from documentation
- Search across multiple documentation sites
- Integrate docs into your workflow

## Key Features

### 🔍 Multi-Method Search
- **Keyword search**: Fast, term-based matching with BM25-style scoring
- **Hybrid search**: Keyword + phrase proximity for better relevance
- **Regex search**: Advanced pattern matching for power users

### 🌐 Works with Any Docs Site
- Sitemap-based discovery (standard XML sitemap)
- Fallback crawling from base URL
- HTML content extraction with smart selector detection
- Automatic rate limiting to be respectful

### 💾 Smart Caching
- Pages cached locally with 7-day TTL (configurable)
- Search indexes cached for instant second searches
- Cache statistics and cleanup commands
- Respects cache invalidation

### ⚙️ Profile-Based Configuration
- Support multiple doc sites simultaneously
- Per-profile search methods and cache TTLs
- Configuration stored in `~/.anydocs/config.json`
- Examples for Discord, OpenClaw, and custom sites

### 🌐 JavaScript Rendering (Optional)
- Uses Playwright to render client-side SPAs (Single Page Apps)
- Automatically discovers links on JS-heavy sites like Discord docs
- Gracefully falls back to standard HTTP if Playwright unavailable
- Configure per-discovery session or globally per profile

## Installation

```bash
cd /path/to/skills/anydocs
pip install -r requirements.txt
chmod +x anydocs.py
```

### Optional: Browser-based rendering (for JavaScript-heavy sites)

For sites like Discord that use client-side rendering, install Playwright:

```bash
pip install playwright==1.40.0
playwright install  # Downloads Chromium
```

If Playwright is unavailable, anydocs gracefully falls back to standard HTTP fetching.

## Quick Start

### 1. Configure a Documentation Site

```bash
python anydocs.py config vuejs \
  https://vuejs.org \
  https://vuejs.org/sitemap.xml
```

### 2. Build the Index

```bash
python anydocs.py index vuejs
```

This discovers all pages via sitemap, scrapes content, and builds a searchable index.

### 3. Search

```bash
python anydocs.py search "composition api" --profile vuejs
python anydocs.py search "reactivity" --profile vuejs --limit 5
```

### 4. Fetch a Specific Page

```bash
python anydocs.py fetch "guide/introduction" --profile vuejs
```

## CLI Commands

### Configuration

```bash
# Add or update a profile
anydocs config <profile> <base_url> <sitemap_url> [--search-method hybrid] [--ttl-days 7]

# List configured profiles
anydocs list-profiles
```

### Indexing

```bash
# Build index for a profile
anydocs index <profile>

# Force re-index (skip cache)
anydocs index <profile> --force
```

### Search

```bash
# Basic keyword search
anydocs search "query" --profile discord

# Limit results
anydocs search "query" --profile discord --limit 5

# Regex search
anydocs search "^API" --profile discord --regex
```

### Fetch

```bash
# Fetch a specific page (URL or path)
anydocs fetch "https://discord.com/developers/docs/resources/webhook"
anydocs fetch "resources/webhook" --profile discord
```

### Cache Management

```bash
# Show cache statistics
anydocs cache status

# Clear all cache
anydocs cache clear

# Clear specific profile's cache
anydocs cache clear --profile discord
```

## Python API

For use in agents and scripts:

```python
from lib.config import ConfigManager
from lib.scraper import DiscoveryEngine
from lib.indexer import SearchIndex

# Load configuration
config_mgr = ConfigManager()
config = config_mgr.get_profile("discord")

# Scrape documentation
scraper = DiscoveryEngine(config["base_url"], config["sitemap_url"])
pages = scraper.fetch_all()

# Build search index
index = SearchIndex()
index.build(pages)

# Search
results = index.search("webhooks", limit=10)
for result in results:
    print(f"{result['title']} ({result['relevance_score']})")
    print(f"  {result['url']}")
```

## Configuration File Format

Configuration is stored in `~/.anydocs/config.json`:

```json
{
  "discord": {
    "name": "discord",
    "base_url": "https://discord.com/developers/docs",
    "sitemap_url": "https://discord.com/developers/docs/sitemap.xml",
    "search_method": "hybrid",
    "cache_ttl_days": 7
  },
  "openclaw": {
    "name": "openclaw",
    "base_url": "https://docs.openclaw.ai",
    "sitemap_url": "https://docs.openclaw.ai/sitemap.xml",
    "search_method": "hybrid",
    "cache_ttl_days": 7
  }
}
```

## Search Methods

### Keyword Search
- **Speed**: Fast
- **Best for**: Common terms, exact matches
- **How it works**: Term matching with position weighting (title > tags > content)
- **Example**: `anydocs search "webhooks"`

### Hybrid Search (Default)
- **Speed**: Fast
- **Best for**: Natural language queries
- **How it works**: Keyword search + phrase proximity scoring
- **Example**: `anydocs search "how to set up webhooks"`

### Regex Search
- **Speed**: Medium
- **Best for**: Complex patterns
- **How it works**: Compiled regex pattern matching across all content
- **Example**: `anydocs search "^(GET|POST)" --regex`

## Caching Behavior

- **Pages**: Cached as JSON with 7-day TTL (configurable)
- **Indexes**: Cached after indexing, invalidated on TTL expiry
- **Cache location**: `~/.anydocs/cache/`
- **Manual refresh**: Use `--force` flag or clear cache

## Performance Notes

- First index build takes 2-10 minutes depending on site size
- Subsequent searches are instant (cached indexes)
- Rate limit: 0.5s per page to be respectful
- Typical search returns ~100 results in <100ms

## Troubleshooting

### "No index for 'profile'" error
Run `anydocs index <profile>` first to build the index.

### Sitemap not found
Check the sitemap URL. Falls back to crawling from base_url if unavailable.

### Slow indexing
This is normal for large sites. Rate limiting prevents overwhelming servers.

### Cache grows too large
Run `anydocs cache clear` or set `--ttl-days` to a smaller value.

## Examples

### Vue.js Framework Docs (SPA Example)
```bash
anydocs config vuejs \
  https://vuejs.org \
  https://vuejs.org/sitemap.xml
anydocs index vuejs
anydocs search "composition api"
```

### Next.js API Docs
```bash
anydocs config nextjs \
  https://nextjs.org \
  https://nextjs.org/sitemap.xml
anydocs index nextjs
anydocs search "app router" --profile nextjs
```

### Internal Company Documentation
```bash
anydocs config internal \
  https://docs.company.local \
  https://docs.company.local/sitemap.xml
anydocs index internal --force
anydocs search "deployment" --profile internal
```

## Architecture

- **scraper.py**: Discovers URLs via sitemap, fetches and parses HTML
- **indexer.py**: Builds searchable indexes, implements multiple search strategies
- **config.py**: Manages configuration profiles
- **cache.py**: TTL-based file caching for pages and indexes
- **cli.py**: Click-based command-line interface

## Contributing

To add new documentation sites, run:
```bash
anydocs config <profile> <base_url> <sitemap_url>
```

To extend search functionality, modify `lib/indexer.py`.

## License

Part of the OpenClaw system.
