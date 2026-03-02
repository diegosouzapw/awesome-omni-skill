---
name: arxivterminal
description: CLI tool (arxivterminal) for fetching, searching, and managing arXiv papers locally. Use when working with arXiv papers using the arxivterminal command - fetching new papers by category, searching the local database, viewing papers from specific dates, or managing the local paper database.
---

# arXivTerminal

CLI tool for managing arXiv papers with local database storage.

## Quick Reference

### Fetch Papers from arXiv
When you need to download papers from arXiv and store them locally:
- Use `arxiv fetch --num-days N --categories CATEGORIES`
- See [arxivterminal-fetch.md](references/arxivterminal-fetch.md) for detailed options and examples

### Search Local Database
When you need to search papers already in your local database:
- Use `arxiv search QUERY`
- See [arxivterminal-search.md](references/arxivterminal-search.md) for search options including experimental semantic search

### Show Papers by Date
When you need to view papers from a specific time period:
- Use `arxiv show --days-ago N`
- See [arxivterminal-show.md](references/arxivterminal-show.md) for details

### Database Statistics
When you need to check what's in your database:
- Use `arxiv stats`
- See [arxivterminal-stats.md](references/arxivterminal-stats.md) for output format

### Database Management
When you need to clean up or reset your database:
- Use `arxiv delete-all`
- See [arxivterminal-management.md](references/arxivterminal-management.md) for database location and backup procedures

## Data Storage

- **Database**: `~/Library/Application Support/arxivterminal/papers.db`
- **Logs**: `~/Library/Logs/arxivterminal/arxivterminal.log`

## Common Workflows

### Daily Research Workflow
```bash
arxiv fetch --num-days 1 --categories cs.AI,cs.CL
arxiv search -e -l 20 "large language models"
```

### Weekly Review
```bash
arxiv fetch --num-days 7 --categories cs.AI,cs.LG,cs.CV
arxiv stats
```
