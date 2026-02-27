---
name: project-scaffolder
description: Guide for setting up Claude Code infrastructure in new or existing projects
version: 1.0.0
triggers:
  - init claude
  - setup claude code
  - scaffold project
  - initialize infrastructure
  - new project
---

# Project Scaffolder

## Overview

This skill guides the process of setting up Claude Code infrastructure in a project. It handles stack detection, template selection, and file generation. Use this when setting up Claude Code for a new project or adding it to an existing codebase.

## Quick Reference

| Stack | Detection Files | Primary Use |
|-------|-----------------|-------------|
| TypeScript | `tsconfig.json`, `*.ts`, `*.tsx` | React/Node fullstack |
| Python | `pyproject.toml`, `requirements.txt` | FastAPI/Django backend |
| Rust | `Cargo.toml`, `*.rs` | Axum/Actix backend |

## Stack Detection

### TypeScript Detection

Look for:
```
tsconfig.json
package.json (with "typescript" in dependencies/devDependencies)
*.ts or *.tsx files in src/
```

Indicators of specific frameworks:
- `next.config.js` → Next.js
- `vite.config.ts` → Vite
- `express` in package.json → Express backend
- `prisma/` directory → Prisma ORM

### Python Detection

Look for:
```
pyproject.toml
requirements.txt
setup.py
*.py files
```

Indicators of specific frameworks:
- `fastapi` in dependencies → FastAPI
- `django` in dependencies → Django
- `flask` in dependencies → Flask
- `alembic/` directory → Alembic migrations

### Rust Detection

Look for:
```
Cargo.toml
Cargo.lock
src/main.rs or src/lib.rs
```

Indicators of specific frameworks:
- `axum` in Cargo.toml → Axum
- `actix-web` in Cargo.toml → Actix-web
- `diesel` in Cargo.toml → Diesel ORM
- `sqlx` in Cargo.toml → SQLx

## Skill Selection Matrix

Based on detected stack, select appropriate skills:

### Universal Skills (All Stacks)

| Skill | Purpose |
|-------|---------|
| `git-workflow` | Git conventions, branching |
| `dev-docs-workflow` | Context management |
| `code-review-workflow` | Review checklists |
| `skill-developer` | Creating new skills |

### Domain Skills (Cross-Cutting)

| Skill | Purpose | When to Include |
|-------|---------|-----------------|
| `api-validation` | Request validation | Any API project |
| `auth-guidelines` | Security patterns | Auth features |
| `error-handling` | Error patterns | All projects |
| `error-tracking` | Sentry integration | Production apps |

### TypeScript-Specific Skills

| Skill | Purpose |
|-------|---------|
| `backend-dev-guidelines` | Express/Node patterns |
| `frontend-dev-guidelines` | React/MUI patterns |
| `database-guidelines` | Prisma patterns |
| `testing-guidelines` | Jest/Vitest patterns |
| `state-management` | TanStack/Zustand |

### Python-Specific Skills

| Skill | Purpose |
|-------|---------|
| `python-backend-guidelines` | FastAPI/Django patterns |
| `python-database-guidelines` | SQLAlchemy patterns |
| `python-testing-guidelines` | pytest patterns |

### Rust-Specific Skills

| Skill | Purpose |
|-------|---------|
| `rust-backend-guidelines` | Axum/Actix patterns |
| `rust-database-guidelines` | Diesel/SQLx patterns |
| `rust-testing-guidelines` | cargo test patterns |
| `rust-error-handling` | Result/Option patterns |
| `rust-ownership` | Borrow checker patterns |

## CLAUDE.md Generation

### Template Variables

| Variable | Source | Example |
|----------|--------|---------|
| `{{PROJECT_NAME}}` | Directory name | "my-app" |
| `{{FRAMEWORK_VERSION}}` | Constant | "1.0.0" |
| `{{STACK_NAME}}` | Manifest displayName | "TypeScript/React/Node" |
| `{{STACK_VERSION}}` | Manifest version | "1.0.0" |
| `{{TIMESTAMP}}` | Current date | "2025-12-07" |

### Required Sections

Every CLAUDE.md should include:

1. **Quick Commands** - Build, test, lint, format
2. **Project Structure** - Directory layout
3. **Tech Stack** - Technologies used
4. **Dev Docs Workflow** - How to use /dev-docs
5. **Available Skills** - Listed by category
6. **Available Agents** - With usage examples
7. **Coding Conventions** - Project-specific rules

## Directory Structure

After initialization:

```
project/
├── CLAUDE.md                 # Generated from template
├── .claude/
│   ├── settings.json         # Hook configurations
│   ├── hooks/               # Automation hooks
│   ├── skills/              # Skill files
│   ├── commands/            # Slash commands
│   ├── agents/              # Specialized agents
│   ├── templates/           # Scaffolding templates
│   └── cache/               # Session cache
└── dev/
    └── active/              # Dev docs workspace
```

## Customization Points

After scaffolding, users should customize:

### In CLAUDE.md

- Project structure section
- Coding conventions
- Quick commands (if different from defaults)

### In skill-rules.json

- Add project-specific keywords
- Adjust enforcement levels
- Add custom file triggers

### Create Custom Skills

For project-specific patterns not covered by standard skills.

## Workflow

### New Project

1. Create project directory
2. Initialize stack (npm init, cargo init, etc.)
3. Run `/init-claude-code`
4. Customize CLAUDE.md
5. Start developing with skills

### Existing Project

1. Navigate to project root
2. Run `/init-claude-code`
3. Review detected stack (confirm or override)
4. Review generated CLAUDE.md
5. Merge with existing documentation if needed

## Troubleshooting

### Multiple Stacks Detected

If project has multiple config files:
- Ask user which is primary
- Consider monorepo setup
- May need custom configuration

### No Stack Detected

If no config files found:
- Ask user to specify stack
- Or create minimal config first

### Missing Skills

Some skills may not exist yet:
- Note which are missing
- Continue with available skills
- Create missing skills as needed

## Resources

| Topic | Link |
|-------|------|
| Stack Detection | [mdc:resources/detection-patterns.md] |
| Skill Matrix | [mdc:resources/skill-matrix.md] |
| Template Guide | [mdc:resources/template-guide.md] |
