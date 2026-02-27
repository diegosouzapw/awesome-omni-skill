---
name: repo-docs-generator
description: "Generate comprehensive AGENTS.md, README.md, and CLAUDE.md documentation for any repository. Deep-dives into codebase structure, identifies technologies, creates ASCII architecture diagrams, and respects existing documentation content."
allowed-tools: Bash Read Edit Write Grep Glob Task
argument-hint: "[path] (e.g., /path/to/repo or . for current directory)"
---

# Repository Documentation Generator Skill

Generates comprehensive, agent-friendly documentation (AGENTS.md, README.md, CLAUDE.md) for any software repository by analyzing its structure, technologies, and patterns.

## When to Use This Skill

- Setting up documentation for a new repository
- Enhancing existing documentation with architecture diagrams
- Creating standardized AGENTS.md files across an organization
- Onboarding AI agents to understand a codebase
- Documenting legacy codebases that lack proper documentation

## Philosophy

**Respect existing content.** Never blindly replace documentation that humans have written. Always:
1. Read existing AGENTS.md, README.md, and CLAUDE.md files first
2. Identify valuable content that must be preserved
3. Enhance rather than replace when possible
4. Ask for confirmation before making destructive changes

## Core Workflow

### Phase 1: Discovery

Thoroughly explore the repository before writing anything:

```bash
# 1. Understand the repo structure
find . -maxdepth 3 -type f -name "*.md" 2>/dev/null | head -20
ls -la
cat README.md 2>/dev/null || echo "No README.md"
cat AGENTS.md 2>/dev/null || echo "No AGENTS.md"
cat CLAUDE.md 2>/dev/null || echo "No CLAUDE.md"

# 2. Identify the tech stack
ls package.json pyproject.toml requirements.txt Cargo.toml go.mod build.gradle pom.xml Gemfile 2>/dev/null
cat package.json 2>/dev/null | head -50
cat pyproject.toml 2>/dev/null | head -50

# 3. Find main entry points
find . -maxdepth 2 -name "main.*" -o -name "index.*" -o -name "app.*" 2>/dev/null
ls src/ app/ lib/ 2>/dev/null

# 4. Identify the build/deploy system
ls Dockerfile docker-compose.yml .github/workflows .circleci serverless.yml 2>/dev/null

# 5. Check for existing documentation
find . -maxdepth 2 -name "*.md" -type f 2>/dev/null

# 6. Identify quality gates (pre-commit, linters, formatters)
ls .pre-commit-config.yaml .pre-commit-config.yml 2>/dev/null
rg -n "djlint|ruff|mypy|pyright|ty|eslint|prettier|black|isort" .pre-commit-config.yaml .pre-commit-config.yml pyproject.toml package.json 2>/dev/null | head -50
```

### Phase 2: Analysis

Based on discovery, identify:

1. **Repository Type**: Web app, API, CLI tool, library, infrastructure, scripts collection, etc.
2. **Primary Language(s)**: Python, JavaScript, Java, Go, Rust, etc.
3. **Framework(s)**: Django, React, Express, Spring, etc.
4. **Architecture Pattern**: Monolith, microservices, serverless, etc.
5. **Key Components**: What are the main modules/packages/services?
6. **Data Storage**: PostgreSQL, MongoDB, Redis, S3, etc.
7. **CI/CD**: GitHub Actions, CircleCI, Jenkins, etc.
8. **Deployment**: Docker, Kubernetes, Lambda, Heroku, etc.

### Phase 2.5: Quality Gates (High ROI)

If the repo uses pre-commit and/or strict linters, treat them as the highest
ROI documentation target. In AGENTS.md, explicitly document:

- How to run the gates (preferred wrappers, scoped vs all-files behavior)
- What the gates enforce (template rules, type checks, commit-msg conventions)
- Type-gate detection and precedence for Python repos:
  - `ty` first when configured, then `pyright`, then `mypy`
  - `ty` is mandatory if configured
  - no "baseline acceptable" language for touched files
- “Gotchas” that repeatedly waste time (e.g., djlint inline styles, named
  endblocks, date/time math that must be verified with code/tests)
- Local typing policy docs if present (for example:
  `docs/python-typing-3.14-best-practices.md`, `TY_MIGRATION_GUIDE.md`)

### Phase 3: Documentation Generation

#### AGENTS.md Structure (Required)

```markdown
# AGENTS.md — The [Project Name] Story

*A guide for AI agents and humans alike. [One-sentence description of what this guide explains.]*

---

## What This Repo Is

**[Project Name]** is [brief description of what it does and why it exists].

[Metaphor or analogy that captures its essence.]

```
[HIGH-LEVEL ASCII ARCHITECTURE DIAGRAM]
```

---

## Architecture Diagrams

### [Subsystem 1] Architecture

```
[DETAILED ASCII DIAGRAM]
```

### [Subsystem 2] Architecture

```
[DETAILED ASCII DIAGRAM]
```

---

## Folder Structure

```
project/
|
+-- dir1/            # Description
|       +-- file.ext   # Description
+-- dir2/            # Description
```

---

## Technologies

| Category | Technology | Purpose |
|----------|------------|---------|
| **Language** | ... | ... |
| **Framework** | ... | ... |
| **Database** | ... | ... |

---

## Do

- [Best practice 1]
- [Best practice 2]
- [Best practice 3]

## Don't

- Don't [anti-pattern 1]
- Don't [anti-pattern 2]
- Don't [anti-pattern 3]

---

## Common Commands

```bash
# Development
command1    # Description

# Testing
command2    # Description

# Deployment
command3    # Description
```

---

## The Hard Lessons

### Lesson 1: [Title]

[What happened]

**The cause:** [Root cause]

**The fix:** [Solution]

---

## Quick Reference

**[Category 1]:**
```bash
command
```

**[Category 2]:**
```
key info
```

**When in doubt:** [Most important advice]
```

#### ASCII Diagram Rules (Critical)

**DO use these characters:**
- Box corners: `+`
- Horizontal lines: `-`, `=`
- Vertical lines: `|`
- Arrows: `>`, `<`, `v`, `^`, `---->`, `|`
- Bullet points: `*`, `-`
- Section dividers: `===`, `---`

**DO NOT use:**
- Unicode box-drawing characters (U+2500-U+257F): `─`, `│`, `┌`, `┐`, `└`, `┘`, `├`, `┤`, `┬`, `┴`, `┼`
- Emojis in diagrams
- Fancy Unicode arrows: `→`, `←`, `↓`, `↑`

**Correct example:**
```
+------------------------------------------------------------------+
|                         PROJECT NAME                              |
+------------------------------------------------------------------+
|                                                                   |
|  +------------------+  +------------------+  +------------------+ |
|  | COMPONENT A      |  | COMPONENT B      |  | COMPONENT C      | |
|  | Description      |  | Description      |  | Description      | |
|  +------------------+  +------------------+  +------------------+ |
|                                                                   |
+------------------------------------------------------------------+
```

**Flow diagram example:**
```
+----------------+     +----------------+     +----------------+
| Input          |---->| Process        |---->| Output         |
| (source)       |     | (transform)    |     | (result)       |
+----------------+     +----------------+     +----------------+
        |
        | Annotated flow description
        v
+----------------+
| Next Step      |
+----------------+
```

#### README.md Strategy

**If README.md exists:**
1. Read it entirely
2. Identify human-written content that must be preserved
3. Add architecture overview at the TOP (after title)
4. Add reference to AGENTS.md for detailed documentation
5. Keep all existing sections intact
6. Only add, don't remove

**If README.md doesn't exist:**
- Create a concise version with:
  - Project name and description
  - Architecture overview diagram
  - Quick start instructions
  - Reference to AGENTS.md for details

#### CLAUDE.md Structure (Required)

CLAUDE.md should be minimal and source AGENTS.md:

```markdown
@AGENTS.md

---

## Notes

This `CLAUDE.md` intentionally sources `AGENTS.md` so that requirements,
commands, and agent behavior live in a single source of truth for this repo.

For Claude Code best practices on `CLAUDE.md` / `AGENTS.md` in web-based repos,
see:
- https://docs.claude.com/en/docs/claude-code/claude-code-on-the-web#best-practices

Relevant guidance (summarized):
- Document requirements: clearly specify dependencies and commands for the
  project.
- If you have an `AGENTS.md` file, you can source it in your `CLAUDE.md` using
  `@AGENTS.md` to maintain a single source of truth.
```

### Phase 4: Validation

Before finalizing:

1. **Verify ASCII diagrams render correctly** - No Unicode box characters
2. **Check all paths exist** - Referenced files should exist
3. **Validate commands** - Commands should be runnable
4. **Confirm tech stack** - All mentioned technologies are actually used
5. **Review against existing docs** - No valuable content was lost

## Technology-Specific Patterns

### Python Projects

Look for:
- `pyproject.toml`, `setup.py`, `requirements.txt`
- `uv.lock`, `poetry.lock`, `Pipfile.lock`
- Django: `manage.py`, `settings.py`, apps with `models.py`
- Flask: `app.py`, `wsgi.py`
- FastAPI: `main.py` with `FastAPI()` instance

Common commands:
```bash
# uv-based
uv run python ...
uv sync

# pip-based
pip install -r requirements.txt
python manage.py runserver

# poetry-based
poetry install
poetry run python ...
```

### JavaScript/TypeScript Projects

Look for:
- `package.json`, `yarn.lock`, `package-lock.json`
- `tsconfig.json` for TypeScript
- React: `src/App.tsx`, `src/index.tsx`
- Node.js: `server.js`, `index.js`
- Vite: `vite.config.ts`
- Next.js: `next.config.js`, `pages/` or `app/`

Common commands:
```bash
npm install / yarn
npm run dev / yarn dev
npm run build / yarn build
npm test / yarn test
```

### Java Projects

Look for:
- `pom.xml` (Maven), `build.gradle` (Gradle)
- `src/main/java/`, `src/test/java/`
- Spring Boot: `Application.java`, `application.properties`

Common commands:
```bash
# Maven
mvn clean install
mvn spring-boot:run

# Gradle
./gradlew build
./gradlew bootRun

# Ant (legacy)
ant build
ant clean
```

### Infrastructure/Terraform

Look for:
- `*.tf` files
- `modules/`, `environments/`
- `terraform.tfvars`, `backend.tf`

Common commands:
```bash
terraform init
terraform plan
terraform apply
terraform fmt
```

### Serverless

Look for:
- `serverless.yml` (Serverless Framework)
- `zappa_settings.json` (Zappa)
- `sam.yaml` (AWS SAM)
- `functions/` directory

Common commands:
```bash
serverless deploy --stage dev
serverless offline
zappa deploy staging
```

## Output Shape

When documentation is generated, report:

```
## Documentation Generated

**Repository:** /path/to/repo
**Type:** [Web App / API / Library / Infrastructure / Scripts]
**Tech Stack:** [Primary technologies]

### Files Created/Updated:

| File | Action | Notes |
|------|--------|-------|
| AGENTS.md | Created/Updated | [summary] |
| README.md | Enhanced/Created | [summary] |
| CLAUDE.md | Created/Updated | Sources AGENTS.md |

### Architecture Diagrams Included:

1. [Diagram name 1]
2. [Diagram name 2]
3. [Diagram name 3]

### Preserved Content:

- [Existing section 1 from README.md]
- [Existing section 2 from README.md]

### Recommendations:

- [Any follow-up suggestions]
```

## Important Rules

1. **Always read existing docs first** - Never overwrite without understanding what exists
2. **Use standard ASCII only** - No Unicode box characters in diagrams
3. **Be specific about technologies** - Don't guess; verify by reading config files
4. **Include "The Hard Lessons"** - If you can identify common pitfalls, document them
5. **Test commands before documenting** - Don't document commands that don't work
6. **Reference files that exist** - Don't mention files that aren't in the repo
7. **Keep README.md concise** - Detailed docs go in AGENTS.md
8. **CLAUDE.md sources AGENTS.md** - Single source of truth pattern

## Error Recovery

### Existing AGENTS.md has critical content

If the existing AGENTS.md has valuable content that shouldn't be lost:

1. Extract and preserve critical sections
2. Merge with new structure
3. Show diff to user before writing
4. Ask for confirmation

### Unable to determine tech stack

If you can't identify the technology:

1. List what you found (or didn't find)
2. Ask user for clarification
3. Generate placeholder sections that can be filled in

### Repository is monorepo

If the repo contains multiple distinct projects:

1. Identify each sub-project
2. Consider generating per-project AGENTS.md in subdirectories
3. Create top-level AGENTS.md that explains the monorepo structure
4. Link to sub-project documentation

## Examples

### Example 1: Python Django Project

After analyzing a Django project, the AGENTS.md might include:

```
## Architecture Diagrams

### Request Flow

```
+------------------------------------------------------------------+
|                    DJANGO REQUEST LIFECYCLE                       |
+------------------------------------------------------------------+

+----------------+     +----------------+     +----------------+
| Browser        |---->| nginx          |---->| gunicorn       |
| (HTTP request) |     | (reverse proxy)|     | (WSGI server)  |
+----------------+     +----------------+     +----------------+
                                                      |
                                                      v
                              +----------------+     +----------------+
                              | Django         |---->| PostgreSQL     |
                              | (views/models) |     | (database)     |
                              +----------------+     +----------------+
```
```

### Example 2: React Frontend

After analyzing a React project:

```
## Architecture Diagrams

### Component Hierarchy

```
+------------------------------------------------------------------+
|                    REACT APPLICATION                              |
+------------------------------------------------------------------+

                    +------------------------+
                    | App.tsx                |
                    | (Router, Providers)    |
                    +------------------------+
                               |
         +---------------------+---------------------+
         |                     |                     |
         v                     v                     v
+----------------+   +----------------+   +----------------+
| AuthLayout     |   | DashboardLayout|   | PublicLayout   |
| (login/signup) |   | (main app)     |   | (landing)      |
+----------------+   +----------------+   +----------------+
```
```

### Example 3: Preserving Existing README

If README.md exists with:
```markdown
# My Project

Some important info written by humans.

## Installation

Custom installation instructions.
```

The enhanced README.md becomes:
```markdown
# My Project

Some important info written by humans.

> **Architecture Note**: For detailed architecture documentation, agent guidelines, and development patterns, see [AGENTS.md](./AGENTS.md).

## Architecture Overview

```
[ASCII DIAGRAM]
```

---

## Installation

Custom installation instructions.
```

## Quick Reference

**Discover repo:**
```bash
ls -la && cat README.md 2>/dev/null && cat AGENTS.md 2>/dev/null
```

**Find tech stack:**
```bash
ls package.json pyproject.toml *.tf serverless.yml 2>/dev/null
```

**Check existing docs:**
```bash
find . -maxdepth 2 -name "*.md" -type f
```

**When in doubt:** Read everything first, enhance carefully, preserve human content, use standard ASCII only for diagrams.

---

## Canonicalize Mode

Use `/repo-docs:canonicalize` to audit and fix existing documentation across a repository, making AGENTS.md the canonical source of truth and normalizing all CLAUDE.md files to minimal stubs.

### When to Use Canonicalize

- Repository has scattered/divergent AGENTS.md and CLAUDE.md files
- Documentation contains stale content (old package managers, outdated commands)
- CLAUDE.md files have unique content that should live in AGENTS.md
- Want to enforce the "single source of truth" pattern across a codebase

### Canonicalize Workflow

#### Phase 1: Discovery

Find all directories with AGENTS.md and/or CLAUDE.md:

```bash
# Find all AGENTS.md and CLAUDE.md files recursively
find . -name "AGENTS.md" -o -name "CLAUDE.md" | sort

# Group by directory
find . -name "AGENTS.md" -o -name "CLAUDE.md" | xargs -I{} dirname {} | sort -u
```

#### Phase 2: Per-Directory Analysis

For each directory with docs, analyze the **actual** current behavior:

**Tooling Analysis:**
```bash
# Check for uv (modern Python)
ls uv.lock pyproject.toml 2>/dev/null

# Check for .bin/* wrappers
ls .bin/ 2>/dev/null

# Check CI configuration
ls .github/workflows/*.yml 2>/dev/null
cat .github/workflows/*.yml 2>/dev/null | grep -E "run:|command:" | head -20

# Check for Makefile/scripts
ls Makefile *.sh scripts/ 2>/dev/null

# Check pre-commit + linting configuration (often the real “source of truth”)
ls .pre-commit-config.yaml .pre-commit-config.yml 2>/dev/null
rg -n "djlint|ruff|mypy|pyright|ty|eslint|prettier|black|isort" .pre-commit-config.yaml .pre-commit-config.yml pyproject.toml package.json 2>/dev/null | head -50
```

**Identify Stale Patterns (remove these):**
- `pip install -r requirements.txt` → should be `uv sync`
- `poetry install` / `poetry run` → should be `uv sync` / `uv run`
- `python manage.py` → should be `.bin/django` or `uv run python manage.py`
- `pytest` → should be `.bin/pytest` or `uv run pytest`
- References to old Django versions, old CI patterns, removed features

**Identify Current Patterns (use these):**
- `uv sync`, `uv run ...`
- `.bin/django`, `.bin/pytest`, `.bin/ruff`, plus active type gate wrapper
  (`.bin/ty`, `.bin/pyright`, or `.bin/mypy`)
- `pre-commit run ...` (and any repo wrappers like `make precommit`)
- Current CI job names and commands
- Actual architecture from code inspection

#### Phase 3: Comparison & Merge

For each directory:

1. **Read both files:**
   ```bash
   cat AGENTS.md 2>/dev/null
   cat CLAUDE.md 2>/dev/null
   ```

2. **Identify content categories:**
   - **Keep**: Still true and important
   - **Remove**: Stale, outdated, or incorrect
   - **Merge**: In CLAUDE.md but not AGENTS.md (move to AGENTS.md)

3. **Merge strategy:**
   - If CLAUDE.md has better/more current detail → move to AGENTS.md
   - If both have same info → keep AGENTS.md version
   - If conflict → prefer what matches actual code behavior

#### Phase 4: Rewrite AGENTS.md

Update AGENTS.md to be canonical for its directory scope:

**Command updates:**
````markdown
## Common Commands

```bash
# Environment setup
uv sync                              # Install dependencies

# Running the server
.bin/django runserver                # Or: uv run python manage.py runserver

# Testing
.bin/pytest                          # Or: uv run pytest
.bin/pytest -x -vvs path/to/test.py  # Single test with output

# Linting
.bin/ruff check .                    # Check for issues
.bin/ruff format .                   # Auto-format

# Pre-commit (if present, this is usually the authoritative lint gate)
pre-commit run --all-files           # Or: pre-commit run --files <changed files>

# Type checking
# Prefer ty when configured, then pyright, then mypy
.bin/ty check .                      # Run ty (if configured)
.bin/pyright .                       # Or pyright
.bin/mypy .                          # Or mypy
```
````

**Behavior descriptions must match actual code:**
- Check what CI actually runs, not what old docs say
- Verify management commands exist before documenting them
- Confirm architecture matches current codebase structure
- If `.pre-commit-config.*` exists, summarize key hooks and “gotchas” in
  AGENTS.md (e.g. djlint template rules, required commit message prefixes) so
  agents write compliant code on the first pass instead of discovering failures
  at commit time.

#### Phase 5: Normalize CLAUDE.md

Replace every CLAUDE.md with this minimal stub:

```markdown
@AGENTS.md

---

## Notes

This `CLAUDE.md` intentionally sources `AGENTS.md` so that requirements,
commands, and agent behavior live in a single source of truth for this repo.

For Claude Code best practices on `CLAUDE.md` / `AGENTS.md` in web-based repos,
see:
- https://docs.claude.com/en/docs/claude-code/claude-code-on-the-web#best-practices

Relevant guidance (summarized):
- Document requirements: clearly specify dependencies and commands for the
  project.
- If you have an `AGENTS.md` file, you can source it in your `CLAUDE.md` using
  `@AGENTS.md` to maintain a single source of truth.
```

**Important:** No additional rules, requirements, or divergent specs in CLAUDE.md. All behavioral guidance must live in AGENTS.md.

### Canonicalize Output Shape

```
## Documentation Canonicalized

**Repository:** /path/to/repo
**Directories processed:** N

### Summary by Directory:

| Directory | AGENTS.md | CLAUDE.md | Changes |
|-----------|-----------|-----------|---------|
| `/` | Updated | Normalized | Merged 3 sections, removed stale pip commands |
| `/backend` | Updated | Normalized | Updated to .bin/* wrappers |
| `/infra` | Created | Normalized | Was CLAUDE.md only, now has AGENTS.md |

### Stale Content Removed:

- `pip install -r requirements.txt` → `uv sync`
- `poetry run pytest` → `.bin/pytest`
- References to Django 3.x (now 4.x)
- Old CI workflow names

### Content Merged from CLAUDE.md → AGENTS.md:

- `/backend/CLAUDE.md`: Database migration safety rules
- `/infra/CLAUDE.md`: Terraform state locking guidance

### All CLAUDE.md files now source @AGENTS.md
```

### Canonicalize Rules

1. **Analyze before changing** - Always understand actual behavior first
2. **AGENTS.md is the source of truth** - All rules live there
3. **CLAUDE.md is just a pointer** - Minimal stub, no unique content
4. **Commands must be current** - `uv`, `.bin/*` wrappers, actual CI commands
5. **Remove stale content aggressively** - Old package managers, old patterns
6. **Merge, don't lose** - Move valuable CLAUDE.md content to AGENTS.md first
7. **Recursive** - Process every directory with docs, not just root
8. **Confirm before destructive changes** - Show diff, ask user if uncertain
9. **Encode recurring failure modes** - If linters (pre-commit/djlint) or
   reasoning pitfalls (date/time math) repeatedly trip agents, write explicit
   “gotchas” and verification commands into AGENTS.md so they’re avoided up
   front.

### Handling Edge Cases

#### Directory has CLAUDE.md but no AGENTS.md

1. Create AGENTS.md from CLAUDE.md content
2. Update to current patterns
3. Replace CLAUDE.md with minimal stub

#### Directory has both but CLAUDE.md has unique rules

1. Merge unique rules into AGENTS.md
2. Verify rules match actual code behavior
3. Replace CLAUDE.md with minimal stub

#### AGENTS.md references files/commands that don't exist

1. Remove or update the reference
2. Document what actually exists
3. Note the removal in output report

#### Subdirectory AGENTS.md contradicts parent

1. Subdirectory AGENTS.md is authoritative for its scope
2. Parent should not repeat subdirectory rules
3. Each AGENTS.md documents its directory scope only
