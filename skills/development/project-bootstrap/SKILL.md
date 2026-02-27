---
name: project-bootstrap
version: 1.0.0
description: Initial project analysis after vendoring — detects stack, conventions, and generates a project-overview skill.
triggers: [bootstrap, init, setup, first-run, project-setup]
---

# Project Bootstrap

Performs initial project analysis when the AI coding protocols are first vendored into a project. Scans the codebase to detect the technology stack, architectural patterns, and key conventions, then generates an initial project-overview skill and an adapted blueprint.

## When to Activate
- Right after `make ai` installation into a new project
- When the agent first encounters an `.ai/` directory without project-specific skills
- When explicitly asked to bootstrap or initialize AI protocols for the project

## Core Concepts
- **Idempotent** — checks for existing project skills before generating; never overwrites previous work
- **Detection over assumption** — scans actual project files rather than guessing based on directory names
- **Minimal initial output** — generates one project-overview skill and one blueprint; the `skill-generator` skill handles ongoing growth
- **Blueprint-aware** — generates tool-specific configuration (CLAUDE.md or `.cursor/rules/`) referencing relevant skills

## Detailed Guidance

### Guard Clause

Before running any analysis, check:

1. Does `.ai/skills/project/` exist and contain at least one SKILL.md file?
2. If yes → **skip bootstrap**. Inform the user that project-specific skills already exist and suggest using `skill-generator` for new additions.
3. If no → proceed with bootstrap.

### Phase 1: Project Scan

Scan these files and directories to build a project profile:

| Source | What to detect |
|--------|---------------|
| `package.json` | Node.js/JS/TS project, framework (Next.js, Nuxt, Express, etc.), test runner (Jest, Vitest, Playwright), linter/formatter |
| `composer.json` | PHP project, framework (Laravel, Symfony), testing (PHPUnit, Pest), coding standards (PHP-CS-Fixer, PHPStan) |
| `pyproject.toml` / `requirements.txt` | Python project, framework (Django, FastAPI, Flask), testing (pytest), tools (ruff, mypy) |
| `go.mod` | Go project, major dependencies |
| `Cargo.toml` | Rust project, major crates |
| `Gemfile` | Ruby project, framework (Rails, Sinatra) |
| Top-level directories | `src/`, `app/`, `lib/`, `tests/`, `spec/`, `docs/`, `infra/`, `packages/` — architectural signals |
| Config files | `.eslintrc`, `tsconfig.json`, `phpstan.neon`, `.prettierrc`, `docker-compose.yml`, `Makefile`, `turbo.json` — tooling signals |
| Monorepo indicators | `turbo.json`, `pnpm-workspace.yaml`, `lerna.json`, `packages/` — monorepo detection |

### Phase 2: Pattern Detection

From the scan results, classify:

- **Language(s):** Primary and secondary
- **Framework:** Web framework, CLI framework, library, etc.
- **Architecture:** Monorepo, MVC, hexagonal/ports-and-adapters, serverless, microservices
- **Testing setup:** Runner, assertion style, coverage tool
- **Code quality:** Linter, formatter, static analysis
- **Key directories:** Where source, tests, config, and infrastructure live

### Phase 3: Generate Project Overview Skill

Create `.ai/skills/project/project-overview/SKILL.md` with:

```markdown
---
name: project-overview
version: 1.0.0
description: Project stack, conventions, and key file paths for {project-name}.
triggers: [project, overview, stack, conventions]
---

# Project Overview: {project-name}

## Stack
- **Language:** {detected}
- **Framework:** {detected}
- **Testing:** {detected}
- **Linting/Formatting:** {detected}
- **Architecture:** {detected}

## Key Directories
- `src/` — {purpose}
- `tests/` — {purpose}
- ...

## Conventions
- {detected conventions from config files and code patterns}

## Relevant Skills
- {list of generic skills from .ai/skills/ that match this stack}
```

### Phase 4: Generate Blueprint

Generate a tool-specific blueprint based on the detected stack:

**For Claude Code** — suggest content for `CLAUDE.md` in the project root:

```markdown
Follow the engineering standards vendored in ./.ai/skills/.
See ./.ai/blueprints/claude-cli.md for the operational protocol.

Project-specific conventions are in ./.ai/skills/project/.
```

**For Cursor** — generate rules in `.cursor/rules/` (NOT `.cursorrules`, which is deprecated):

- Create `.cursor/rules/ai-protocols.mdc` referencing vendored skills
- Include project-specific stack info from the generated overview

### What NOT to Do

- Don't generate skills for standard language features — that's what the generic skills cover
- Don't guess at conventions that aren't evidenced by config files or code
- Don't generate multiple project skills — start with one overview, let `skill-generator` handle the rest
- Don't modify any existing files outside `.ai/skills/project/` without user confirmation

## Examples

**Scenario:** Bootstrap on a Next.js project with TypeScript, Vitest, and ESLint.

**Detected profile:**
- Language: TypeScript
- Framework: Next.js 14 (App Router)
- Testing: Vitest + Testing Library
- Linting: ESLint + Prettier
- Architecture: App Router with `src/app/`, `src/components/`, `src/lib/`

**Generated `.ai/skills/project/project-overview/SKILL.md`:**
```markdown
---
name: project-overview
version: 1.0.0
description: Next.js 14 App Router project with TypeScript, Vitest, and ESLint.
triggers: [project, overview, stack, conventions]
---

# Project Overview: my-nextjs-app

## Stack
- **Language:** TypeScript 5.x (strict mode)
- **Framework:** Next.js 14 (App Router)
- **Testing:** Vitest + @testing-library/react
- **Linting/Formatting:** ESLint (next/core-web-vitals) + Prettier
- **Architecture:** App Router — src/app/ for routes, src/components/ for UI, src/lib/ for utilities

## Key Directories
- `src/app/` — Next.js App Router pages and layouts
- `src/components/` — Reusable React components
- `src/lib/` — Shared utilities and helpers
- `tests/` — Vitest test files

## Relevant Skills
- `typescript-standard` — TypeScript conventions and TDD patterns
- `recursive-exploration` — Codebase navigation
- `code-review` — Review protocol
```

## Guidelines
1. Always check for existing project skills before generating — never overwrite
2. Base all detections on actual files, not assumptions
3. Generate exactly one project-overview skill and one blueprint suggestion
4. Reference only generic skills that match the detected stack
5. Place all generated skills in `.ai/skills/project/`
6. For Cursor, use `.cursor/rules/` (not `.cursorrules`)
7. Don't modify files outside `.ai/skills/project/` without user confirmation

## Integration
- Related: `skill-generator` (handles ongoing skill creation after bootstrap)
- Related: `recursive-exploration` (used during the scanning phase)
- Outputs: project-overview skill, blueprint suggestions

## Skill Metadata
- Created: 2025-07-01
- Last Updated: 2025-07-01
- Author: didacrios
- Version: 1.0.0
