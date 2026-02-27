---
name: x-bookmarks-manager
description: Read X/Twitter bookmarks using the bird CLI, verify the active account via `bird whoami`, and return bookmarks as JSON. Use when the user asks to list, search, summarize, or export X bookmarks, or when you must validate the active X account before fetching bookmarks.
---

# X Bookmarks Manager

## Overview
Fetch X bookmarks via a local bird CLI install and return JSON for the agent to summarize or filter.

## Workflow (agent-driven)
1. Ensure bird is installed locally via `scripts/setup.sh` (installs with npm into `.skills-data`).
2. Ensure `BIRD_EXPECTED_USER` is set in `.skills-data/x-bookmarks-manager/.env`.
3. Run `scripts/x-bookmarks` to validate the account and output JSON.
4. Parse or summarize the JSON for the user.

## Commands
- `scripts/x-bookmarks`: validate `bird whoami` and print `bird bookmarks --json`.
- `scripts/setup.sh`: install bird in `.skills-data/x-bookmarks-manager/venv/node_modules/.bin/bird` and create `.env`.

## Output
- Raw JSON from `bird bookmarks --json` on stdout.

## Account safety
- Always run `bird whoami` before fetching bookmarks.
- If the active account does not match `BIRD_EXPECTED_USER`, stop and ask the user to fix login or update `.env`.

## Local data and env
- Store all mutable state under <project_root>/.skills-data/<skill-name>/.
- Keep config and registries in .skills-data/<skill-name>/ (for example: config.json, <feature>.json).
- Use .skills-data/<skill-name>/.env for SKILL_ROOT, SKILL_DATA_DIR, and any per-skill env keys.
- Install local tools into .skills-data/<skill-name>/bin and prepend it to PATH when needed.
- Install dependencies under .skills-data/<skill-name>/venv:
  - Python: .skills-data/<skill-name>/venv/python
  - Node: .skills-data/<skill-name>/venv/node_modules
  - Go: .skills-data/<skill-name>/venv/go (modcache, gocache)
  - PHP: .skills-data/<skill-name>/venv/php (cache, vendor)
- Write logs/cache/tmp under .skills-data/<skill-name>/logs, .skills-data/<skill-name>/cache, .skills-data/<skill-name>/tmp.
- Keep automation in <skill-root>/scripts and read SKILL_DATA_DIR (default to <project_root>/.skills-data/<skill-name>/).
- Do not write outside <skill-root> and <project_root>/.skills-data/<skill-name>/ unless the user requests it.

## Notes
- bird is installed under `.skills-data/x-bookmarks-manager/venv/node_modules/.bin/bird` and invoked directly.
- Network access is required to download bird from npm on first setup.
- If bird is installed manually (npm/pnpm/bun), set `BIRD_BIN` in `.env` to the correct path.
- If bird is not authenticated, ask the user to log in with bird and retry.
