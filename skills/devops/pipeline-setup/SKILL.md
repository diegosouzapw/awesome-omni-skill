---
name: pipeline-setup
description: "Set up a repository for the agent pipeline. Auto-detects framework, stack, and directory structure, then writes the Pipeline Configuration section into the conventions file."
disable-model-invocation: true
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Write
  - Edit
  - Task
  - AskUserQuestion
---

# Pipeline Setup

You **set up a repository for the agent pipeline** by detecting its framework, stack, and directory structure, then writing a `## Pipeline Configuration` section into the repo's conventions file. This is the entry point for adding any repo to the pipeline.

## Inputs

- The current working directory (must be a git repository)
- `$ARGUMENTS` — optional. If provided, treated as a path to a different repo to set up.

## Outputs

- A `## Pipeline Configuration` section appended to the repo's conventions file (`CLAUDE.md`, `AGENTS.md`, or `CONVENTIONS.md`)
- WCP artifact types ensured for the target namespace (if WCP is available)

## Step-by-Step Procedure

### Step 1: Determine Target Repo

If `$ARGUMENTS` is provided and non-empty, use that as the target repo path. Otherwise, use the current working directory.

Verify it's a git repository:

```bash
git -C <target-path> rev-parse --git-dir 2>/dev/null
```

If it fails, check if the directory contains subdirectories that are git repos:

```bash
ls -d <target-path>/*/
```

If child repos are found, use `AskUserQuestion` to let the user pick which repo to set up:

> "This directory isn't a git repo, but I found these repos inside it: [list]. Which one should I set up?"

If no git repos are found, **STOP**:

> "No git repository found at `<target-path>` or in its children."

### Step 2: Check for Existing Pipeline Configuration

Read the conventions file (first of `CLAUDE.md`, `AGENTS.md`, `CONVENTIONS.md` found in repo root).

If a `## Pipeline Configuration` section already exists, use `AskUserQuestion`:

> "This repo already has a Pipeline Configuration section in `<file>`. Overwrite it, or skip?"

If "skip", **STOP** with a success message.

If no conventions file exists at all, use `AskUserQuestion`:

> "No conventions file found. Which file should I create?"
>
> Options: `CLAUDE.md` (Recommended), `AGENTS.md`, `CONVENTIONS.md`

### Step 3: Detect Default Branch

```bash
git -C <target-path> symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's|refs/remotes/origin/||'
```

If that fails, fall back to:

```bash
git -C <target-path> branch --show-current
```

Store as `default-branch`.

### Step 4: Detect Framework and Stack

Use the Task tool with an Explore agent to analyze the repository. The agent should detect everything listed below. Pass the target repo path to the agent.

**Language & Framework** — detect from dependency files:

| File | Language | Framework detection |
|------|----------|-------------------|
| `Gemfile` | Ruby | Look for `rails` gem → Rails (extract version). Read `.ruby-version` for Ruby version. |
| `mix.exs` | Elixir | Look for `:phoenix` → Phoenix. Look for `:scenic` → Scenic. |
| `package.json` | JavaScript/TypeScript | Look for `next` → Next.js, `express` → Express, `@angular/core` → Angular, `react` → React. Check for `typescript` dep → TypeScript. |
| `requirements.txt` or `pyproject.toml` | Python | Look for `django` → Django, `flask` → Flask, `fastapi` → FastAPI. |
| `go.mod` | Go | Module name for framework hints. |
| `Cargo.toml` | Rust | Check for `actix-web`, `axum`, `rocket`. |
| `Package.swift` or `*.xcodeproj` | Swift | iOS app. |
| `build.gradle` or `build.gradle.kts` | Kotlin/Java | Android app. |

**Test framework:**

| Language | Detection |
|----------|-----------|
| Ruby | `rspec-rails` in Gemfile → RSpec, `minitest` → Minitest |
| Elixir | ExUnit (default) |
| JS/TS | `jest` in package.json → Jest, `vitest` → Vitest, `mocha` → Mocha |
| Python | `pytest` in requirements → pytest, else unittest |
| Go | Built-in `go test` |

**Additional stack details** (detect what's present, skip what's not):

| Category | Ruby/Rails | JS/TS | Python | Elixir |
|----------|-----------|-------|--------|--------|
| **ORM** | ActiveRecord (default) | Prisma, Sequelize, TypeORM | Django ORM, SQLAlchemy | Ecto |
| **CSS** | `tailwindcss-rails`, `bootstrap`, `sassc-rails` | `tailwindcss`, `@mui/material` | — | — |
| **JS bundling** | `importmap-rails`, `jsbundling-rails` | Webpack, Vite, esbuild | — | esbuild in `assets/` |
| **Asset pipeline** | `propshaft`, `sprockets` | — | — | — |
| **Database** | `pg`, `sqlite3`, `mysql2` in Gemfile | Check `prisma/schema.prisma` or config | Check settings.py | Check `config/dev.exs` |
| **Background jobs** | `sidekiq`, `solid_queue`, `good_job` | `bullmq`, `bee-queue` | `celery`, `rq` | `oban` |
| **Deploy** | `kamal`, `capistrano`, `Procfile` | `Dockerfile`, `vercel.json`, `netlify.toml` | `Dockerfile`, `Procfile` | `Dockerfile`, `fly.toml` |
| **Email** | `resend`, `postmark`, `sendgrid` | — | — | `swoosh` |

**Test command:**

| Indicator | Command |
|-----------|---------|
| `.rspec` file or `rspec-rails` in Gemfile | `bundle exec rspec` |
| `jest` in package.json | `npx jest` |
| `vitest` in package.json | `npx vitest run` |
| `pytest` or `conftest.py` | `pytest` |
| `mix.exs` | `mix test` |
| `go.mod` | `go test ./...` |

**Test directory:**

| Framework | Directory |
|-----------|-----------|
| RSpec | `spec/` |
| Minitest (Rails) | `test/` |
| Jest/Vitest | `test/`, `__tests__/`, or `src/**/*.test.*` |
| pytest | `tests/` or `test/` |
| ExUnit | `test/` |
| Go | Inline (`*_test.go` files) |

### Step 5: Detect Directory Structure

Use Glob to find which standard directories exist. Only include directories that actually exist in the repo.

**Rails directories to check:**
`app/models/`, `app/controllers/`, `app/views/`, `app/services/`, `app/helpers/`, `app/javascript/`, `app/javascript/controllers/`, `app/jobs/`, `app/mailers/`, `app/blueprints/`, `app/serializers/`, `config/routes.rb`, `db/migrate/`, `db/schema.rb`, `spec/models/`, `spec/requests/`, `spec/controllers/`, `spec/services/`, `spec/jobs/`, `spec/mailers/`, `spec/system/`, `spec/features/`, `spec/factories/`, `spec/support/`, `test/`, `lib/tasks/`

**Phoenix directories to check:**
`lib/*/`, `lib/*_web/controllers/`, `lib/*_web/live/`, `lib/*_web/templates/`, `lib/*_web/components/`, `priv/repo/migrations/`, `test/`, `test/support/`

**Node.js directories to check:**
`src/`, `src/controllers/`, `src/models/`, `src/routes/`, `src/services/`, `src/middleware/`, `test/`, `__tests__/`, `prisma/`

**Python/Django directories to check:**
`*/models.py`, `*/views.py`, `*/serializers.py`, `*/urls.py`, `*/tests/`, `migrations/`, `templates/`

**General directories to check:**
`src/`, `lib/`, `test/`, `tests/`, `config/`, `scripts/`

### Step 6: Detect Post-Flight Checks

Look for linters and security tools in dependency files and CI config:

| Tool | Detect via | Command | Auto-fix? | Blocking? |
|------|-----------|---------|-----------|-----------|
| RuboCop | `rubocop` in Gemfile | `bin/rubocop -A` or `bundle exec rubocop -A` | Yes | Yes |
| StandardRB | `standard` in Gemfile | `bundle exec standardrb --fix` | Yes | Yes |
| Brakeman | `brakeman` in Gemfile | `bin/brakeman --quiet --no-pager --exit-on-warn --exit-on-error` | No | Yes |
| bundler-audit | `bundler-audit` in Gemfile | `bundle exec bundler-audit` | No | Yes |
| ESLint | `eslint` in package.json | `npx eslint --fix .` | Yes | Yes |
| Prettier | `prettier` in package.json | `npx prettier --write .` | Yes | Yes |
| TypeScript | `typescript` in package.json | `npx tsc --noEmit` | No | Yes |
| ruff | `ruff` in requirements | `ruff check --fix .` | Yes | Yes |
| mypy | `mypy` in requirements | `mypy .` | No | Yes |
| bandit | `bandit` in requirements | `bandit -r src/` | No | Yes |
| Credo | `:credo` in mix.exs | `mix credo --strict` | No | Yes |
| importmap audit | `importmap-rails` in Gemfile | `bin/importmap audit` | No | Yes |

Only include tools that are actually present in the project's dependencies.

### Step 7: Present Findings for Confirmation

Use `AskUserQuestion` to present the detected configuration. Ask these questions:

**Question 1** (single-select): Confirm the detected stack.

> "I detected **[Language] [version]** / **[Framework] [version]** with **[test framework]** and **[database]**. Default branch: `[branch]`. Is this correct?"
>
> Options: "Yes, that's correct", "Let me correct some details"

If the user wants to correct, ask follow-up questions for the specific values.

**Question 2** (multi-select): Which optional sections apply?

> "Which of these apply to this repo?"
>
> Options:
> - "API Conventions — has API endpoints consumed by other clients"
> - "Multi-Tenant Security — multiple accounts/orgs with data isolation"
> - "Backwards Compatibility — must support old clients or API versions"
> - "Post-Flight Checks — linting and security scanning before PRs" (pre-select if tools were detected)

**Question 3** (single-select): Project tracker.

> "What project tracker do you use?"
>
> Options: "None", "Linear", "GitHub Issues"

If Linear: ask for team name. If GitHub Issues: ask for repository name.

### Step 8: Ask About Optional Sections

For each optional section the user selected in Question 2, ask the minimum necessary to populate it:

**API Conventions:**
> "What serialization format do you use?" Options: "jbuilder", "Blueprinter", "ActiveModel Serializers", "Alba", "Plain JSON", "Other"
>
> "What's your API versioning strategy?" Options: "URL path (/api/v1/...)", "Header-based", "No versioning", "Other"

**Multi-Tenant Security:**
> "What's the tenant model called?" (free text, default: "Account")
>
> "How is the current tenant set?" (free text, default: "`current_account` in ApplicationController")

**Backwards Compatibility:**
> "What needs backwards compatibility?" Options: "API versions", "Mobile app versions", "Both", "Other"

**Post-Flight Checks:** Show the detected tools from Step 6 and ask:
> "I detected these tools: [list with commands]. Include all of them?"
>
> Options: "Yes, include all", "Let me pick which ones"

### Step 9: Determine Implementation Order

Based on the detected framework, use the standard build sequence:

**Rails:**
1. Migration(s)
2. Model(s) — with validations, associations, scopes
3. Service(s) — business logic
4. Route(s)
5. Controller(s)
6. Views (ERB)
7. Stimulus controller(s)

**Phoenix/Elixir:**
1. Migration(s)
2. Schema(s) / Context(s)
3. Router entries
4. Controller(s) / LiveView(s)
5. Templates / Components
6. Background workers

**Django:**
1. Model(s) + migration(s)
2. Serializer(s) / Form(s)
3. View(s) / ViewSet(s)
4. URL route(s)
5. Template(s)
6. Management command(s)

**Node.js/Express:**
1. Database schema / migration(s)
2. Model(s) / ORM setup
3. Service(s) — business logic
4. Route(s) + controller(s)
5. Middleware
6. Frontend components (if applicable)

**React/Next.js:**
1. Type definitions / interfaces
2. API route(s) / server actions
3. Data fetching hooks / services
4. Component(s)
5. Page(s) / layout(s)
6. Styles

### Step 10: Generate Pipeline Configuration

Build the `## Pipeline Configuration` markdown section. Use the exact table format shown below — skills parse these tables by header names.

```markdown
## Pipeline Configuration

> Pipeline skills read this section to understand how to run the agent pipeline against this repo.
> Run skills from this repo's root directory (not from the pipeline repo).

### Project Tracker

| Setting | Value |
|---------|-------|
| **Tool** | [None / Linear / GitHub Issues] |
[If Linear: | **Team** | [team-name] |]
[If GitHub Issues: | **Repository** | [repo-name] |]

### Repository Details

| Setting | Value |
|---------|-------|
| **Default branch** | `[default-branch]` |
| **Test command** | `[test-command]` |
| **Test directory** | `[test-directory]` |
| **Branch prefix** | `pipeline/` |
| **PR base branch** | `[default-branch]` |

### Platforms

| Platform | Status | Notes |
|----------|--------|-------|
| [Platform label] | Active | [description] |

### Framework & Stack

| Setting | Value |
|---------|-------|
| **Language** | [detected language + version] |
| **Framework** | [detected framework + version] |
| **Test framework** | [detected test framework] |
[include every additional stack detail detected — ORM, CSS, JS bundling, etc.]

### Directory Structure

| Purpose | Path |
|---------|------|
[one row per detected directory, with a human-readable purpose label]

### Implementation Order

[numbered list from Step 9]

### Guardrails

(REQUIRED — safety rules for agents)

| Guardrail | Rule |
|-----------|------|
| **Production access** | Agents NEVER have production access. No deploy credentials, no production database access. |
| **Default branch** | Never commit or merge directly to the default branch. |
| **Push** | Never push without explicit user request. |
| **Destructive operations** | No `drop_table`, `reset`, or data deletion without human approval. |
```

Then append any optional sections the user confirmed (Post-Flight Checks, API Conventions, Multi-Tenant Security, Backwards Compatibility).

### Step 11: Write the Configuration

**If the conventions file already exists** (and user chose to overwrite):
- If a `## Pipeline Configuration` section exists, replace everything from `## Pipeline Configuration` to the next `## ` heading (or end of file) with the new section.
- If no Pipeline Configuration section exists, append it to the end of the file.

**If creating a new conventions file:**
- Write a minimal file with just a title and the Pipeline Configuration section:

```markdown
# [Repo Name] — Claude Code Instructions

[The user can add repo-specific instructions above this line.]

## Pipeline Configuration

[generated content]
```

### Step 12: Ensure WCP Artifact Types

If the WCP MCP server is available (`mcp__wcp__wcp_schema`), ensure the required pipeline artifact types exist for the namespace that will be used for pipeline runs.

Call `wcp_schema_update` to add any missing artifact types:

```
wcp_schema_update(namespace, add_artifact_types=["progress", "metrics", "quality"])
```

The default WCP schema already includes `prd`, `discovery`, `architecture`, `adr`, `gameplan`, `plan`, `test-matrix`, `review`, and `qa-plan`. The three above are pipeline-specific extensions.

If the WCP MCP server is not available, skip this step silently.

### Step 13: Check for Pipeline Skills

Check if `.claude/skills/prd/SKILL.md` exists in the target repo.

If skills are NOT present, tell the user:

> "Pipeline skills aren't installed yet. Copy them from the pipeline repo:"
> ```
> cp -r <agent-pipeline-path>/.claude/skills/* <target-repo>/.claude/skills/
> ```

If skills ARE present, skip this message.

## When You're Done

Tell the user:

1. **What was written:** "Added Pipeline Configuration to `<file>` with [N] required sections and [N] optional sections."

2. **What was detected:** Brief summary of framework, test framework, database, and any notable findings.

3. **Next steps:**
   - Review the Pipeline Configuration section and adjust any values
   - Copy pipeline skills if not already present (show command only if needed)
   - Create a WCP work item with your brief, then run `/prd <callsign>` to start your first project

## What NOT To Do

- **Do not modify source code.** Only write to the conventions file.
- **Do not run tests.** Only read files and run non-destructive commands.
- **Do not commit anything.** The user reviews and commits.
- **Do not guess.** If detection is ambiguous, ask via `AskUserQuestion`.
- **Do not include optional sections the user didn't confirm.** Fewer sections is better.
- **Do not include directories that don't exist.** Only list directories actually found in the repo.
