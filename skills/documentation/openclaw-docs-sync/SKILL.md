---
name: openclaw-docs-sync
description: Sync OpenClaw + ClawHub + Skills docs into a local mirror for the QMD memory backend.
metadata:
  author: Seth Rose
  version: 3.0.0
  keywords:
    - openclaw
    - clawhub
    - skills
    - docs
    - memory
    - qmd
    - search
---

# OpenClaw Docs Sync

## Purpose
Keep a local mirror of the OpenClaw framework docs, ClawHub CLI/docs, and the Skills repository. OpenClaw's built-in QMD memory backend indexes these docs for hybrid semantic search.

## When to use
Use this skill when the agent needs **fresh documentation** or **precise references** from:
- OpenClaw framework docs
- ClawHub CLI/docs  
- Skills repo (all `SKILL.md` files)

## Setup

### Step 1: Run the sync script
Run the script whenever you need a fresh mirror:

```bash
# Recommended (portable)
npx tsx scripts/sync-docs.ts

# Or with Bun (fastest)
bun run scripts/sync-docs.ts
```

What it does:
- Mirrors OpenClaw `docs/` into `~/.openclaw/docs/openclaw-docs/`
- Mirrors ClawHub `docs/` into `~/.openclaw/docs/clawhub-docs/`
- Mirrors Skills `skills/` into `~/.openclaw/docs/openclaw-skills/`
- Runs security scans on downloaded content

### Step 2: Add to openclaw.json
After syncing, add the path to your `openclaw.json` config using `memory.qmd.paths`:

```json
{
  "memory": {
    "backend": "qmd",
    "citations": "auto",
    "qmd": {
      "includeDefaultMemory": true,
      "update": { "interval": "5m" },
      "paths": [
        {
          "name": "openclaw-docs",
          "path": "~/.openclaw/docs",
          "pattern": "**/*.{md,mdx}"
        }
      ]
    }
  }
}
```

## How the agent uses synced docs

Once configured in `openclaw.json`, OpenClaw's QMD memory backend automatically:
1. **Indexes** the synced docs on startup and at regular intervals (default 5m)
2. **Embeds** chunks for hybrid search (BM25 + vectors + reranking)
3. **Searches** via `memory_search` tool when you ask questions
4. **Retrieves** exact file content via `memory_get` tool

The agent will cite sources like:
```
Source: qmd/openclaw-docs/concepts/memory.md#L45-L52
```

## Repo structure in synced mirror

```
~/.openclaw/docs/
├── openclaw-docs/   # OpenClaw framework docs
├── clawhub-docs/    # ClawHub CLI docs  
└── openclaw-skills/ # All SKILL.md files from skills repo
```

## Notes

- The sync script only ingests `.md` and `.mdx` files
- Synced docs are read-only; edit upstream repos to update
- Security scans run automatically; threats logged to `~/.openclaw/security-scan.log`
- See [QMD Backend docs](https://docs.openclaw.ai/concepts/memory#qmd-backend-experimental) for full configuration options