---
name: repo-understanding
description: Build a complete mental model of a repository's structure, commands, dependencies, and conventions. Invoke as @repo-understanding.
---

# Skill: Repository Understanding

## Purpose
Before making changes, understand how the repo is organized, what commands it provides, how tests run, and what conventions the team follows.

## When to Use This Skill
- Starting work on a new repository
- Onboarding to a project
- Planning a large refactoring or feature
- Before making cross-cutting changes

## Steps

### 1) Understand the directory structure
Read the repo root and understand:
- `/crates`, `/src`, `/packages`: source code directories
- `/tests`, `/test`: test files
- `/docs`: documentation
- `/scripts`: build/CI scripts
- `/infra`, `/deploy`: infrastructure/deployment configs
- `/assets`, `/static`: non-code files

Example output:
```
markenz/
  crates/           # Rust libraries (physics, world, rng)
  apps/             # Rust applications (engine)
  tests/            # Integration tests
  docs/             # Documentation
  observability/    # Logging schemas and conventions
  runbooks/         # Incident response playbooks
  scripts/ci/       # CI helper scripts
```

### 2) Read the AGENTS.md file
This is the authoritative source for:
- Canonical commands (lint, test, format, typecheck, build)
- Directory-scoped rules and conventions
- How to run the project locally
- How to contribute

Example:
```markdown
# Canonical Commands
- cargo build           # Build all crates
- cargo test --all      # Run all tests
- cargo clippy --all    # Linter
```

### 3) Identify main entrypoints
For applications:
- Which files are the main entry points? (main.rs, index.js, server.py)
- How does the app start? (CLI args, env vars, configs)
- What are the key services or modules?

For libraries:
- What is the public API? (exported functions, types, classes)
- What are the main invariants and constraints?

### 4) Understand the dependency graph
- What external dependencies does the project use?
- Which modules depend on which?
- Are there circular dependencies or tight coupling?

Example (Rust):
```bash
cargo tree
```

### 5) Review the test structure
- Where are tests located? (same file, separate directory, docs)
- How do you run tests? (`npm test`, `pytest`, `cargo test`)
- Are there separate test suites? (unit, integration, e2e)
- What's the coverage target?

### 6) Understand the build/CI process
- How does the code get built? (npm, cargo, Python setuptools)
- What CI system is used? (.github/workflows, GitLab CI, etc.)
- What are the quality gates? (linters, type checkers, tests)
- How are artifacts packaged and released?

Example:
```bash
cat .github/workflows/ci.yml | grep "run:" | head -10
```

### 7) Identify the tech stack
- Language(s): JavaScript, Rust, Python, etc.
- Frameworks: React, Express, Django, Actix, etc.
- Databases: PostgreSQL, MongoDB, Redis, etc.
- Testing: Jest, pytest, cargo test, etc.
- CI: GitHub Actions, GitLab CI, Jenkins, etc.

### 8) Review the GLOBAL_RULES or standards
Read the governance files:
- AGENTS.md (repo-level rules)
- GLOBAL_RULES.md (shared across team)
- .windsurf/ (Windsurf-specific conventions)

Understand:
- Code style guidelines
- Security requirements (secrets, validation, redaction)
- Observability requirements (logging, metrics, tracing)
- Test coverage targets
- Documentation standards

### 9) Capture key mental models
Document these in your head:
- Data flow: How does data enter, flow through, and exit the system?
- Error paths: How are failures handled and logged?
- Concurrency model: Is it single-threaded, multi-threaded, async?
- Deployment: How does code get to production?

### 10) Ask clarifying questions
If anything is unclear:
- Check the README and docs
- Look for comments in key files
- Check the git log for recent changes
- Ask the team or open issues

## Quality Checklist

- [ ] Directory structure understood
- [ ] AGENTS.md read and key commands identified
- [ ] Main entrypoints identified
- [ ] Dependency graph understood
- [ ] Test structure known
- [ ] Build/CI process clear
- [ ] Tech stack documented
- [ ] Governance rules reviewed
- [ ] Data flow understood
- [ ] Can run tests locally

## Verification Commands

```bash
# Understand structure
ls -la
cat AGENTS.md
cat README.md

# Identify commands
grep -r "\"scripts\":" package.json | head -20
cat Justfile | grep "^[a-z]"
grep "^##" AGENTS.md | head -20

# Run a basic test
npm test
cargo test --lib
python -m pytest

# Check dependencies
cargo tree | head -50
npm list | head -50

# Understand CI
cat .github/workflows/ci.yml | head -30
```

## KAIZA-AUDIT Compliance

When using this skill as part of another task, your KAIZA-AUDIT block should include:
- **Scope**: Modules/areas touched
- **Key Decisions**: Explain how your changes respect the repo's conventions and tech stack
- **Verification**: Confirm commands from AGENTS.md pass (lint, tests, etc.)
