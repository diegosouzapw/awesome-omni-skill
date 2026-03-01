---
name: context7-skills
description: Use when a user asks to search, install, list, or remove skills with the Context7 ctx7 skills CLI (including npx ctx7) and needs correct subcommands or client flags.
---

# Context7 Skills

## Overview
Use the Context7 CLI to manage skills via `ctx7 skills` (or `npx ctx7 skills`). Keep commands short, explicit, and limited to search/install/list/remove.

## Scope
- Covers only Context7 CLI skill management: `ctx7 skills search|install|list|remove`.
- Does not cover Context7 MCP docs, agents, or API tools.

## Quick Reference

| Action | Command | Notes |
| --- | --- | --- |
| Search | `ctx7 skills search <query>` | Use multiple keywords if needed. `npx ctx7` is fine. |
| Install | `ctx7 skills install <repo> [skill ...] [--cursor|--claude|--global]` | Repo often `/anthropics/skills`. Omit skill to install interactively. |
| List | `ctx7 skills list [--cursor|--claude|--global]` | Lists installed skills for a client. |
| Remove | `ctx7 skills remove <skill> [--cursor|--claude|--global]` | Removes a skill by name. |

## Implementation

Example session:

```bash
# Search for skills about pytorch
npx ctx7 skills search pytorch

# Install the uv skill from the registry repo
ctx7 skills install /anthropics/skills uv

# List installed skills and find slide-creator in the output
ctx7 skills list --claude
```

## Rationalization Table

| Excuse | Reality |
| --- | --- |
| "I can just use ctx7 search" | The command is `ctx7 skills search`. The `skills` subcommand is required. |
| "This is about docs, so use MCP tools" | This skill is only for CLI skill management. Stick to `ctx7 skills`. |
| "Client flags are optional, I can skip them" | Use `--cursor`, `--claude`, or `--global` when the user specifies a target client. |

## Red Flags - STOP and Correct

- Using `ctx7 search` or `ctx7 install` without the `skills` subcommand
- Answering with MCP docs or API tools instead of CLI commands
- Ignoring a requested client target (Cursor/Claude/global)

## Common Mistakes

- Confusing `ctx7` with `npx ctx7` when the CLI is not installed.
- Omitting the repository path for install (e.g., `/anthropics/skills`).
- Assuming `list` filters by name; it only lists, so scan the output.
