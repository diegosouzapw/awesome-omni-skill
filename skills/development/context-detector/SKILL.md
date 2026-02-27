---
name: context-detector
description: Detect project language, framework, and existing conventions. Use when creating workflows to adapt generated artifacts to match project patterns.
---

# Context Detector

## When to Use This Skill

- Creating new workflows that should match project conventions
- Detecting project language and framework
- Analyzing existing artifacts for naming patterns
- Adapting tool suggestions to project stack

## Language Detection

Check these files to detect project language:

| File | Language/Framework |
|------|-------------------|
| `package.json` | Node.js/JavaScript/TypeScript |
| `tsconfig.json` | TypeScript |
| `Cargo.toml` | Rust |
| `go.mod` | Go |
| `pyproject.toml` | Python (modern) |
| `requirements.txt` | Python |
| `Gemfile` | Ruby |
| `pom.xml` | Java (Maven) |
| `build.gradle` | Java/Kotlin (Gradle) |
| `composer.json` | PHP |
| `mix.exs` | Elixir |
| `pubspec.yaml` | Dart/Flutter |
| `*.csproj` | C#/.NET |

## Framework Detection

### JavaScript/TypeScript Frameworks

Check `package.json` dependencies:

| Dependency | Framework |
|------------|-----------|
| `react` | React |
| `next` | Next.js |
| `vue` | Vue.js |
| `nuxt` | Nuxt |
| `@angular/core` | Angular |
| `svelte` | Svelte |
| `express` | Express.js |
| `fastify` | Fastify |
| `nestjs` | NestJS |

### Python Frameworks

Check `pyproject.toml` or `requirements.txt`:

| Package | Framework |
|---------|-----------|
| `django` | Django |
| `flask` | Flask |
| `fastapi` | FastAPI |
| `pytest` | Testing with pytest |

## Tool Suggestions by Stack

### Node.js/TypeScript
```json
{
  "formatter": "prettier",
  "linter": "eslint",
  "test": "jest or vitest",
  "typecheck": "tsc --noEmit"
}
```

### Python
```json
{
  "formatter": "black",
  "linter": "ruff or flake8",
  "imports": "isort",
  "test": "pytest",
  "typecheck": "mypy"
}
```

### Rust
```json
{
  "formatter": "rustfmt",
  "linter": "clippy",
  "test": "cargo test"
}
```

### Go
```json
{
  "formatter": "gofmt",
  "linter": "golangci-lint",
  "test": "go test"
}
```

## Existing Artifact Analysis

### Scan for Existing Patterns

1. **Commands**: Read files in `.claude/commands/`
   - Extract naming conventions (kebab-case, namespaces)
   - Note frontmatter patterns used
   - Identify tool access patterns

2. **Skills**: Read `.claude/skills/*/SKILL.md`
   - Extract description style
   - Note trigger patterns
   - Identify structure conventions

3. **Agents**: Read `.claude/agents/*.md`
   - Extract naming patterns
   - Note tool scoping patterns
   - Identify output format conventions

4. **Hooks**: Read `.claude/settings.json`
   - Identify existing hook patterns
   - Note script locations
   - Check for conflicts

### Convention Extraction Checklist

- [ ] Naming: kebab-case, snake_case, or camelCase?
- [ ] Description length: short (<100 chars) or detailed?
- [ ] Tool scoping: minimal or generous?
- [ ] Output format: markdown, JSON, or plain text?
- [ ] Error handling: fail fast or graceful degradation?

## Adapting to Conventions

### Match Existing Command Style

If existing commands use:
```markdown
---
allowed-tools: Read, Write
description: Short description
---
```

New commands should match that structure.

### Match Hook Patterns

If project uses hook scripts in `.claude/hooks/`:
- Create new scripts there, not inline
- Match the shebang style (`#!/bin/bash` vs `#!/usr/bin/env bash`)
- Match error handling patterns

### Match Skill Structure

If existing skills have:
```markdown
## When to Use
## Process
## Output Format
```

New skills should use the same sections.

## Detection Process

### Step 1: Identify Language
```bash
# Check for config files
ls -la package.json Cargo.toml go.mod pyproject.toml 2>/dev/null
```

### Step 2: Identify Framework
```bash
# For Node.js
cat package.json | jq '.dependencies | keys'
```

### Step 3: Scan Existing Artifacts
```bash
# Count existing artifacts
ls -la .claude/commands/**/*.md 2>/dev/null | wc -l
ls -la .claude/skills/*/SKILL.md 2>/dev/null | wc -l
ls -la .claude/agents/*.md 2>/dev/null | wc -l
```

### Step 4: Extract Conventions
Read 2-3 examples of each type and note patterns.

## Output Format

When reporting detected context:

```markdown
## Project Context

### Language/Framework
- Primary: TypeScript
- Framework: Next.js
- Test Framework: Jest

### Detected Tools
- Formatter: prettier (in package.json)
- Linter: eslint (in package.json)
- Build: next build

### Existing Artifacts
- Commands: 3 (kebab-case naming)
- Skills: 2 (detailed descriptions)
- Agents: 1 (minimal tool scope)
- Hooks: 2 (script-based in .claude/hooks/)

### Conventions
- Descriptions: Under 100 chars
- Tools: Minimal scoping
- Output: Markdown format
```

## Anti-Patterns

### ❌ Assuming stack without checking
Always verify by reading config files.

### ❌ Ignoring existing patterns
Match existing conventions even if not optimal.

### ❌ Over-detecting
Focus on what matters for artifact generation.

### ❌ Hard-coding tool choices
Adapt to what the project actually uses.
