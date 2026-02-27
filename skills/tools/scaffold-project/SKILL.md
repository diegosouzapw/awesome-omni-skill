---
name: scaffold-project
description: "Bootstrap or review project-level context primitives for Claude Code. First run creates .claude/primitives/ and .claude/skills/project-context/. Re-runs review existing primitives against the codebase, auto-update what changed, and flag what's stale. Invoke with '/scaffold-project' or say 'scaffold project', 'set up project context', 'review project context'."
---

# Scaffold Project

Dual purpose. **First run:** creates the primitive files and session-start skill. **Re-runs:** review existing primitives against the current codebase — auto-updates what changed, flags what's stale, asks about what it can't determine. After scaffolding, `/project-context` runs automatically at session start via the project AGENTS.md.

---

## Phase 0: Pre-check

1. Confirm cwd looks like a project root. Check for `package.json`, `pyproject.toml`, `go.mod`, `Gemfile`, or any framework config file. If none found, ask: "This doesn't look like a project root. Are you in the right directory?"

2. Check if `.claude/primitives/` already exists. If it does, this is a **review run**, not a fresh bootstrap. Inform the user:
   > Primitives already exist — this will be a review pass. Auto-updatable fields (versions, scripts, folder structure) will be refreshed. Manually-curated sections (deployment, gotchas, glossary, architecture) will be flagged if they look stale but not overwritten. Proceed?

3. Determine project type:
   - `package.json` exists → **Node project**. Proceed to Phase 1 (auto-discovery).
   - No `package.json` but other indicators exist (`pyproject.toml`, `go.mod`, `Gemfile`) → **Non-Node project**. Note the language. Ask the user to confirm stack details (auto-population is limited for non-Node). Proceed to Phase 2 with user-provided info.
   - Nothing at all → **New/empty project**. Ask the key questions (see [New Project Questions](#new-project-questions) below). Proceed to Phase 2 with user answers.

---

## Phase 1: Discovery (Node projects — read everything, write nothing)

Read these files. Skip gracefully if missing — don't error, just leave that field blank.

### Config files
- `package.json` — full contents (deps, devDeps, scripts, name, engines)
- `.env.example` or `env.example` — environment variables
- `.nvmrc` — Node version
- `tsconfig.json` or `jsconfig.json` — path aliases, compiler options
- `vite.config.ts` / `vite.config.js` — Vite config (check `server.port`)
- `next.config.js` / `next.config.ts` — Next.js config
- `nuxt.config.ts` — Nuxt config
- `supabase/config.toml` — Supabase project_id
- `tailwind.config.js` / `tailwind.config.ts` — Tailwind config
- `.prettierrc` / `.prettierrc.js` / `.prettierrc.json` — Prettier config
- `eslint.config.js` / `.eslintrc` / `.eslintrc.json` / `.eslintrc.js` — ESLint config
- `README.md` — first 50 lines only

### Lock files (package manager detection)
Check which exists (first match wins): `pnpm-lock.yaml` → pnpm, `yarn.lock` → yarn, `package-lock.json` → npm

### Existing context
- `CLAUDE.md` at project root — extract any GitHub config, conventions, active context, deploy info that should migrate to primitives
- `.claude/` directory contents — note what already exists (settings.json, commands/, etc.) — don't overwrite non-primitive files
- `.claude/AGENTS.md` — **if it exists, read the full file.** Extract: any orient table rows not in the standard template, any existing conventions, any project-specific instructions. These will be merged into the new AGENTS.md (see Phase 2).
- `.claude/skills/` — **list all subdirectories.** Any existing project skill (a directory with a `SKILL.md` inside) that is NOT `project-context` is a project-specific skill that must be preserved. Record the skill name and its first-line description (from its SKILL.md frontmatter `description` field). These will appear in the new AGENTS.md orient table and primitive map.
- `.claude/primitives/` — **if it exists, read ALL primitive files.** This is the baseline for the review diff in Phase 2. Note which optional primitives (glossary.md, architecture.md) already exist.

### Optional primitive signals

Two primitives are optional — only create them if signals are present OR the user opts in.

**glossary.md** — create if ANY of these are true:
- README.md contains a terminology, glossary, or definitions section
- The project domain is specialized (legal, medical, finance, real estate, etc.) — infer from README or package name
- Source files use domain-specific terms that aren't standard programming concepts
- User explicitly opts in

**architecture.md** — create if ANY of these are true:
- Multiple top-level source directories beyond a single `src/` (e.g., `api/`, `workers/`, `packages/`)
- README has an architecture, overview, or system design section
- Project has multiple services or a monorepo structure
- User explicitly opts in

If signals are detected, note them — they'll be used in Phase 2. If borderline, ask: "This looks like it might benefit from a [glossary / architecture] doc. Want me to create one?"

### Convention scan (read 3-5 source files)
- List `src/` directory structure. Note: feature-based (`src/features/`) vs layer-based (`src/components/` + `src/pages/`)
- Read one `.tsx` or `.vue` component file. Note: import order, export style (default vs named), props pattern
- Check for co-located test files (`.test.ts` next to source) vs separate test directory

### Derive from all of the above

Use this mapping. If you can't determine a value, leave it as `<not detected>`.

| Field | How to derive |
|---|---|
| Node version | `.nvmrc` content, or `package.json` `engines.node` |
| Package manager | Lock file detection above |
| Framework | `react` in deps = React. `next` = Next.js. `vue` = Vue. `nuxt` = Nuxt. `svelte` = Svelte |
| Bundler | `vite.config.*` exists = Vite. `next` in deps = Next (built-in). `webpack.config.*` = Webpack |
| TypeScript version | `package.json` devDeps `typescript` version string |
| CSS | `tailwindcss` in deps or `tailwind.config.*` = Tailwind. `styled-components` in deps. `*.module.css` files in src = CSS Modules |
| State management | `@tanstack/react-query` = TanStack Query. `zustand` = Zustand. `redux` or `@reduxjs/toolkit` = Redux |
| Routing | `react-router-dom` version. Or `next` = Next App/Pages Router |
| Supabase refs | `supabase/config.toml` `project_id`. Or `.env.example` `SUPABASE_*` patterns |
| Test runner | `vitest` = Vitest. `jest` = Jest. `@playwright/test` = Playwright (E2E) |
| Dev port | Vite default 5173. Next default 3000. Check `vite.config` `server.port` for override |
| Env vars | `.env.example` contents. Redact any actual secret values |
| Scripts | All entries from `package.json` `scripts` — note what each one does based on name and command |
| Folder structure | `src/features/` = feature-based. `src/components/` + `src/pages/` = layer-based |

---

## New Project Questions

If the project is new/empty, ask these questions:

- What framework? (React / Next.js / Vue / Nuxt / other)
- Supabase? (yes / no)
- Test runner? (vitest / jest / none)
- TypeScript? (yes / no)
- CSS approach? (Tailwind / CSS Modules / other)

Use answers to populate primitives. Leave sections that don't apply as `<not applicable>`.

---

## Phase 2: Write or Review Primitives

**Two paths. Pick the right one.**

### If this is a FRESH scaffold (no existing primitives)
Write all files from scratch using discovery data or user answers. Templates and structure are below.

### If this is a REVIEW run (primitives already exist)
Don't rewrite from scratch. For each primitive, diff discovered state against the existing file. Apply this logic:

| Field type | Action |
|---|---|
| **Auto-updatable** — derived directly from codebase | Update silently. These are: versions, package manager, framework, bundler, scripts, dev port, folder structure, component patterns. |
| **Manually-curated** — written by hand, can't be re-derived | Flag if stale, don't overwrite. These are: Deployment, Environment Variables, Common Gotchas, MCP Servers, Key Dependencies, Glossary terms, Architecture descriptions. |
| **Never touch on review** | `active-context.md` — this is the live session scratchpad. |

After updating, present a **staleness report** — list every section that looks outdated with a one-line note on why. The user can then `/remember` updates or edit manually.

---

The file templates and structure follow. Use them for fresh scaffolds. For review runs, use them as the reference for what each file *should* look like.

### Create `.claude/` directory structure

```bash
mkdir -p .claude/primitives
mkdir -p .claude/skills/project-context
mkdir -p .claude/rules
mkdir -p .cursor/rules
```

Don't touch anything else in `.claude/` that already exists.

### Create workflow enforcement rules

These ensure the workflow manager triggers before the agent responds. Create both files — they target Claude Code and Cursor respectively.

**`.claude/rules/workflow-manager.md`:**

```markdown
# Workflow Manager Protocol

BEFORE responding to any work request, you MUST:

1. Announce: "🎯 Workflow Manager active. Checking project state..."
2. Read and follow `~/.claude/skills/workflow-router/SKILL.md`
3. Use the Quick Start checklist to determine current state
4. Report findings and propose the next skill
5. Wait for human confirmation

Do NOT skip this. Do NOT answer the question first. Orient, then act.
```

**`.cursor/rules/workflow-manager.mdc`:**

```markdown
---
description: Workflow Manager Protocol - orient before any work
alwaysApply: true
---

BEFORE responding to any work request, you MUST:

1. Announce: "🎯 Workflow Manager active. Checking project state..."
2. Read and follow `~/.claude/skills/workflow-router/SKILL.md`
3. Use the Quick Start checklist to determine current state
4. Report findings and propose the next skill
5. Wait for human confirmation

Do NOT skip this. Do NOT answer the question first. Orient, then act.
```

### Write `.claude/AGENTS.md` (merge-aware)

This is the only file scaffold-project actively merges rather than overwrites. If an existing `.claude/AGENTS.md` was read in Phase 1, preserve its unique content. If not, write the template clean.

**Merge rules:**
1. The session-start blockquote, "Orient Here" heading, and "Conventions" section always come from the template below — these are the prescribed structure. Do not keep old versions of these.
2. The orient table starts with the 6 standard rows (primitives + `/remember` + global skills). Then **append any rows from the old orient table that are not duplicates** — these are project-specific orient entries the team added manually.
3. The Primitive Map always shows the 4 standard primitives. Then **append any existing project skills** discovered in Phase 1 (name + one-line description from their SKILL.md frontmatter).
4. If the old AGENTS.md had any sections beyond Orient Here, Conventions, and Primitive Map (e.g. project-specific notes, team conventions, runbooks), **preserve them as-is below the Primitive Map.**

**Template (fill in, then apply merge rules above):**

```markdown
# <Project Name> — Agent Context

> **MANDATORY FIRST ACTION:** Before responding to ANY work request, read `~/.claude/skills/workflow-router/SKILL.md` and follow its Quick Start checklist. Announce `🎯 Workflow Manager active. Checking project state...`, orient, then propose the next skill. Do NOT skip this step.

Project-level entry point for Claude Code.

---

## Orient Here

| If you need to... | Read this |
|---|---|
| Know the tech stack and versions | `.claude/primitives/stack.md` |
| Know coding conventions for this project | `.claude/primitives/conventions.md` |
| Know how to run dev locally | `.claude/primitives/local-dev.md` |
| Check what's currently in progress | `.claude/primitives/active-context.md` |
| Store a fact or instruction | Use `/remember` |
| Use global workflow skills | `~/.claude/skills/AGENTS.md` |
<!-- OPTIONAL: add if glossary.md was created -->
<!-- | Look up domain terms or acronyms | `.claude/primitives/glossary.md` | -->
<!-- OPTIONAL: add if architecture.md was created -->
<!-- | Understand system architecture and data flow | `.claude/primitives/architecture.md` | -->
<!-- MERGE: append non-duplicate rows from old orient table here -->

---

## Conventions

1. **Primitives are factual snapshots.** They describe what the project IS. Update them when the project fundamentally changes, or use `/remember` for one-off facts.
2. **active-context.md is the scratchpad.** The only primitive that changes every session. Update it as you work.
3. **Global skills handle workflow.** `/interview`, `/implement-direct`, `/diagnose`, `/qa-handoff` etc. live at `~/.claude/skills/`. Don't re-implement them here.
4. **Other primitives are on-demand.** Don't load stack.md, conventions.md, or local-dev.md at session start. Read them when you actually need them (e.g., before writing code, before running dev).

---

## Primitive Map

```
primitives/
  stack.md            ← frameworks, versions, key dependencies, MCP servers
  conventions.md      ← file naming, component patterns, import order
  local-dev.md        ← commands, ports, env vars, scripts, deployment
  active-context.md   ← current branch, active issues, decisions [MUTABLE — update every session]
<!-- OPTIONAL: add if glossary.md was created -->
  glossary.md         ← domain terms, acronyms, jargon
<!-- OPTIONAL: add if architecture.md was created -->
  architecture.md     ← system structure, data flow, module relationships
```
<!-- MERGE: append existing project skills here, e.g.:
skills/
  my-skill/           ← <description from SKILL.md frontmatter>
-->
<!-- MERGE: append any extra sections from old AGENTS.md here -->
```

### Write `.claude/primitives/stack.md`

```markdown
# Tech Stack

**Last Updated:** <today's date YYYY-MM-DD>

---

## Runtime

- **Node:** <version>
- **Package Manager:** <npm | yarn | pnpm>

## Frontend

- **Framework:** <React <version> | Next.js <version> | Vue <version> | ...>
- **Bundler:** <Vite <version> | Webpack | Next.js built-in>
- **Language:** <TypeScript <version> | JavaScript>
- **UI Components:** <shadcn/ui | Radix | none | ...>
- **CSS:** <Tailwind CSS <version> | CSS Modules | styled-components | ...>
- **State:** <TanStack Query <version> | Zustand <version> | Redux | none>
- **Routing:** <React Router v<N> | Next.js App Router | Next.js Pages Router | ...>

## Backend / Data

- **Database:** <Supabase | Firebase | none | ...>
  - Dev project ref: <from config.toml or .env.example, or "not found">
- **Auth:** <Supabase Auth | NextAuth | none | ...>
- **Edge Functions:** <yes | no>

## Testing

- **Unit/Integration:** <Vitest | Jest | none>
- **E2E:** <Playwright | Cypress | none>
- **Test command:** `<from package.json scripts.test>`

## Tools / MCP Servers

<!-- Add MCP servers and tools here. Use /remember to update. -->

- <list any known MCP servers or tools, or "None configured">

## Key Dependencies

<!-- Non-obvious dependencies an agent might not know about. Use /remember to add. -->

- <list, or "None noted">
```

### Write `.claude/primitives/conventions.md`

```markdown
# Coding Conventions

**Last Updated:** <today's date YYYY-MM-DD>

---

## File and Folder Structure

- **Organization:** <Feature-based (`src/features/<name>/`) | Layer-based (`src/components/`, `src/pages/`)>
- **Component files:** <PascalCase (`MyComponent.tsx`) | camelCase>
- **Utility files:** <camelCase (`formatDate.ts`) | ...>
- **Test files:** <co-located with `.test.ts` suffix | in `__tests__/` subdirs | in `test/` at root>

## Imports

- **Order:**
  1. Node/external packages
  2. <Path-aliased internal modules (e.g., `@/components/...`), if aliases exist>
  3. Relative imports
- **Path aliases:** <list from tsconfig.json paths, or "None">

## Components

- **Export style:** <Named export (`export const`) | Default export (`export default`)>
- **Props pattern:** <Inline interface above component | Separate `type Props = {...}`>

## Error Handling

- <Describe the pattern observed: try/catch with toast? Error boundaries? Thrown errors vs returned objects?>

## Commit Style

- Conventional commits: `feat:`, `fix:`, `chore:`, etc.

## Linting / Formatting

- **Linter:** <ESLint — config at <path>>
- **Formatter:** <Prettier — config at <path>>
- **Key rules:** <Any non-default rules that would trip up an agent, or "Standard defaults">
```

### Write `.claude/primitives/local-dev.md`

```markdown
# Local Development

**Last Updated:** <today's date YYYY-MM-DD>

---

## Quick Start

```bash
# 1. Install dependencies
<npm install | yarn | pnpm install>

# 2. Set up environment
cp .env.example .env.local
# Edit .env.local with actual values (see Environment Variables below)

# 3. Start backend (if applicable)
<supabase start — or remove this step if not applicable>

# 4. Start dev server
<npm run dev | yarn dev | pnpm dev>
```

## Dev Server

- **Command:** `<from package.json scripts.dev>`
- **URL:** http://localhost:<port>

## Backend / Database

<!-- Remove this section if not applicable -->

- **Start:** `supabase start`
- **Local URL:** http://127.0.0.1:<port>
- **Studio:** http://127.0.0.1:<studio-port>
- **Reset DB:** `supabase db reset`

## Environment Variables

Required for local dev. See `.env.example` for full list.

| Variable | Purpose |
|----------|---------|
| <NAME> | <what it does> |

## Scripts

What each `package.json` script does:

| Script | Purpose |
|--------|---------|
| `<script name>` | <what it does> |

<!-- Add more scripts as you discover them. Use /remember to update. -->

## Testing

| What | Command |
|------|---------|
| Unit tests | `<from package.json scripts.test>` |
| Single test | `<e.g., npx vitest <pattern>>` |
| <E2E tests — if applicable> | `<command>` |

## Deployment

<!-- Add deploy commands, CI/CD info, server details here. Use /remember to update. -->

- <not yet documented>

## Build

```bash
<npm run build | yarn build | pnpm build>
```

## Common Gotchas

<!-- Add known issues and lessons learned here. Use /remember to update. -->

- <none yet>
```

### Write `.claude/primitives/active-context.md`

```markdown
# Active Context

> Updated every session. Single source of truth for what is happening right now.

---

## Current State

- **Branch:** <current git branch>
- **Sprint / Focus:** <not set>

## Active Issues

| # | Title | Status | Notes |
|---|-------|--------|-------|
|   |       |        |       |

## Recent Decisions

| Date | Decision | Rationale |
|------|----------|-----------|

## Blockers

- <none>

---

> **Usage:** This is read automatically at session start (see AGENTS.md). Update Active Issues and Blockers as you work. Log decisions in Recent Decisions. Use `/remember` to add facts to other primitives.
```

### Write `.claude/primitives/glossary.md` (optional)

Only create if detection signals were found in Phase 1 or user opted in. Seed with terms discovered during convention scan and README review — don't leave it empty, an empty glossary is worse than no glossary.

```markdown
# Glossary

> Domain terms, acronyms, and jargon for this project. If a term could be misunderstood or conflicts with common usage, it belongs here. Use `/remember` to add terms as you encounter them.

**Last Updated:** <today's date YYYY-MM-DD>

---

| Term | Means | Notes |
|------|-------|-------|
| <term> | <plain-English definition> | <where it appears, or common misinterpretation to avoid> |
```

### Write `.claude/primitives/architecture.md` (optional)

Only create if detection signals were found in Phase 1 or user opted in. Focus on the structure that an agent wouldn't be able to infer from just reading files — how do the pieces connect? What's the data flow? Where are the boundaries?

```markdown
# Architecture

> How this system is structured and how the pieces connect. Not what frameworks we use (that's stack.md) — this is WHY things are organized the way they are.

**Last Updated:** <today's date YYYY-MM-DD>

---

## System Overview

<1-3 sentence summary of what the system does and its main components>

## Module Map

<Describe the main modules/directories and what each one owns. Use a simple list or diagram — not a full file tree.>

## Data Flow

<How data moves through the system. Key: where do user inputs enter? Where does data get stored? What triggers what? Focus on the non-obvious paths.>

## Key Boundaries

<Where are the important seams? e.g., "Frontend talks to backend only via these API routes." "Auth state is managed here, everything else reads from here." These are the things that bite new contributors.>

## What NOT to Change Without Thinking

<Architectural constraints — things that look like they could be refactored but actually can't, or that have non-obvious dependencies. Use `/remember` to add as you discover them.>
```

### Write `.claude/skills/project-context/SKILL.md`

```markdown
---
name: project-context
description: "Full project orientation. Reads active context, checks git state, and presents a summary. Also runs automatically via AGENTS.md at session start — invoke explicitly with '/project-context' if you need a fresh orientation mid-session or want to update active-context."
allowed-tools: Read, Grep, Glob, Bash
---

# Project Context

Full orientation for this project. Runs automatically at session start via AGENTS.md. Invoke explicitly with `/project-context` if you need a fresh read mid-session.

## Instructions

### Step 1: Read Active Context

Read `.claude/primitives/active-context.md`.

### Step 2: Check Git State

Run:
```bash
git status
git branch --show-current
git log --oneline -5
```

### Step 3: Update Active Context

If the current branch differs from what's recorded, update the **Branch** field in active-context.md.

### Step 4: Present Summary

Output this structure:

---

**Project:** <name>
**Branch:** <current branch>

**Active Issues:**
<table from active-context, or "None tracked">

**Recent Decisions:**
<last 3 from active-context, or "None recorded">

**Blockers:**
<from active-context, or "None">

---

**Need more context?**
- Stack & versions → `.claude/primitives/stack.md`
- Coding conventions → `.claude/primitives/conventions.md`
- Dev commands & scripts → `.claude/primitives/local-dev.md`

Ready. What would you like to work on?

---

## What This Skill Does NOT Do

- Does not load stack.md, conventions.md, or local-dev.md (those are on-demand)
- Does not start dev servers
- Does not pull from git
```
```

---

## Phase 3: Confirm

Print a summary appropriate to whether this was a fresh scaffold or a review run.

### Fresh scaffold output:
```
Scaffold complete.

Created:
  .claude/rules/workflow-manager.md    ← Claude Code workflow enforcement
  .cursor/rules/workflow-manager.mdc   ← Cursor workflow enforcement
  .claude/AGENTS.md              ← project entry point
  .claude/primitives/stack.md    ← <one-line stack summary>
  .claude/primitives/conventions.md  ← <org pattern, key conventions>
  .claude/primitives/local-dev.md    ← <dev command, port>
  .claude/primitives/active-context.md  ← empty, fill as you work
  .claude/skills/project-context/SKILL.md ← explicit orientation skill
  <if created> .claude/primitives/glossary.md     ← <N terms seeded>
  <if created> .claude/primitives/architecture.md ← <one-line summary>

Session start is automatic — AGENTS.md instructs the agent to orient on first load.
Use /remember to add facts as you discover them.
```

### Review run output:
```
Review complete.

Auto-updated:
  <list each field that was refreshed and what changed, e.g. "stack.md: TypeScript 5.1 → 5.3">
  (or "Nothing changed — primitives are current." if nothing moved)

⚠ Stale (needs your attention):
  <list each section that looks outdated with a one-line note, e.g. "local-dev.md Deployment: last updated 2025-11-02, no deploy commands found in package.json">
  (or "Nothing flagged." if everything looks fresh)

Preserved (merged into AGENTS.md):
  <list any existing project skills and custom orient rows carried over>
  (or "Nothing to merge." if clean)

Use /remember to update any flagged sections.
```

---

## What This Skill Does NOT Do

- Does not modify or delete existing `CLAUDE.md`
- Does not touch existing `.claude/` files other than `AGENTS.md` (settings.json, commands/, rules/, existing project skills, etc. are all preserved)
- Does not delete or overwrite existing project skills in `.claude/skills/` — they are discovered and referenced in the new AGENTS.md
- Does not add itself to any workflow chain (no contract)
- Does not need to be run again unless the project fundamentally changes
