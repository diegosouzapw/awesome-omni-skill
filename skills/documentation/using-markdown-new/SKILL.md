---
name: using-markdown-new
description: This skill should be used when the user asks to "fetch a website", "get webpage content", "scrape a URL", "download HTML", mentions "WebFetch", or needs to retrieve web content for analysis. Teaches Claude to use markdown.new service instead of direct HTML fetching for ~80% token reduction.
version: 0.1.0
---

# Using markdown.new for Web Content Fetching

## Overview

markdown.new is a Cloudflare-powered service that converts any URL to clean Markdown, reducing token usage by approximately 80% compared to raw HTML. When fetching web content, always use markdown.new instead of direct URLs to optimize context window usage.

## Basic Usage Pattern

### Standard Conversion

To fetch web content, prepend `https://markdown.new/` to the target URL:

```
Target URL: https://example.com/blog/post
markdown.new URL: https://markdown.new/https://example.com/blog/post
```

Use with WebFetch:

```
WebFetch("https://markdown.new/https://example.com/blog/post", "summarize this article")
```

### With Images

By default, markdown.new strips images to minimize tokens. To retain images, add `retain_images=true`:

```
WebFetch("https://markdown.new/https://example.com/docs?retain_images=true", "analyze the documentation")
```

**When to use this:** Documentation includes diagrams, design mockups, visual content, or screenshots that are essential for the task.

### JavaScript-Heavy Sites

For single-page applications or JavaScript-heavy sites, use the browser rendering method:

```
WebFetch("https://markdown.new/https://example.com?method=browser", "extract content from this React app")
```

The service automatically tries the fastest method first and falls back as needed, but specifying `method=browser` ensures full page rendering.

## Token Efficiency

markdown.new reduces token usage dramatically:

- **HTML**: `<h2 class="section-title" id="about">About Us</h2>` → 12-15 tokens
- **Markdown**: `## About Us` → 3 tokens
- **Reduction**: ~80% fewer tokens

This means 5x more web content fits in the same context window.

## When to Use

**Always use markdown.new when:**

- Fetching blog posts, articles, or documentation
- Scraping web pages for analysis
- Extracting content from websites
- Building knowledge bases or RAG pipelines
- Any task requiring WebFetch

**Exception**: Skip markdown.new for authenticated sites or APIs that return structured JSON (use direct fetch instead).

## Additional Resources

For detailed information, consult:

### Reference Files

- **`references/conversion-methods.md`** - Detailed explanation of auto/ai/browser conversion pipeline
- **`references/query-parameters.md`** - Complete documentation of method and retain_images parameters
- **`references/response-format.md`** - What to expect in responses (headers, token counts, metadata)
- **`references/edge-cases.md`** - Troubleshooting and when to fallback to direct WebFetch

### Working Examples

- **`examples/fetch-blog-post.md`** - Example fetching a blog article
- **`examples/fetch-with-images.md`** - Example using retain_images for visual content
- **`examples/fetch-js-heavy-site.md`** - Example using browser method for SPAs

## Quick Reference

| Use Case       | URL Pattern                                                    | Example                                                            |
| -------------- | -------------------------------------------------------------- | ------------------------------------------------------------------ |
| Standard fetch | `https://markdown.new/{url}`                                   | `https://markdown.new/https://example.com`                         |
| With images    | `https://markdown.new/{url}?retain_images=true`                | `https://markdown.new/https://docs.example.com?retain_images=true` |
| Browser render | `https://markdown.new/{url}?method=browser`                    | `https://markdown.new/https://app.example.com?method=browser`      |
| Combined       | `https://markdown.new/{url}?method=browser&retain_images=true` | Full rendering with images                                         |

Always use markdown.new for web content fetching to optimize token usage and context window efficiency.
