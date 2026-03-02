---
name: planner
version: 1.0.0
min_doctl_version: "1.82.0"
description: Generate staged project plans from design through deployment. Use when planning App Platform projects, breaking complex deployments into resumable stages, or tracking multi-step infrastructure setup.
related_skills: [designer, deployment, postgres, networking]
deprecated: false
---

# Planner Skill

Generate staged project plans that break complex projects into manageable, resumable steps - from initial design through coding, testing, to cloud deployment.

> **Tip**: For a complete overview of all available skills, see the [root SKILL.md](../../SKILL.md).

---

## Greenfield vs Brownfield Detection

| Scenario | Detection | Behavior |
|----------|-----------|----------|
| **Greenfield** | No Dockerfiles, no `.do/app.yaml` | Create artifacts from scratch |
| **Brownfield** | Dockerfiles and/or `.do/app.yaml` exist | Review, validate, enhance existing |

The AI assistant automatically detects the project state. Don't recreate valid artifacts. The critical validation gate is `doctl app dev build` — if that passes, artifacts are correct.

---

## Why Staged Deployment?

Complex App Platform deployments fail when executed monolithically:

1. **Blind deployment fails** - Deploying without infrastructure prep results in cryptic errors
2. **Context is lost** - Credentials, cluster IDs, and progress are forgotten between sessions
3. **Debugging is harder** - When everything deploys at once, it's unclear which component failed
4. **Resumability is poor** - Starting over wastes time; partial progress isn't tracked

---

## Build Locally First

**CRITICAL**: Always validate builds locally before cloud deployment.

```bash
doctl app dev build                    # Build using local app spec
doctl app dev build --app <APP_ID>     # Build using existing deployed app
doctl app dev build --env-file .env    # Build with environment overrides
```

| Build Location | Feedback Time | Cost of Failure |
|----------------|---------------|-----------------|
| Cloud | 5-10 minutes | Wasted cycle, confusing logs |
| Local | 30-60 seconds | Immediate fix, clear errors |

**Full details**: See [build-local-first.md](reference/build-local-first.md)

---

## Tier Classification

### Detection from User Description

| User Mentions | Tier |
|--------------|------|
| "static", "frontend only", "no database", "SPA" | Tier 1 |
| "PostgreSQL", "MySQL", "database", "API + frontend" | Tier 2 |
| "Kafka", "OpenSearch", "workers", "event-driven", "microservices" | Tier 3 |

### Decision Tree

```
Does user mention Kafka, OpenSearch, workers, or "event-driven"?
  YES → Tier 3 (Complex)
  NO → Does user mention database (PostgreSQL, MySQL, MongoDB)?
         YES → Tier 2 (Database-backed)
         NO → Tier 1 (Simple)
```

### Detection from app.yaml (Fallback)

If `.do/app.yaml` exists:
- `workers` present → Tier 3
- `databases` with `KAFKA` or `OPENSEARCH` → Tier 3
- `databases` present → Tier 2
- Neither → Tier 1

---

## Tier Structures

### Tier 1: Simple (5 stages)

**Criteria**: Static sites, single service, no database

```
Plan/
├── 01-local-design.md
├── 02-local-coding.md
├── 03-local-testing.md
├── 04-cloud-deploy.md
└── 05-cloud-validate.md
```

**Examples**: Next.js static export, React SPA, Hugo site, Astro

### Tier 2: Database-Backed (7 stages)

**Criteria**: 1-2 services, single database (PostgreSQL/MySQL)

```
Plan/
├── 01-local-design.md
├── 02-local-coding.md
├── 03-local-testing.md
├── 04-cloud-database.md
├── 05-cloud-config.md
├── 06-cloud-deploy.md
└── 07-cloud-validate.md
```

**Examples**: Rails + PostgreSQL, Django + MySQL, Node.js API + Postgres

### Tier 3: Complex (9 stages)

**Criteria**: Multi-service, workers, Kafka/OpenSearch, VPC networking

```
Plan/
├── 01-local-design.md
├── 02-local-coding.md
├── 03-local-testing.md
├── 04-cloud-database-clusters.md
├── 05-cloud-config-users-topics.md
├── 06-cloud-debug-validation.md
├── 07-cloud-secrets-cicd.md
├── 08-cloud-deploy-production.md
└── 09-cloud-end-to-end.md
```

**Examples**: Event-driven microservices, ML pipelines, real-time analytics

> **CRITICAL: Tier 3 apps MUST use GitHub Actions for deployment**
>
> Tier 3 apps have `${SECRET_NAME}` placeholders only resolved by `digitalocean/app_action/deploy@v2`.
> - Set `deploy_on_push: false` in `.do/app.yaml`
> - Deploy via `git push` (triggers GitHub Actions), NOT `doctl apps create`

---

## Stage Templates

Stage templates are provided in the `templates/` directory:

| Directory | Purpose |
|-----------|---------|
| `templates/local/` | Local stages 1-3 (all tiers) |
| `templates/tier-1-simple/` | Tier 1 cloud stages |
| `templates/tier-2-database/` | Tier 2 cloud stages |
| `templates/tier-3-complex/` | Tier 3 cloud stages |

Each stage file follows this structure:

```markdown
# Stage N: [Stage Name]

**Status**: TODO
**Prerequisites**: Stage N-1 complete

## Tasks
- [ ] Task description
- [ ] Expected outcome

## Verification
[Commands to verify completion]

## Next Steps
Proceed to Stage N+1
```

**Key validation gate**: Stage 3 includes `doctl app dev build` and health check validation.

---

## Usage Examples

### Example 1: Simple Static Site

**User**: "Deploy a Next.js static site"

1. Detect Tier 1 (no database)
2. Generate 5 stage files in Plan/
3. Return: "Created 5-stage plan. Start with Stage 1: Local Design."

### Example 2: API with Database

**User**: "Node.js API with PostgreSQL"

1. Detect Tier 2 (PostgreSQL)
2. Generate 7 stage files in Plan/
3. Return: "Created 7-stage plan. Database cluster created in Stage 4."

### Example 3: Event-Driven Microservices

**User**: "Event-driven app with Kafka and OpenSearch"

1. Detect Tier 3 (Kafka + OpenSearch)
2. Generate 9 stage files in Plan/
3. Return: "Created 9-stage plan with debug container validation."

---

## Key Features

### Session Resumability
- Stage files persist in git (status tracking only)
- All captured values stored in `.env.secrets` (gitignored)
- AI assistants read `.env.secrets` to resume from any stage

### Artifact Storage
- Plan files contain **status only**, not secrets
- All IDs, passwords, URLs stored in `.env.secrets`
- See [artifact-storage.md](reference/artifact-storage.md) for philosophy

### Pause Points
- Each stage ends with verification checklist
- Explicit "proceed to Stage N+1" guidance
- Troubleshooting sections for common failures

---

## Reference Files

- **[build-local-first.md](reference/build-local-first.md)** — Full `doctl app dev build` workflow
- **[artifact-storage.md](reference/artifact-storage.md)** — Storage philosophy, AI instructions

---

## Integration with Other Skills

- **← designer**: Uses `.do/app.yaml` for complexity detection if exists
- **→ deployment**: Plan/ files provide context; Stage 7 creates GitHub Actions workflow
- **→ troubleshooting**: Plan/ stages help identify where failure occurred
- **→ devcontainers**: Local development setup for testing
