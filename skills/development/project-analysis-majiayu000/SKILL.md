---
name: project-analysis
description: Analyze and understand codebase structure, dependencies, and patterns. Use when onboarding to a new project or auditing existing code.
---

# Project Analysis Skill

## Purpose
Quickly understand codebases by systematically analyzing structure, dependencies, patterns, and conventions.

## Analysis Framework

### Phase 1: High-Level Overview (5 minutes)

```bash
# 1. Check project type and structure
ls -la
cat README.md
cat package.json | head -50  # or pyproject.toml, Cargo.toml, etc.

# 2. Understand directory structure
tree -L 2 -I 'node_modules|.git|dist|build'

# 3. Check git history for activity
git log --oneline -20
git shortlog -sn --since="6 months ago"
```

### Phase 2: Dependency Analysis (5 minutes)

```bash
# Node.js
cat package.json | jq '.dependencies, .devDependencies'

# Python
cat requirements.txt
cat pyproject.toml

# Go
cat go.mod

# Rust
cat Cargo.toml
```

**Key Questions:**
- What frameworks are used?
- What database/cache?
- Any outdated/vulnerable deps?

### Phase 3: Architecture Mapping (10 minutes)

```markdown
## Architecture Map

### Entry Points
- `src/index.ts` - Main application entry
- `src/api/` - REST API endpoints
- `src/workers/` - Background job processors

### Core Modules
- `src/services/` - Business logic
- `src/models/` - Data models/entities
- `src/repositories/` - Data access layer

### Infrastructure
- `src/config/` - Configuration management
- `src/middleware/` - Express middleware
- `src/utils/` - Shared utilities

### Data Flow
Client → API Routes → Controllers → Services → Repositories → Database
```

### Phase 4: Pattern Recognition

#### Common Patterns to Identify

| Pattern | Files to Check | Indicators |
|---------|---------------|------------|
| MVC | controllers/, models/, views/ | Separate concerns |
| Clean Architecture | domain/, application/, infrastructure/ | Dependency inversion |
| Repository | repositories/, *Repository.ts | Data access abstraction |
| Factory | factories/, *Factory.ts | Object creation |
| Singleton | getInstance() | Global state |
| Observer | EventEmitter, on(), emit() | Event-driven |

### Phase 5: Code Quality Assessment

```bash
# Check for linting/formatting
ls .eslintrc* .prettierrc* .editorconfig

# Check for tests
ls -la tests/ __tests__/ *.test.* *.spec.*
npm test -- --coverage 2>/dev/null

# Check for CI/CD
ls .github/workflows/ .gitlab-ci.yml Jenkinsfile

# Check for documentation
ls docs/ *.md CONTRIBUTING* CHANGELOG*
```

## Analysis Output Template

```markdown
# Project Analysis: [Project Name]

## Summary
- **Type:** [Web API / CLI / Library / Monorepo]
- **Language:** [TypeScript / Python / Go / Rust]
- **Framework:** [Express / FastAPI / Gin / Actix]
- **Database:** [PostgreSQL / MongoDB / Redis]
- **Last Activity:** [Date]
- **Contributors:** [Number]

## Directory Structure
```
project/
├── src/                 # Source code
│   ├── api/             # HTTP endpoints
│   ├── services/        # Business logic
│   └── models/          # Data models
├── tests/               # Test files
├── docs/                # Documentation
└── scripts/             # Build/deploy scripts
```

## Key Dependencies
| Package | Version | Purpose |
|---------|---------|---------|
| express | 4.18.2 | HTTP server |
| prisma | 5.7.0 | ORM |
| zod | 3.22.0 | Validation |

## Architecture Style
[Description of architectural approach]

## Data Flow
[Diagram or description of how data flows through the system]

## Code Quality Indicators
- [ ] Tests: X% coverage
- [ ] Linting: [ESLint configured]
- [ ] Types: [Strict TypeScript]
- [ ] CI/CD: [GitHub Actions]
- [ ] Docs: [README, API docs]

## Notable Patterns
1. [Pattern 1 with location]
2. [Pattern 2 with location]

## Potential Issues
1. [Issue 1]
2. [Issue 2]

## Recommendations
1. [Recommendation 1]
2. [Recommendation 2]
```

## Framework-Specific Analysis

### Node.js / TypeScript

```bash
# Check TypeScript config
cat tsconfig.json

# Check for monorepo
ls packages/ apps/ pnpm-workspace.yaml lerna.json

# Check build output
cat package.json | jq '.scripts'
```

### Python

```bash
# Check Python version
cat .python-version
cat pyproject.toml | grep python

# Check dependency management
ls requirements.txt pyproject.toml poetry.lock Pipfile

# Check for type hints
grep -r "def.*:.*->" src/ | head -5
```

### Go

```bash
# Check Go version
cat go.mod | head -3

# Check project layout
ls cmd/ internal/ pkg/ api/

# Check for common patterns
grep -r "interface {" --include="*.go" | head -5
```

### Rust

```bash
# Check edition and dependencies
cat Cargo.toml

# Check module structure
ls src/*.rs

# Check for async runtime
grep -E "tokio|async-std" Cargo.toml
```

## Quick Health Check

```markdown
## Project Health Score

### Documentation (X/5)
- [ ] README with setup instructions
- [ ] API documentation
- [ ] Architecture docs
- [ ] Contributing guide
- [ ] Changelog

### Testing (X/5)
- [ ] Test files exist
- [ ] Coverage > 60%
- [ ] Integration tests
- [ ] CI runs tests
- [ ] Tests are maintained

### Code Quality (X/5)
- [ ] Linter configured
- [ ] Formatter configured
- [ ] Type safety (TypeScript/types)
- [ ] No obvious code smells
- [ ] Dependencies up to date

### DevOps (X/5)
- [ ] CI/CD pipeline
- [ ] Environment configuration
- [ ] Deployment docs
- [ ] Monitoring setup
- [ ] Security scanning

### Overall: X/20
```

## Onboarding Questions

When analyzing a project for onboarding, answer:

1. **How do I run it locally?**
   - Setup commands
   - Environment variables needed
   - Database/service dependencies

2. **How is it structured?**
   - Where does the code live?
   - What are the main components?
   - How do they interact?

3. **How do I make changes?**
   - Branch strategy
   - PR process
   - Testing requirements

4. **How is it deployed?**
   - CI/CD pipeline
   - Environments (dev/staging/prod)
   - Release process

5. **Where do I get help?**
   - Documentation location
   - Team contacts
   - Common troubleshooting

## Red Flags to Watch For

| Red Flag | Impact | Recommendation |
|----------|--------|----------------|
| No tests | High risk changes | Add tests before modifying |
| No types | Runtime errors | Enable strict TypeScript |
| Outdated deps | Security risk | Run `npm audit` |
| No CI/CD | Manual deploys | Set up GitHub Actions |
| No .gitignore | Secrets in repo | Add immediately |
| Giant files | Hard to maintain | Plan refactoring |
| Deep nesting | Complex logic | Simplify, extract |
| Copy-paste code | Maintenance burden | DRY refactoring |
