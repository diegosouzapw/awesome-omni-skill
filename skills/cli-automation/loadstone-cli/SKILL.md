---
name: loadstone-cli
description: Use the Loadstone CLI to search the RuneScape 3 Wiki for quests or item information and RuneMetrics for character information like levels, status, quests completion. Use when the user asks for RS3 wiki data, Ironman-focused answers, RuneMetrics stats, or when they want CLI command guidance for Loadstone.
metadata:
  author: loadstone
  version: "1.0"
---

# Loadstone CLI Skill

Use this skill to operate the Loadstone CLI in this repository and return data from the RuneScape 3 Wiki or RuneMetrics in a user-friendly format.

## When to use

- User asks for RS3 Wiki data (pages, categories, tables, training info)
- User asks for RuneMetrics profile or quest info
- User asks how to run or interpret Loadstone CLI commands
- User wants structured JSON output for LLM consumption

## Required context

- Prefer `--json` when the user wants structured output.

## Command usage

> [!IMPORTANT]
> The `loadstone` command MUST be available on the user's PATH. Do not attempt to run it via `bun run`, `npm run`, or by pointing to source files directly.

```bash
loadstone <command> [args] [options]
```

## Profile data workflow

For token-efficient profile queries, use the filtering options:

- Use `--skills-only` for skill level queries (e.g., "what's my crafting level?") - reduces ~14k tokens to ~300
- Use `--completed-quests-only` for completed quest counts
- Use `--started-quests-only` for current quest progress
- Use `--not-started-quests-only` for available quests to start
- Use `--quests-only` for all quests with minimal data
- Add `--include-raw` if you need the complete raw API response

**Note:** The `raw` field is excluded by default in JSON mode to save tokens. Use `--include-raw` only when necessary.

Examples:

```bash
# Get only skill levels
loadstone profile "Some Player" --json --skills-only
```

```bash
# Get only completed quests
loadstone profile "Some Player" --json --completed-quests-only
```

```bash
# Get only started quests
loadstone profile "Some Player" --json --started-quests-only
```

## Page content workflow

- First request available headings with `--headings`.
- Choose only the headings relevant to the user request.
- Request those sections with `--fields "Heading 1","Heading 2"`.

Examples:

```bash
loadstone page "Abyssal whip" --headings
```

```bash
loadstone page "Abyssal whip" --fields "Usage","Drop sources" --json
```

## Output handling

- If the user asks for data, prefer `--json` and return structured results.
- If the user asks for a summary, parse JSON and provide a concise summary.
- If a command fails, return the error and offer a fix (missing command, network, rate limit).
