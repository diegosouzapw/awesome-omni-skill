---
name: shot-scraper
description: Automated web scraping using shot-scraper (Playwright-based CLI) via GitHub Actions to extract structured data from websites and export to JSON/SQLite for Datasette. Use when users need to periodically scrape web data, set up automated data collection workflows, extract structured information from pages using JavaScript execution, or create data pipelines that run on schedules.
license: Apache-2.0
---

# shot-scraper

## Quick Start

Extract data by executing JavaScript on a webpage and save as JSON:

```bash
# Install
pip install shot-scraper sqlite-utils
shot-scraper install

# Scrape data
shot-scraper javascript https://example.com/news "({
  articles: Array.from(document.querySelectorAll('article')).map(a => ({
    title: a.querySelector('h2')?.innerText,
    link: a.querySelector('a')?.href,
    date: a.querySelector('.date')?.innerText
  })),
  timestamp: new Date().toISOString()
})" -o data.json

# Import to SQLite (Datasette format)
sqlite-utils insert data.db articles data.json --pk=link --alter
```

## Core Workflow

1. **Write JavaScript extractor** - Returns JSON object or array
2. **Run via GitHub Actions** - Scheduled or on-demand
3. **Import to SQLite** - Use `sqlite-utils` for Datasette compatibility
4. **Commit results** - Track data changes over time

## JavaScript Execution

### Basic extraction
```bash
shot-scraper javascript URL "document.title"
shot-scraper javascript URL -i script.js -o output.json
```

### Return JSON objects
```javascript
// Wrap object literals in parentheses
({
  title: document.title,
  items: Array.from(document.querySelectorAll('.item')).map(i => i.innerText)
})
```

### Handle dynamic content
```javascript
new Promise(done => {
  setTimeout(() => {
    done({ data: document.querySelector('.dynamic')?.innerText });
  }, 2000);
});
```

### Common options
- `--wait MILLISECONDS` - Wait before executing
- `--timeout MILLISECONDS` - Max execution time
- `--auth FILE` - Authentication context
- `--user-agent STRING` - Custom user agent
- `--log-console` - Show console.log output

## GitHub Actions Integration

Basic scheduled scraper workflow:

```yaml
name: Scrape Website
on:
  schedule:
    - cron: '0 6 * * *'  # Daily at 6 AM
  workflow_dispatch:

jobs:
  scrape:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      
      - name: Install tools
        run: |
          pip install shot-scraper sqlite-utils
          shot-scraper install
      
      - name: Scrape data
        run: |
          shot-scraper javascript https://example.com \
            -i scrape.js \
            -o data/output-$(date +%Y%m%d).json
      
      - name: Import to database
        run: |
          sqlite-utils insert data/scraper.db records \
            data/output-*.json \
            --pk=id --alter
      
      - name: Commit results
        run: |
          git config user.name "Bot"
          git config user.email "bot@example.com"
          git add data/
          git commit -m "Update $(date +%Y-%m-%d)" || exit 0
          git push
```

## Data Export Patterns

### Single record per run
```javascript
({ timestamp: new Date().toISOString(), total: document.querySelectorAll('.item').length })
```
```bash
sqlite-utils insert data.db snapshots output.json --alter
```

### Multiple records per run
```javascript
Array.from(document.querySelectorAll('.product')).map(p => ({
  id: p.dataset.id,
  name: p.querySelector('.name')?.innerText,
  price: parseFloat(p.querySelector('.price')?.innerText.replace(/[^0-9.]/g, '')),
  timestamp: new Date().toISOString()
}))
```
```bash
sqlite-utils insert data.db products output.json --pk=id --replace
```

### With nested data
```bash
# Flatten nested objects
sqlite-utils insert data.db items output.json --flatten --alter
```

## Common Patterns

**Extract tables:**
```javascript
Array.from(document.querySelectorAll('table tr')).map(row => {
  const cells = row.querySelectorAll('td');
  return { col1: cells[0]?.innerText, col2: cells[1]?.innerText };
})
```

**Parse prices/numbers:**
```javascript
parseFloat(text.replace(/[^0-9.]/g, ''))
```

**Wait for element:**
```javascript
new Promise(done => {
  const check = setInterval(() => {
    const el = document.querySelector('.target');
    if (el) { clearInterval(check); done({ data: el.innerText }); }
  }, 100);
  setTimeout(() => { clearInterval(check); done({ error: 'timeout' }); }, 10000);
});
```

## sqlite-utils Commands

```bash
# Insert with primary key
sqlite-utils insert db.db table data.json --pk=id

# Replace existing records
sqlite-utils insert db.db table data.json --pk=id --replace

# Auto-alter schema to fit new data
sqlite-utils insert db.db table data.json --alter

# Create indexes for Datasette
sqlite-utils create-index db.db table column_name

# Query data
sqlite-utils query db.db "SELECT * FROM table" --csv
```

## Advanced Topics

**Authentication:** Store auth.json as GitHub secret, write to file in workflow:
```yaml
env:
  AUTH_JSON: ${{ secrets.AUTH_JSON }}
run: |
  echo "$AUTH_JSON" > auth.json
  shot-scraper javascript URL -a auth.json -i script.js -o data.json
```

**Error handling:** Add retry logic to workflow (see [workflows.md](references/workflows.md))

**Multiple pages:** Process URL list (see [examples.md](references/examples.md))

**Screenshots:** Take alongside data for verification:
```bash
shot-scraper URL -o screenshot.png
shot-scraper javascript URL -i script.js -o data.json
```

## References

- **[examples.md](references/examples.md)** - Complete workflow examples (price tracking, monitoring, error handling)
- **[scraping-patterns.md](references/scraping-patterns.md)** - JavaScript patterns for common scraping scenarios
- **[workflows.md](references/workflows.md)** - Advanced GitHub Actions patterns

## External Resources

- shot-scraper: https://github.com/simonw/shot-scraper
- sqlite-utils: https://sqlite-utils.datasette.io/
- Datasette: https://datasette.io/