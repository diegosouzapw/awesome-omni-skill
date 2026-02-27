---
name: ninja-enrich
description: "Enrich meta.yaml long_description fields from man pages and websites"
argument-hint: "[--force] [--path mainmenu/...] [--dry-run]"
disable-model-invocation: true
allowed-tools: Bash, Read
---

# Ninja Enrich

Populate `long_description` fields in `meta.yaml` files by fetching content
from man pages, website meta descriptions, or `--help` output.

**Usage:** `/ninja-enrich` or `/ninja-enrich --force`

## Execution

Run the enrich script:

```bash
.claude/scripts/ninja-enrich.sh $ARGUMENTS
```

## Sources (tried in order per tool)

1. **Man page** — `man -P cat <binary>` → parse `NAME` section
2. **Website** — fetch `website:` URL → extract `<meta description>` or `og:description`
3. **--help** — `<binary> --help` → first meaningful output line
4. **Skip** — no source available

## Flags

| Flag | Effect |
|------|--------|
| `--force` | Re-fetch even if `long_description` already set |
| `--path mainmenu/network` | Limit enrichment to a subtree |
| `--dry-run` | Show what would change without writing files |

## What it updates

- `long_description:` field in each `*.meta.yaml` / `meta.yaml` (source of truth)
- SQLite cache (`.cache/menu.db`) rebuilt automatically after YAML updates

## When to use

- After adding new scripts that have man pages
- After adding `website:` fields to web-app meta files
- Periodically to refresh descriptions as upstream docs change (use `--force`)
- With `--dry-run` to preview changes before committing
