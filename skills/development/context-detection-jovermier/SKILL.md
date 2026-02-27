---
name: context-detection
description: Automatically detect project tech stack, frameworks, and development context
---

# Project Context Detection

Automatically analyzes the current project to detect technologies, frameworks, and development patterns.

## When to Use

This skill is invoked when:
- Running `/auto-skills` command

## Detection Methods

### 1. File System Analysis

Check for configuration files and lock files:

| Technology | Indicators |
|------------|------------|
| **Node.js** | `package.json`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml` |
| **Go** | `go.mod`, `go.sum`, `*.go` files |
| **Python** | `requirements.txt`, `pyproject.toml`, `Pipfile`, `poetry.lock` |
| **Ruby** | `Gemfile`, `Gemfile.lock` |
| **Rust** | `Cargo.toml`, `Cargo.lock` |
| **Java** | `pom.xml`, `build.gradle` |
| **TypeScript** | `tsconfig.json` |

### 2. Framework Detection

| Framework | Indicators |
|-----------|------------|
| **Next.js** | `next.config.js`, `app/` directory with `page.tsx` |
| **React** | `package.json` contains `react`, JSX files |
| **Vue** | `package.json` contains `vue`, `.vue` files |
| **GraphQL** | `.graphql` files, `schema.graphql`, Apollo/GraphQL in deps |
| **Express** | `express` in dependencies |
| **FastAPI** | `fastapi` in dependencies |
| **Django** | `django` in dependencies, `manage.py` |
| **Rails** | `rails` in dependencies, `config/routes.rb` |

### 3. Testing Frameworks

| Framework | Indicators |
|-----------|------------|
| **Jest** | `jest.config.js`, `*.test.js`, `*.spec.js` |
| **Playwright** | `playwright.config.js`, `*.spec.ts` |
| **Pytest** | `pytest.ini`, `conftest.py`, `test_*.py` |
| **Go testing** | `*_test.go` files |

### 4. Code Analysis

Analyze file extensions and imports:
- Count file types (`.go`, `.tsx`, `.py`, etc.)
- Analyze import statements for frameworks
- Check for API routes, database schemas

## Output Format

Return a structured context object:

```json
{
  "languages": ["go", "javascript"],
  "frameworks": ["nextjs", "graphql"],
  "testing": ["playwright", "go-testing"],
  "packageManager": "pnpm",
  "hasApi": true,
  "hasDatabase": true,
  "projectType": "fullstack"
}
```

## Example Workflow

When detecting context for a project:

1. Scan root directory for config files
2. Analyze `package.json`, `go.mod`, or equivalent
3. Count file types in `src/` or main directories
4. Check for test files and frameworks
5. Return structured context for auto-installation decisions

## Integration

This skill feeds into:
- **scan**: Determines which SkillsMP skills to fetch
