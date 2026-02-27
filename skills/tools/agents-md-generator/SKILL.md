---
name: agents-md-generator
description: Generate AGENTS.md files for software projects by analyzing project structure, dependencies, and tooling. Use when asked to create an AGENTS.md file, document a project for AI agents, or establish project conventions for agent consumption. Triggers on requests like "create AGENTS.md", "generate agent instructions", "document this project for agents", or "add AI agent documentation".
---

# AGENTS.md Generator

Generate structured AGENTS.md files by analyzing a project's tools, dependencies, and organization.

## Workflow

1. **Identify project root** — Locate `package.json`, `pyproject.toml`, `Cargo.toml`, or similar manifest
2. **Analyze dependencies** — Extract framework, testing, linting, and build tools from manifest
3. **Scan structure** — Map `src/`, `lib/`, `test/`, and other key directories
4. **Detect quality tools** — Identify typecheck, lint, test, and build commands
5. **Generate AGENTS.md** — Output using the template format

## Analysis Checklist

### Detect Project Type

| Manifest File | Language | Common Frameworks |
|---------------|----------|-------------------|
| `package.json` | JavaScript/TypeScript | React, Next.js, Express, Vite |
| `pyproject.toml` / `setup.py` | Python | FastAPI, Django, Flask |
| `Cargo.toml` | Rust | Actix, Axum, Tokio |
| `go.mod` | Go | Gin, Echo, Fiber |
| `pom.xml` / `build.gradle` | Java/Kotlin | Spring, Quarkus |

### Identify Quality Tools

**Typechecking:**
- TypeScript: `tsc --noEmit`
- Python: `mypy`, `pyright`, `ty check`
- Rust: `cargo check`

**Linting:**
- JavaScript/TypeScript: `eslint`, `biome`
- Python: `ruff`, `flake8`, `pylint`
- Rust: `clippy`

**Testing:**
- JavaScript/TypeScript: `vitest`, `jest`, `mocha`
- Python: `pytest`, `unittest`
- Rust: `cargo test`

### Determine Project Classification

- **Library**: Exports modules for other packages (has `exports` or `lib` config, no `bin`)
- **Application**: Runnable entry point (has `main`, `bin`, or serves HTTP)
- **Tool**: CLI utility (has `bin` field or command-line interface)

## Output Template

Use the following structure exactly:

```markdown
# <Project Name> <Library|Application|Tool>

<1-2 sentence description of what the project does and its purpose>

## Quality Instructions

- **Typecheck**: `<command>` — <brief note if needed>
- **Lint**: `<command>` — <brief note if needed>
- **Test**: `<command>` — <brief note if needed>
- **Build**: `<command>` — <brief note if needed> (if applicable)

## Structure

- `src/` — <purpose>
- `lib/` — <purpose>
- `test/` — <purpose>
<additional key directories only>

## Technical Stack

- **Language**: <primary language and version if relevant>
- **Framework**: <main framework if applicable>
- **Key Libraries**: <2-4 most impactful dependencies>
- **Build Tool**: <bundler or build system>
```

## Generation Guidelines

### Be Minimal
- List only directories that matter for understanding the codebase
- Include only the most impactful 3-5 dependencies
- Omit obvious tooling (e.g., don't list `typescript` if already listing `tsc`)

### Be Specific
- Use actual command invocations, not generic descriptions
- Reference real paths from the analyzed project
- Match the project's actual tooling configuration

### Be Concise
- One-line descriptions for directories
- No redundant explanations
- Skip sections that don't apply (e.g., no "Framework" for a pure utility library)

## Example Output

```markdown
# @acme/data-utils Library

Lightweight data transformation utilities for parsing, validating, and formatting structured data.

## Quality Instructions

- **Typecheck**: `tsc --noEmit`
- **Lint**: `eslint --fix src/`
- **Test**: `vitest run`
- **Build**: `pnpm build`

## Structure

- `src/` — Source modules organized by domain (parsers, validators, formatters)
- `src/__tests__/` — Unit tests colocated with source

## Technical Stack

- **Language**: TypeScript 5.x
- **Key Libraries**: zod (validation), date-fns (date formatting)
- **Build Tool**: tsup
```