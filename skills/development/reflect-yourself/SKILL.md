---
name: reflect-yourself
description: Self-learning system that captures corrections, discovers workflow patterns, and syncs learnings to skills and rules. Use when ending a session, after corrections, or when the user wants to formalize learnings (v1.0.5).
---

# reflect-yourself

A self-learning system for Cursor that captures corrections and syncs them to the right place.

## Commands

When the user invokes these commands, read the corresponding file in `commands/`:

| Command | File |
|---------|------|
| `/reflect-yourself` | `commands/reflect-yourself.md` |
| `/reflect-yourself-skills` | `commands/reflect-yourself-skills.md` |
| `/reflect-yourself-queue` | `commands/reflect-yourself-queue.md` |
| `/reflect-yourself-skip` | `commands/reflect-yourself-skip.md` |

## Quick Start

1. At session end, run `/reflect-yourself`
2. Review captured learnings
3. Approve, edit, or skip each one
4. Learnings are applied to skills/rules

## Placement Logic

- **Project Skills** (`.cursor/skills/`) → Codebase-specific
- **Personal Skills** (`~/.cursor/skills/`) → Reusable across projects
- **Project Rules** (`.cursor/rules/`) → Coding standards

## Queue Location

Pending learnings: `~/.cursor/reflect-queue.json` (global location, never pollutes project repos)
