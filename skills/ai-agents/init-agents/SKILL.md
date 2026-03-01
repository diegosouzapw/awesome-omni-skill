---
name: init-agents
description: "Initialize or update AGENTS.md (or CLAUDE.md for Claude Code) with AI agent guidance. Use when user says 'init agents', 'create AGENTS.md', 'setup agent instructions', or wants project-specific AI coding assistant configuration."
---

# Init Agents

Initialize or update `AGENTS.md` at project root — the instruction manual that tells AI coding assistants exactly how to work in this project.

## What AGENTS.md Is

A **vendor-agnostic markdown file** that provides persistent, project-specific guidance to AI coding agents. Codex, OpenCode, Cursor, and GitHub Copilot read it. Claude Code uses `CLAUDE.md` instead — use the same content; the filename differs by tool.

**Contains:**
- Clear dos and don'ts (tech stack, versions, patterns)
- Executable commands (file-scoped type-check, lint, format, test)
- Project structure hints and key file locations
- Safety and permission boundaries
- Code style examples (good vs bad)
- Git workflow and PR checklist

**Does NOT contain:**
- Business rules (that belongs in `KNOWLEDGE.md`)
- Product roadmap (not included in this document)

## Tool Conventions

| Tool | File | Location |
|------|------|----------|
| Codex | `AGENTS.md` | Project root, `~/.codex/AGENTS.md` global |
| OpenCode | `AGENTS.md` | `~/.config/opencode/AGENTS.md` global |
| Cursor | `AGENTS.md` or `.cursor/rules/` | Project root |
| Claude Code | `CLAUDE.md` | Project root |
| GitHub Copilot | `AGENTS.md` | `.github/agents/*.md` for specialized agents |

Project root `AGENTS.md` applies to Codex, Cursor, Copilot. Claude Code expects `CLAUDE.md`. When user mentions Claude Code specifically, create/update `CLAUDE.md`; otherwise use `AGENTS.md`.

## Six Core Areas (Best Practice)

Effective agent files cover all six:

1. **Commands** — Executable commands with flags (put early in the file)
2. **Testing** — How to run tests, test-first expectations
3. **Project structure** — Key paths, where things live
4. **Code style** — Naming, formatting, patterns with examples
5. **Git workflow** — Commit format, PR checklist
6. **Boundaries** — Allowed / ask first / never

## Format

```markdown
# AGENTS.md

This file provides guidance to AI coding agents working in this repository.

## Project Overview
[1-2 sentences: what this project is, key stack]

## Commands
# Type-check single file
npm run tsc --noEmit path/to/file.ts

# Lint single file
npm run eslint --fix path/to/file.ts

# Format single file
npm run prettier --write path/to/file.ts

# Run tests (single file or suite)
npm test -- path/to/file.test.ts

## Project Structure
- `src/` — [purpose]
- `docs/` — [purpose]
- [key files that define architecture]

## Do
- [specific rule with versions/libraries]
- [specific rule]

## Don't
- [specific prohibition]
- [specific prohibition]

## Safety and Permissions
**Allowed without prompt:** [read files, run lint/format on single file, run single test]
**Ask first:** [package installs, git push, full build, schema changes]
**Never:** [commit secrets, edit vendor/, modify production configs]

## When Stuck
- Ask a clarifying question or propose a short plan
- Do not push large speculative changes without confirmation
```

## Process

### Step 1: Explore Project

Gather in parallel:
- Tech stack (package.json, composer.json, requirements.txt, go.mod, etc.) — versions matter
- Build/lint/test commands from scripts
- Project structure and key entry points
- Existing rules (`.cursorrules`, `.cursor/rules/`, `.github/copilot-instructions.md`, `CONTRIBUTING.md`)
- `KNOWLEDGE.md`, if it exists — reference it, don't duplicate

### Step 2: Extract Commands

Find file-scoped commands (prefer over full-project builds):
- Type-check: `tsc --noEmit`, `pyright`, etc.
- Lint: `eslint --fix`, `ruff check --fix`, etc.
- Format: `prettier --write`, `black`, etc.
- Test: `vitest run`, `pytest`, `jest`, etc.

Include exact flags. Put commands early in the file.

### Step 3: Define Boundaries

Three tiers:
- **Always do** — Run lint/test on changed files, follow style examples
- **Ask first** — Package installs, git push, full builds, schema changes
- **Never** — Commit secrets, edit vendor/node_modules, modify production configs

### Step 4: Add Code Examples

Point to real files that demonstrate good patterns. Call out legacy code to avoid.
Examples beat paragraphs of description.

### Step 5: Write or Merge

**If `AGENTS.md` (or `CLAUDE.md`) exists:**
1. Read existing content
2. Merge new sections (don't duplicate)
3. Update outdated commands and structure
4. Preserve user customizations

**If it doesn't exist:**
1. Create from template above
2. Fill with discovered project context
3. Keep it concise — expand over time based on agent mistakes

## Companion Documents

| Document | Use When |
|----------|----------|
| `KNOWLEDGE.md` | Business context, domain rules, gotchas — suggest `init-knowledge` if missing |

When running init-agents, if companion docs exist, reference them in AGENTS.md (e.g. "See KNOWLEDGE.md for business context"). Don't duplicate their content.

## Rules

- **Just do it** — no approval needed, write directly
- **Be specific** — "React 18 with TypeScript and Vite" not "React project"
- **Commands first** — put executable commands early, with flags
- **Code examples over prose** — one real snippet beats three paragraphs
- **File-scoped commands** — prefer single-file validation over full builds
- **Never commit secrets** — most common and valuable boundary
- **Hierarchy** — project `AGENTS.md` overrides global; nested `AGENTS.override.md` for subdirs (Codex)
- **Iterate** — start minimal, add detail when agents make mistakes
