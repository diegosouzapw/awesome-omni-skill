---
name: anti-scraping
description: Use when need to bypass Cloudflare protection, scrape websites with anti-bot measures, render JavaScript pages, or simulate real browser behavior for web scraping
---

# Anti-Scraping & Web Scraping

**When to use**: Websites with Cloudflare protection, JavaScript rendering requirements, or anti-bot measures.

## Overview

Provides battle-tested solutions for bypassing common anti-scraping measures using Playwright headless browser with stealth configurations.

## Key Capabilities

- ✅ Cloudflare challenge bypass
- ✅ JavaScript rendering
- ✅ Real browser context simulation
- ✅ Stealth mode (hides automation detection)
- ✅ Screenshot capture for debugging

## Quick Start

### Prerequisites
```bash
# Install Playwright
npm install -g playwright
playwright install chromium
```

### Basic Usage Pattern

```javascript
// n8n Execute Command node
const { execSync } = require('child_process');

const url = 'https://example.com';
const outputFile = '/tmp/page.html';

// Playwright command with stealth
const command = `node playwright-cloudflare.js "${url}" "${outputFile}"`;
execSync(command);

// Read result
const html = fs.readFileSync(outputFile, 'utf8');
```

## Core Script: playwright-cloudflare.js

**Location**: `n8n-skills/anti-scraping/playwright-cloudflare.js`

**Key Features**:
- Disables automation detection
- Sets real browser headers
- Configures viewport and user agent
- Handles Cloudflare waiting
- Captures screenshots on failure

**Configuration**:
```javascript
const config = {
  waitForCloudflare: true,      // Wait for CF challenge
  waitTime: 15000,               // Max wait time (ms)
  selector: '.product-list',     // Element to wait for
  screenshotOnError: true,       // Debug screenshots
  userAgent: 'Mozilla/5.0...'   // Real browser UA
};
```

## n8n Workflow Pattern

```
[Manual Trigger]
    ↓
[Set Parameters]
    target_url: https://site.com
    wait_selector: .content
    ↓
[Execute Command: Playwright]
    Command: node
    Arguments: playwright-cloudflare.js {{$json.target_url}} /tmp/output.html
    ↓
[Read HTML File]
    File: /tmp/output.html
    ↓
[Parse with Cheerio]
    (use html-parsing skill)
```

## Performance

- **Speed**: 15-25 seconds per page
- **Success Rate**: ~95% for Cloudflare sites
- **Resource Usage**: ~200-300MB RAM per browser instance

## Troubleshooting

### Cloudflare Still Blocking
```bash
# Increase wait time
--wait 30000

# Add specific selector to wait for
--selector '.product-list'

# Check screenshot for errors
/tmp/error-screenshot.png
```

### Timeout Errors
```bash
# Increase timeout in playwright script
timeout: 60000  // 60 seconds
```

### Memory Issues
```bash
# Close browser properly
await browser.close();

# Limit concurrent instances
# Use n8n Split Into Batches with batch size = 1
```

## Best Practices

1. **Add Delays**: Wait 3-5 seconds between requests
2. **Rotate User Agents**: Change UA periodically
3. **Use Residential Proxies**: For high-volume scraping
4. **Handle Errors**: Implement retry logic with exponential backoff
5. **Respect robots.txt**: Check site policies

## Common Patterns

### Pattern 1: Single Page Scraping
```
Trigger → Playwright → Parse → Export
```

### Pattern 2: Multi-Page with Pagination
```
Trigger → Generate URLs (pagination skill) →
Split Into Batches → Playwright → Wait 5s →
Parse → Deduplicate → Export
```

### Pattern 3: With Error Handling
```
Playwright → [Error Trigger] → Retry Logic → Notification
```

## Integration with Other Skills

- **pagination**: Generate URLs for multi-page scraping
- **html-parsing**: Extract data from rendered HTML
- **error-handling**: Retry on failures
- **debugging**: Validate extracted data

## Full Code and Documentation

Complete implementation with examples:
`/mnt/d/work/n8n_agent/n8n-skills/anti-scraping/`

Files:
- `playwright-cloudflare.js` - Main scraping script
- `README.md` - Detailed documentation
- `example-workflow.json` - n8n workflow example
- `config.template.env` - Configuration template
