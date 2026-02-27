---
name: setup-cdk-templates
description: Use when creating CLAUDE.md files or .claude/ directories - detects project type, generates appropriate templates, and scaffolds Claude configuration with commands and hooks
---

# Setup CDK Templates

## Overview

Project context templates for Claude Code. Detects project type and generates appropriate CLAUDE.md files, `.claude/` directory structures, and custom commands.

## When to Use

- Creating CLAUDE.md for new or existing projects
- Setting up `.claude/` directory with commands/hooks
- User asks about Claude context optimization
- Part of `setup-claude-dev-kit` bundle

## Quick Reference

| Component | Location |
|-----------|----------|
| CLAUDE.md | Project root |
| Commands | `.claude/commands/` |
| Hooks | `.claude/hooks/` |
| Local settings | `.claude/settings.local.json` |

## Project Type Detection

Run these checks to detect project type:

```bash
detect_project_type() {
  if [ -f "package.json" ]; then
    if grep -q '"next"' package.json; then
      echo "web-app-next"
    elif grep -q '"react"' package.json; then
      echo "web-app-react"
    elif grep -q '"vue"' package.json; then
      echo "web-app-vue"
    elif grep -q '"express"' package.json || grep -q '"fastify"' package.json; then
      echo "api-node"
    elif grep -q '"bin"' package.json; then
      echo "cli-node"
    else
      echo "library-node"
    fi
  elif [ -f "requirements.txt" ] || [ -f "pyproject.toml" ]; then
    if [ -f "manage.py" ]; then
      echo "web-app-django"
    elif grep -qE "fastapi|flask" requirements.txt 2>/dev/null; then
      echo "api-python"
    elif [ -d "src" ] && [ -f "pyproject.toml" ]; then
      echo "library-python"
    else
      echo "cli-python"
    fi
  elif [ -f "go.mod" ]; then
    echo "go"
  elif [ -f "Cargo.toml" ]; then
    echo "rust"
  elif [ -f "pnpm-workspace.yaml" ] || [ -f "lerna.json" ] || [ -d "packages" ]; then
    echo "monorepo"
  else
    echo "generic"
  fi
}
```

## Installation Steps

### 1. Detect Project Type

```bash
PROJECT_TYPE=$(detect_project_type)
echo "Detected: $PROJECT_TYPE"
```

### 2. Create CLAUDE.md

Use the appropriate template below based on detected type.

### 3. Create .claude/ Directory

```bash
mkdir -p .claude/commands
mkdir -p .claude/hooks
```

### 4. Add Standard Commands

Create `.claude/commands/test.md`:
```markdown
Run all tests and report results. If tests fail, analyze the failure and suggest fixes.
```

Create `.claude/commands/review.md`:
```markdown
Review the recent changes for:
- Code quality and best practices
- Potential bugs or edge cases
- Performance implications
- Security concerns

Provide specific, actionable feedback.
```

### 5. Verify Setup

```bash
[ -f CLAUDE.md ] && echo "CLAUDE.md created"
[ -d .claude/commands ] && echo "Commands directory ready"
```

---

## CLAUDE.md Templates

### Web App (React/Next.js/Vue)

```markdown
# Project Context

## Overview
[Brief description of the application]

## Tech Stack
- Framework: [React/Next.js/Vue]
- Styling: [Tailwind/CSS Modules/styled-components]
- State: [Redux/Zustand/Pinia]
- API: [REST/GraphQL/tRPC]

## Directory Structure
- `src/components/` - React components
- `src/pages/` or `app/` - Routes
- `src/hooks/` - Custom hooks
- `src/lib/` - Utilities
- `src/styles/` - Global styles

## Commands
- `npm run dev` - Start dev server
- `npm run build` - Production build
- `npm run test` - Run tests
- `npm run lint` - Lint code

## Conventions
- Components: PascalCase (`UserProfile.tsx`)
- Hooks: camelCase with `use` prefix (`useAuth.ts`)
- Utilities: camelCase (`formatDate.ts`)
- Tests: `*.test.ts` or `*.spec.ts`

## Important Notes
- [List critical dependencies or constraints]
- [Note any legacy code or tech debt areas]
```

### API (REST/GraphQL)

```markdown
# Project Context

## Overview
[Brief description of the API]

## Tech Stack
- Runtime: [Node.js/Python/Go]
- Framework: [Express/FastAPI/Gin]
- Database: [PostgreSQL/MongoDB/Redis]
- Auth: [JWT/OAuth/API Keys]

## Directory Structure
- `src/routes/` or `src/api/` - Endpoints
- `src/models/` - Data models
- `src/services/` - Business logic
- `src/middleware/` - Request handling
- `src/utils/` - Helpers

## API Patterns
- RESTful endpoints: `/api/v1/resource`
- Error format: `{ error: string, code: string }`
- Auth header: `Authorization: Bearer <token>`

## Commands
- `npm run dev` - Start with hot reload
- `npm run test` - Run tests
- `npm run db:migrate` - Run migrations
- `npm run db:seed` - Seed data

## Environment
- `.env.example` - Template for env vars
- Never commit `.env` files

## Important Notes
- [Rate limiting configuration]
- [Required external services]
```

### CLI Application

```markdown
# Project Context

## Overview
[Brief description of the CLI tool]

## Tech Stack
- Language: [Node.js/Python/Go/Rust]
- Parser: [Commander/Click/Cobra/Clap]
- Config: [cosmiconfig/configparser]

## Directory Structure
- `src/commands/` - Command implementations
- `src/lib/` - Shared logic
- `src/utils/` - Helpers
- `bin/` - Entry points

## Command Structure
```
mytool <command> [options] [arguments]

Commands:
  init      Initialize new project
  build     Build the project
  deploy    Deploy to production
```

## Commands
- `npm run build` - Compile
- `npm run test` - Run tests
- `npm link` - Install globally for testing

## Conventions
- Exit codes: 0 = success, 1 = error
- Output: stdout for data, stderr for logs
- Config file: `.mytoolrc` or `mytool.config.js`

## Important Notes
- [Cross-platform considerations]
- [Required permissions or dependencies]
```

### Library/Package

```markdown
# Project Context

## Overview
[Brief description of the library]

## Tech Stack
- Language: [TypeScript/Python/Rust]
- Build: [tsup/rollup/setuptools/cargo]
- Testing: [Jest/Vitest/pytest/cargo test]

## Directory Structure
- `src/` - Source code
- `src/index.ts` - Public API exports
- `tests/` - Test files
- `docs/` - Documentation

## Public API
```typescript
// Main exports
export { functionA } from './moduleA';
export { ClassB } from './moduleB';
export type { TypeC } from './types';
```

## Commands
- `npm run build` - Build package
- `npm run test` - Run tests
- `npm run docs` - Generate docs
- `npm publish` - Publish to registry

## Conventions
- Semantic versioning (major.minor.patch)
- Changelog: CHANGELOG.md
- Breaking changes in major versions only

## Important Notes
- [Minimum supported versions]
- [Peer dependencies]
- [Bundle size considerations]
```

### Monorepo

```markdown
# Project Context

## Overview
[Brief description of the monorepo]

## Tech Stack
- Manager: [pnpm/npm/yarn workspaces]
- Build: [Turborepo/Nx/Lerna]
- Packages: [List main packages]

## Directory Structure
- `packages/` - Shared packages
- `apps/` - Applications
- `tooling/` - Build configuration

## Packages
| Package | Description |
|---------|-------------|
| `@org/core` | Core utilities |
| `@org/ui` | Component library |
| `@org/app` | Main application |

## Commands
- `pnpm install` - Install all deps
- `pnpm build` - Build all packages
- `pnpm dev` - Start dev servers
- `pnpm test` - Run all tests

## Conventions
- Package naming: `@org/package-name`
- Shared deps in root `package.json`
- Package-specific deps in package `package.json`

## Important Notes
- [Build order dependencies]
- [Shared configuration locations]
```

### Generic/Unknown

```markdown
# Project Context

## Overview
[Brief description of the project]

## Tech Stack
- [List main technologies]

## Directory Structure
- [Describe key directories]

## Commands
- [List common commands]

## Conventions
- [List naming and style conventions]

## Important Notes
- [List critical information]
```

---

## .claude/ Directory Templates

### Standard Commands

`.claude/commands/debug.md`:
```markdown
Help debug the current issue:
1. Identify the error or unexpected behavior
2. Trace the root cause through the code
3. Suggest fixes with explanations
4. Verify the fix doesn't introduce regressions
```

`.claude/commands/refactor.md`:
```markdown
Refactor the specified code:
1. Analyze current implementation
2. Identify improvement opportunities
3. Apply changes incrementally
4. Ensure tests still pass
5. Document significant changes
```

`.claude/commands/document.md`:
```markdown
Generate or update documentation:
1. Analyze the code structure
2. Write clear, concise docs
3. Include examples where helpful
4. Follow existing documentation style
```

### Standard Hooks

`.claude/hooks/pre-commit.sh`:
```bash
#!/bin/bash
# Run before Claude commits

# Lint check
npm run lint --silent || exit 1

# Type check
npm run typecheck --silent || exit 1

echo "Pre-commit checks passed"
```

## Adaptation Mode

When existing CLAUDE.md or .claude/ detected:

1. **Backup existing:**
```bash
mkdir -p ~/.claude-dev-kit/backups/$(date +%Y-%m-%d)
cp CLAUDE.md ~/.claude-dev-kit/backups/$(date +%Y-%m-%d)/CLAUDE.md.bak 2>/dev/null
cp -r .claude ~/.claude-dev-kit/backups/$(date +%Y-%m-%d)/.claude.bak 2>/dev/null
```

2. **Check content:**
- Existing CLAUDE.md → Offer to merge or enhance
- Custom commands → Preserve, add CDK commands alongside
- Custom hooks → Don't overwrite

3. **Merge approach:**
- Add missing sections to CLAUDE.md
- Create new commands in `.claude/commands/` without overwriting
- Suggest improvements to existing content

## Common Issues

| Issue | Fix |
|-------|-----|
| Wrong project type detected | Manually specify type or edit template |
| Commands not recognized | Ensure `.claude/commands/` exists and files are `.md` |
| Hooks not running | Check file permissions (`chmod +x`) |
| CLAUDE.md too long | Focus on essential context only |

## Tips

- Keep CLAUDE.md under 500 lines for optimal context usage
- Update CLAUDE.md when major changes occur
- Use commands for repetitive tasks
- Custom commands can reference other commands
