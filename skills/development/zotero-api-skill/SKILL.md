---
name: zotero-api-skill
description: Zotero HTTP API helper for downloading, fetching, searching, creating, and updating Zotero items. Use when syncing or managing Zotero items programmatically; defaults to ZOTERO_USER and ZOTERO_API_KEY environment variables.
---

# Zotero API Skill

## Overview
Use `scripts/zotero_cli.py` to perform deterministic Zotero API operations: download items to disk, fetch a single item, search items, create new items, or write updated metadata back to Zotero. The script reads `ZOTERO_USER` (user ID) and `ZOTERO_API_KEY` from the environment unless overridden by flags.

## Quick Start

Download all items (skip attachments by default):
```bash
zotero-api-skill/scripts/zotero_cli.py download --output-dir ./output
```

Fetch a single item as JSON:
```bash
zotero-api-skill/scripts/zotero_cli.py get --key ABCD1234
```

Update an item from a JSON file:
```bash
zotero-api-skill/scripts/zotero_cli.py update --key ABCD1234 --input ./output/ABCD1234/original.json
```

Search items by query:
```bash
zotero-api-skill/scripts/zotero_cli.py search --query "transformer retrieval" --limit 10
```

Create an item from data JSON:
```bash
zotero-api-skill/scripts/zotero_cli.py create --input ./new_item.json --data-only
```

## Tasks

### Download items
Use `download` to save each item into its own directory under `--output-dir`. The script writes `original.json` and a dated `original-MM-DD.json` snapshot.

Common options:
- `--limit`: page size for Zotero API pagination.
- `--max-items`: stop after N items.
- `--include-attachments`: include attachment items.
- `--start`: start offset for pagination.
- `--no-snapshot`: skip the dated snapshot file.

### Fetch a single item
Use `get` with `--key` to print the item JSON to stdout, or pass `--output` to save it to a file.

### Search items
Use `search` with `--query` to run a Zotero quicksearch. Optional filters: `--qmode`, `--item-type`, `--tag`, `--start`, `--limit`.

### Create items
Use `create` with `--input` to POST new item data. The input file can be:
- A list of data objects (array), or
- A single data object, or
- A list of full Zotero items (with a top-level `data` field).

### Update item metadata
Use `update` with `--key` and `--input` to PUT item metadata. The input file can be:
- A full Zotero item JSON (with a top-level `data` field), or
- A data-only object (set `--data-only` to force this).

Prefer updating from a freshly fetched item so required fields like `itemType`, `key`, and `version` stay consistent.

## Notes
- The script uses Zotero API version 3 and retries on transient HTTP errors.
- If you need the same update flow as the repo, edit the `data` payload and use `update` to write back.
