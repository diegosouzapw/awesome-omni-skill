---
name: initialize-repo
description: Analyze the codebase and generate an INIT.md onboarding file for AI coding assistants. Also creates or updates AGENTS.md. Use when the user wants to initialize a repo for AI-assisted development, generate INIT.md, create CLAUDE.md, or bootstrap AI context files.
---

# Initialize Repository for AI Assistants

Generate an `INIT.md` and `AGENTS.md` at the repo root by deeply analyzing the codebase. These files onboard future AI coding assistant instances so they can be productive immediately.

## Phase 1: Discovery

Thoroughly explore the codebase before writing anything. Read files in parallel where possible.

### 1.1 Existing documentation

Read these files if they exist (skip silently if missing):

- `README.md`
- `CLAUDE.md`
- `AGENTS.md`
- `INIT.md`
- `CONTRIBUTING.md`
- `docs/` directory (scan for key files)

### 1.2 Existing AI/editor rules

Read these if they exist:

- `.cursor/rules/` (all files)
- `.cursorrules`
- `.github/copilot-instructions.md`

### 1.3 Build & config files

Read root-level config files to extract commands and tech stack:

- `package.json`, `package-lock.json` or `yarn.lock` or `pnpm-lock.yaml`
- `pyproject.toml`, `setup.py`, `setup.cfg`, `requirements.txt`, `Pipfile`
- `Cargo.toml`
- `go.mod`
- `Makefile`, `Justfile`, `Taskfile.yml`
- `docker-compose.yml`, `Dockerfile`
- `nx.json`, `turbo.json`, `lerna.json` (monorepo configs)
- CI configs: `.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`
- Linter/formatter configs: `.eslintrc*`, `.prettierrc*`, `ruff.toml`, `pyproject.toml [tool.*]`, `rustfmt.toml`, `.golangci.yml`

### 1.4 Architecture mapping

Explore top-level source directories (e.g., `src/`, `app/`, `backend/`, `frontend/`, `services/`, `packages/`, `lib/`, `cmd/`):

- Identify entry points (main files, index files, route definitions)
- Identify service boundaries, API layers, data models
- Map the data flow: where requests enter, how they are processed, where data is stored
- Note key abstractions and patterns (middleware, plugins, dependency injection, etc.)

Do NOT read every file. Focus on entry points, config, and structural files to build a mental model.

## Phase 2: Generate INIT.md

Create `INIT.md` at the repository root. Follow these rules strictly.

### Required preamble

The file MUST begin with this exact text block:

```
You can write your own and get the same result. I use a custom /initialize on new repos to get up and running fast. I tweaked it to work across different coding agents and sprinkled in a few tips I collected along the way.

It should create a relevant AGENTS.md; if one exists, it updates it. Save this prompt as a custom command and use it with any tool — Gemini CLI, Codex, Amp, Firebender, etc. You aren't stuck with any single tool.

One more tip: a reasoning model works best for these types of commands.
```

Follow the preamble with a horizontal rule (`---`), then the content sections below.

### Section 1: Commands

Header: `## Commands`

List the concrete commands needed to develop in this codebase. Extract these from config files, Makefiles, READMEs, and CI configs. Include:

- How to install dependencies
- How to build / compile
- How to run the dev server
- How to run the full test suite
- How to run a single test file or test case
- How to lint and format
- How to run type checks
- Any other common commands (migrations, codegen, Docker, etc.)

Only include commands that actually exist in the project. Do not invent commands.

### Section 2: Architecture

Header: `## Architecture`

Describe the high-level architecture that requires reading multiple files to understand. Focus on:

- System boundaries (frontend/backend, microservices, monorepo packages)
- Request/data flow from entry point to storage
- Key abstractions, patterns, and conventions that are not obvious from a single file
- How modules/packages/services connect to each other
- Technology choices and why they matter for navigating the code

Keep it concise. This is the "big picture" overview, not a file listing.

### Section 3: Conventions (conditional)

Header: `## Conventions`

Only include this section if there are non-obvious project-specific conventions found in rules files, README, or CONTRIBUTING.md. Examples:

- Naming conventions that differ from language defaults
- Required patterns for new features (e.g., "every API endpoint needs a corresponding schema")
- Architectural constraints (e.g., "no direct database access from route handlers")
- Import ordering or module organization rules

If no meaningful conventions are found, omit this section entirely.

### Anti-patterns -- DO NOT include any of the following

- Generic development advice ("write tests", "use meaningful names", "handle errors gracefully")
- Obvious instructions ("do not commit secrets", "provide helpful error messages")
- Exhaustive file trees or component listings (these are easily discoverable)
- Made-up sections like "Common Development Tasks", "Tips for Development", "Support and Documentation" unless content for them was explicitly found in project files
- Information that repeats what is already stated in another section
- Placeholder or aspirational content -- only document what actually exists

## Phase 3: Create or Update AGENTS.md

### If AGENTS.md does NOT exist

Create `AGENTS.md` at the repo root. This file is specifically for AI coding agents. Include:

- A brief one-liner about the project
- Agent-specific workflow hints (e.g., "run tests with X before committing", "use Y for database migrations")
- References to INIT.md for full context
- Any tool-specific notes extracted from existing rules files

Keep it short (under 50 lines). Agents will also read INIT.md for full context.

### If AGENTS.md already exists

Read the existing file, then merge new findings:

- Add any missing commands or architecture notes as new sections
- Do not remove existing content
- Do not duplicate information already present
- Add a comment noting what was updated and when

## Phase 4: Handle Existing CLAUDE.md

### If CLAUDE.md exists

- Read it fully
- Compare its content against the generated INIT.md
- Suggest concrete improvements (missing commands, outdated architecture notes, redundant content)
- Present suggestions to the user -- do NOT overwrite CLAUDE.md

### If CLAUDE.md does NOT exist

- No action needed. INIT.md serves the same purpose.

## Quality Checklist

Before finishing, verify:

- [ ] INIT.md starts with the required preamble text
- [ ] All listed commands were extracted from actual project files (none invented)
- [ ] Architecture section describes the big picture, not individual files
- [ ] No generic/obvious development advice is included
- [ ] No made-up sections exist
- [ ] No repetition between sections
- [ ] AGENTS.md was created or updated
- [ ] If CLAUDE.md existed, improvement suggestions were provided
