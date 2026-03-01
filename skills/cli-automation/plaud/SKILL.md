---
name: plaud
description: Export and download Plaud recordings (transcripts, summaries, audio) using the plaud CLI with safe auth handling.
---

# Plaud CLI skill

Use this skill when you need to authenticate to Plaud and export/download recordings from `app.plaud.ai` / `api.plaud.ai`.

## Hard constraints (security)

- Never print or paste a full Plaud auth token into chat/logs.
- Never pass tokens via CLI flags. Use `plaud auth login`, `plaud auth set --stdin`, or `PLAUD_AUTH_TOKEN`.
- Prefer `--json` outputs and keep stdout machine-readable where available.

## JSON contract

- For stable machine-readable behavior, follow `docs/CONTRACT_V1.md` (in this repo).

## Setup (once)

- If `plaud` isn’t installed globally, use `npx -y plaud ...` (slower but zero-setup).
- `plaud auth login`
- Verify: `plaud auth status --json` and `plaud doctor --json`

Fallbacks:
- `plaud auth import-har /path/to/web.plaud.ai.har`
- `plaud auth set --stdin`

## Common workflows

- List: `plaud recordings list --json --max 50`
- Get one: `plaud recordings get <id> --json`
- Download one: `plaud recordings download <id> --out ./plaud-download --what transcript,summary,json`
- Download audio: `plaud recordings download <id> --out ./plaud-download --what audio --audio-format opus`
- Bulk export: `plaud recordings export --zip`
- Trash: `plaud recordings trash <id>`
- Restore: `plaud recordings restore <id>`
- Tags (list): `plaud recordings tags list --json`
- Tags (add): `plaud recordings tags add <tagId> <id>`
- Tags (clear): `plaud recordings tags clear <id>`
- Re-run transcript/summary: `plaud recordings rerun <id> --wait`
- Recording speaker labels (list): `plaud recordings speakers list <id> --json`
- Recording speaker labels (rename): `plaud recordings speakers rename <id> --from "Speaker 2" --to "Yash Patel"`

## Notes

- `plaud recordings export` prints a JSON summary to stdout; progress goes to stderr.
- When in doubt: run `plaud doctor --json` to confirm auth + API access.
