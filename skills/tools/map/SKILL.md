---
name: map
description: Use when starting a new project, onboarding to an existing codebase, or when no architecture doc exists. Generates a codebase architecture doc and CLAUDE.md scaffold by exploring actual source and asking targeted questions.
---

# /map — Codebase Architecture Generator

Generate `docs/codebase-architecture.md` (the map) and `.claude/CLAUDE.md` (the scaffold) for any project.

## Hard Rules

1. **Reference source, don't duplicate it.** Use `file:line` pointers. Only inline a snippet when it's a one-liner with no source location yet.
2. **Tables over prose.** Scannable, dense.
3. **Project vocabulary.** Use the project's own names for layers, patterns, concepts.
4. **ASCII diagrams.** No Mermaid, no images.
5. **Conditional sections.** Include what's relevant, skip what isn't.
6. **Convention notes inline.** Naming patterns, formatting rules, "when to use A vs B" — attach to the relevant section, not a separate section.
7. **Working directory context.** Always specify where to run commands and what paths are relative to.

## Process

### Step 1: Detect

Check for `docs/codebase-architecture.md` and `.claude/CLAUDE.md`.

- Both exist → offer to **audit** (check stale `file:line` refs, missing files, undocumented additions)
- Missing → proceed to bootstrap

### Step 2: Explore

Use subagents to scan config files, auto-detect the stack, and build a complete picture of the codebase — entry points, processing layers, state, file inventory, communication patterns, key types.

### Step 3: Ask targeted questions

Focus on what exploration couldn't answer:

- What is this project? (one sentence)
- What are the 3-5 most common tasks a developer does in this codebase?
- Any key architectural decisions that aren't obvious from the code?
- Any gotchas or landmines you've hit?

### Step 4: Generate architecture doc

Write `docs/codebase-architecture.md` using the section catalog below.

### Step 5: Generate CLAUDE.md scaffold

Write `.claude/CLAUDE.md`:

- **Auto-filled:** Project name + one-line description, validation commands with working directory context, key stack/integration callouts
- **Empty wisdom sections:** Architecture Principles, Learned Patterns, Gotchas — each with a comment showing the expected format (one-line example with `file:line` ref)
- **Excluded:** Generic advice, stack-specific skill references, personal workflow preferences

### Step 6: User reviews and commits

Present both files. User refines. Commit when approved.

---

## Architecture Doc — Section Catalog

### Required: Header

One sentence + key tech + storage/persistence approach.

### Required: Commands

Build, test, lint, format, REPL — whatever the project actually uses. Include working directory context.

### Required: File Inventory

Tables grouped by domain concern (not directory structure). Every source file gets a one-line Role. Within each group, list subsystem enumerations (union types, enum-like constants) and note shared/reused components.

### Conditional: Data Flow

**When:** Layered processing (web apps, event systems, pipelines, CLI with middleware).

**Cover:** Entry points, processing layers, output channels, both request and push directions, naming conventions at boundaries, sync vs async patterns.

### Conditional: State Shape

**When:** Non-trivial state (game servers, real-time, complex frontend, stateful services).

**Format:** Compact key-type tables with `file:line` refs. Do NOT reproduce struct definitions — point at source. Cover: init location, key collections, polymorphic maps (flag explicitly), construction/transformation sites, derived vs stored state.

### Conditional: Event/API Surface

**When:** External-to-internal mapping layer (REST endpoints, LiveView events, GraphQL resolvers, CLI subcommands, message handlers).

**Cover:** Category groupings by domain, surface → internal format mapping, naming conventions, when to use pattern A vs B, push/subscription events.

### Conditional: Module Relationships

**When:** Composition root pattern, dependency graph, or non-obvious module coupling.

**Cover:** Composition roots, dependency direction, inline "why" notes for non-obvious coupling, boundary enforcement patterns.

### Conditional: Common Task Guide

**When:** Repeatable patterns exist for extending the codebase. This is the **highest-value section**.

For each recipe: numbered steps with `file:line` refs, a **Verify** step (the command to run), and a canonical example to follow. Identify recipes from recent commits, test patterns, repeated file-edit patterns, and framework conventions. Include as many recipes as warranted.
