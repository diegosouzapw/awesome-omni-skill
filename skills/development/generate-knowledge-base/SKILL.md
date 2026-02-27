---
name: generate-knowledge-base
description: Generate a product knowledge base from a codebase. Analyzes source code to create an Obsidian vault with architecture docs, API references, domain logic, data models, and infrastructure documentation. Use when the user asks to document a codebase, create a knowledge base, or generate product docs.
argument-hint: <codebase-path>
---

# Generate Product Knowledge Base

You are generating a comprehensive product knowledge base from source code analysis. The output is an Obsidian vault with interconnected documents covering architecture, data models, APIs, business domains, and infrastructure.

## Before You Start

Read these reference files to understand the expected output format and quality criteria:
- `references/document-formats.md` — the 4-part document structure with examples
- `references/category-patterns.md` — where to find information for each tech stack
- `references/quality-checklist.md` — self-review criteria for every document

## Workflow

Execute these steps in order. Do not skip steps. Wait for user approval at Step 2 before generating documents.

### Step 1 — Setup & Discovery

Gather project information:

1. **Product name**: Ask the user for the product/project name. Use it in all generated doc titles and references.
2. **Codebase path**: Use `$ARGUMENTS` if provided, otherwise ask the user. Resolve to an absolute path. Verify the directory exists.
3. **Output directory**: Ask where to write the vault. Default: a sibling directory named `<product>-knowledge/` next to the codebase.

Detect the tech stack:

1. Glob for marker files at the codebase root and one level deep:
   - `package.json`, `tsconfig.json` → JavaScript/TypeScript
   - `requirements.txt`, `pyproject.toml`, `setup.py`, `Pipfile` → Python
   - `pom.xml`, `build.gradle`, `build.gradle.kts` → Java/Kotlin
   - `go.mod` → Go
   - `Cargo.toml` → Rust
   - `Gemfile` → Ruby
   - `composer.json` → PHP
   - `mix.exs` → Elixir
   - `*.sln`, `*.csproj` → C#/.NET

2. Read each detected marker file to identify specific frameworks:
   - `package.json` → check `dependencies` for `next`, `express`, `nestjs`, `react`, etc.
   - `requirements.txt` / `pyproject.toml` → check for `django`, `fastapi`, `flask`, etc.
   - `build.gradle.kts` → check for `ktor`, `spring-boot`, etc.
   - `go.mod` → check for `gin`, `echo`, `fiber`, etc.

3. Map the directory structure:
   - Find top-level directories: `src/`, `app/`, `cmd/`, `internal/`, `lib/`, `pkg/`, `server/`, `services/`, `api/`, `routes/`, `controllers/`, `models/`, `views/`, `templates/`, `static/`, `public/`, `frontend/`, `backend/`, `infra/`, `terraform/`, `deploy/`, `migrations/`, `.github/`, `.circleci/`
   - Identify monorepo patterns: multiple `package.json` files, workspace configs, `services/` directories with independent modules
   - Find test directories: `test/`, `tests/`, `__tests__/`, `spec/`
   - Find SDK/client directories: `sdk/`, `client/`, `packages/`

4. Report findings to the user:
   ```
   Detected: [Language] with [Framework]
   Services: [list of services/modules found]
   Database: [type if detected from configs]
   Infrastructure: [CI/CD, cloud provider if found]
   ```

### Step 2 — Plan the Vault

Based on detected tech stack, determine which categories to generate:

**Always include:**
- `architecture/` — system overview, tech stack, data flows
- `api/` — endpoint documentation (if HTTP routes found)
- `domains/` — business logic by domain

**Include if relevant sources found:**
- `data-model/` — if migration files, ORM models, or schema definitions found
- `infrastructure/` — if Terraform, CloudFormation, Docker, or CI configs found
- `sdks/` — if SDK or client library code found
- `services/` — if multiple backend services (monorepo/microservices)
- `integrations/` — if third-party service integrations found

Identify business domains by analyzing:
- Directory names under `src/`, `app/`, `internal/`, `services/`
- Route/controller groupings
- Model/entity names
- Service class names

Present the plan to the user:
```
## Generation Plan

Product: [name]
Output: [path]
Tech Stack: [detected]

### Documents to Generate (~XX total)

**Architecture** (X docs)
- architecture/overview.md
- architecture/tech-stack.md
- ...

**API** (X docs)
- api/overview.md
- ...

**Domains** (X docs)
- domains/[domain-1]/overview.md
- ...

Shall I proceed?
```

**Wait for explicit user approval before continuing.**

### Step 3 — Generate Architecture Docs

Generate 3-8 architecture documents by reading:
- README files, docker-compose files
- Entry points (`main.ts`, `app.py`, `Application.kt`, `main.go`, etc.)
- Infrastructure configs (Terraform, CloudFormation, Dockerfile)
- Build configs (`package.json` scripts, `Makefile`, `build.gradle.kts`)

Required documents:
- `architecture/overview.md` — system topology with a Mermaid diagram showing services, data stores, and external dependencies
- `architecture/tech-stack.md` — languages, frameworks, databases, queues, cloud services with version numbers where available

Optional documents (create if sufficient source material exists):
- `architecture/data-flow.md` — request lifecycle, async processing flows
- `architecture/backend-services.md` — service responsibilities, ports, deployment
- `architecture/frontend-apps.md` — frontend architecture, routing, state management

### Step 4 — Generate Data Model Docs

Generate 2-10 data model documents by reading:
- Migration files (`migrations/`, `db/migrate/`, `alembic/`)
- ORM models (Django `models.py`, SQLAlchemy models, Exposed tables, GORM structs)
- Schema definitions (SQL files, Prisma schema, TypeORM entities)
- Seed data files

Required documents:
- `data-model/overview.md` — database architecture, schema organization

Per-entity documents:
- `data-model/<entity>.md` — table/collection schema with columns, types, constraints, relationships

### Step 5 — Generate API Docs

Generate 3-20 API documents by reading:
- Route definitions (Express routers, Django URLs, Ktor routing, Go handlers)
- Controller/handler implementations
- OpenAPI/Swagger specs if available
- Middleware (auth, validation, rate limiting)
- Request/response types (protobuf, TypeScript interfaces, Pydantic models)

Required documents:
- `api/overview.md` — API architecture, authentication methods, common patterns

Per-resource documents:
- `api/<resource>.md` — endpoints for a resource group with routes, methods, request/response shapes, and auth requirements

If the codebase has multiple API servers (external + internal, public + admin), organize as:
- `api/external-api/overview.md`
- `api/internal-api/overview.md`

### Step 6 — Generate Domain Docs

Generate 10-30 domain documents. This is the largest category and should be chunked.

For each identified business domain:
1. Read service layer, domain models, and business logic files
2. Generate `domains/<domain>/overview.md` — concept, lifecycle, state machine
3. Generate `domains/<domain>/<feature>.md` — specific feature logic

**Chunking strategy:**
- Generate domains in batches of 5-10 documents
- After each batch, verify wikilinks between generated docs
- Continue until all domains are covered

Use the Task tool to parallelize independent domain research when the codebase is large.

### Step 7 — Generate Infrastructure Docs

Generate 2-5 infrastructure documents by reading:
- Terraform/CloudFormation/Pulumi files
- CI/CD configs (`.github/workflows/`, `.circleci/`, `Jenkinsfile`, `.gitlab-ci.yml`)
- Docker files (`Dockerfile`, `docker-compose.yml`)
- Monitoring configs (CloudWatch, Datadog, Prometheus)
- Deployment scripts

Required documents:
- `infrastructure/overview.md` — cloud architecture, deployment topology

Optional documents:
- `infrastructure/ci-cd.md` — build and deploy pipeline
- `infrastructure/monitoring.md` — observability, alerting, logging
- `infrastructure/database-management.md` — backup, scaling, connection pooling

### Step 8 — Finalize

1. **Generate README.md**: Create the vault's master index using the `assets/README.md.template`. List every generated document as a `[[wikilink]]` organized by category.

2. **Generate CLAUDE.md**: Create the vault's CLAUDE.md using the `assets/CLAUDE.md.template`. Fill in:
   - Product name
   - Vault structure (categories and their contents)
   - Source code paths table
   - Conventions (wikilinks, document format, Mermaid diagrams)

3. **Validate wikilinks**: Run `scripts/validate-wikilinks.sh` on the output directory. Fix any broken links it reports.

4. **Print summary**:
   ```
   ## Generation Complete

   Product: [name]
   Location: [path]
   Documents: [count] across [N] categories
   Wikilinks: [count] total, [broken] broken

   Categories:
   - architecture/: X docs
   - data-model/: X docs
   - api/: X docs
   - domains/: X docs
   - infrastructure/: X docs

   Open the vault in Obsidian to browse the knowledge graph.
   ```

## Key Rules

1. **Code-first**: Every statement must trace to actual source code. Never invent or assume logic. If you cannot find the implementation, say "Not found in source" rather than guessing.

2. **Source attribution**: Every document must include a `> **Source files**:` block listing the exact files analyzed. Use relative paths from the codebase root.

3. **Fully-qualified wikilinks**: Always use the full path from the vault root: `[[domains/campaigns/overview]]`, never `[[overview]]` or `[[campaigns/overview]]`.

4. **One concern per file**: Each document covers exactly one topic. Split large topics into multiple documents.

5. **Mermaid diagrams**: Include a Mermaid diagram for any flow with 3+ steps. Use `graph TD/TB/LR` for flowcharts and `sequenceDiagram` for interaction flows.

6. **No marketing language**: Write for engineers. Include file paths, function names, and implementation details. This is internal documentation, not a product page.

7. **Quality check**: Before finalizing each document, verify it against `references/quality-checklist.md`.
