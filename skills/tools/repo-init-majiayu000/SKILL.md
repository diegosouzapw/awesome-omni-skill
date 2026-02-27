---
name: repo-init
description: Initialize a new repository with standard scaffolding - git, gitignore, CLAUDE.md, justfile, mise, and beads. Use when starting a new project or setting up an existing repo for Claude Code workflows.
---

# Repository Initialization

Scaffold a new or existing repository with standard project infrastructure.

**Related skills:**
- **just-pro** - Build system patterns and templates
- **mise** - Tool version management
- **go-pro**, **rust-pro**, **typescript-pro** - Language-specific setup (run after repo-init)

## What This Skill Does

Creates universal scaffolding that works across all project types:

| File | Purpose |
|------|---------|
| `.gitignore` | Standard ignores + language-specific patterns |
| `CLAUDE.md` | Project conventions for Claude Code |
| `justfile` | Build system skeleton |
| `.mise.toml` | Tool version pinning (empty, ready for `mise use`) |
| `.beads/` | Issue tracking database |
| `.envrc.example` | Environment variable template |

## Flow

```
1. Gather context (language, project type)
2. Create scaffolding
3. Initialize beads
4. Point to language skill for next steps
```

---

## Step 1: Gather Context

Before scaffolding, clarify:

1. **Project language(s)**: Go, Rust, TypeScript, Python, or multi-language?
2. **Project type**: Library, CLI, web app, API, monorepo?
3. **Existing files**: Is this a fresh repo or adding to existing code?

Use AskUserQuestion if unclear from context.

---

## Step 2: Git Setup

```bash
# Initialize git if needed
git init

# Create .gitignore with common patterns
```

### .gitignore Templates

Each language skill provides a comprehensive `.gitignore` in its `references/` directory. These include common patterns (`.env`, `.envrc`, `.DS_Store`, `Thumbs.db`, IDE files) plus language-specific ignores.

**Copy from the appropriate language skill:**

| Language | Source |
|----------|--------|
| Go | `go-pro/references/gitignore` → `.gitignore` |
| Rust | `rust-pro/references/gitignore` → `.gitignore` |
| TypeScript | `typescript-pro/references/gitignore` → `.gitignore` |
| Python | `python-pro/references/gitignore` → `.gitignore` |

**For multi-language repos:** Start with the primary language's gitignore, then merge patterns from others as needed.

**Minimal fallback** (if language skill unavailable):

```gitignore
# Environment
.env
.env.local
.env.*.local
.envrc

# OS
.DS_Store
Thumbs.db

# IDE
.idea/
.vscode/

# Build (customize per language)
dist/
build/
target/
node_modules/
__pycache__/
```

---

## Step 3: CLAUDE.md

Create project conventions file for Claude Code:

```markdown
# Project Name - Claude Instructions

## Overview

Brief description of what this project does.

## Development

```bash
just setup    # First-time setup
just check    # Run all quality gates
```

## Conventions

- [Add project-specific patterns here]
- [Coding standards, naming conventions, etc.]

## Architecture

- [Key directories and their purposes]
- [Important abstractions]
```

Keep it minimal initially. Add conventions as they emerge.

---

## Step 4: Justfile Skeleton

```just
# Project Build System
# Usage: just --list

default:
    @just --list

# First-time setup
setup:
    mise trust
    mise install
    @echo "Ready. Run 'just check' to verify."

# Quality gates - add language-specific checks
check:
    @echo "Add fmt, lint, test recipes"

# Remove build artifacts
clean:
    @echo "Add clean commands"
```

For language-specific recipes, see:
- **just-pro** skill references (`package-go.just`, `package-rust.just`, etc.)

---

## Step 5: Mise Configuration

Create empty `.mise.toml` ready for tool pinning:

```toml
[tools]
# Add tools with: mise use <tool>@<version>
# Examples:
# node = "22"
# go = "1.25"
# rust = "1.83"
# just = "latest"
```

---

## Step 6: Beads Initialization

```bash
bd init -q
```

This creates `.beads/` directory with issue tracking database.

For teams, consider:
- `bd init --team` - Interactive team workflow setup
- `bd init --stealth` - Personal use without affecting collaborators

---

## Step 7: Environment Template

Create `.envrc.example` (committed) as template for `.envrc` (gitignored):

```bash
# Copy to .envrc and fill in values
# cp .envrc.example .envrc && direnv allow

# Mise integration
if command -v mise &> /dev/null; then
  eval "$(mise hook-env -s bash)"
fi

# Project-specific environment
# export DATABASE_URL="postgres://localhost/myapp"
# export API_KEY=""
```

---

## Step 8: Next Steps

After scaffolding, point user to language-specific setup:

| Language | Next Step |
|----------|-----------|
| Go | Invoke **go-pro** skill, run `go mod init`, copy `.golangci.yml` |
| Rust | Invoke **rust-pro** skill, run `cargo init`, copy `clippy.toml` |
| TypeScript | Invoke **typescript-pro** skill, run `npm init` |
| Multi-language | Follow each lang skill for respective packages |

---

## Quick Reference

```bash
# Full init sequence
git init
# Create .gitignore, CLAUDE.md, justfile, .mise.toml, .envrc.example
bd init -q
mise use just@latest
# Then follow language skill for specifics
```

## Monorepo Variant

For monorepos, the root gets:
- Root `justfile` with module imports (see just-pro monorepo patterns)
- Root `.mise.toml` with shared tooling
- Single `.beads/` at root

Each package gets:
- Package-local `justfile`
- Language-specific configs (Cargo.toml, package.json, etc.)
