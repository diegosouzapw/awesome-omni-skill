---
name: discover
description: Extracts semantic memory from project analysis. Scans codebase, docs, and configs to understand tech stack, constraints, and goals.
stage: 1
---

# Discover Skill (Stage 1)

This skill analyzes existing projects and generates Gastown-compatible semantic memory.

## When to Use

Use this skill when:
- Starting migration of an existing project
- Need to understand a codebase's tech stack
- Want to document project constraints and goals
- Preparing for roadmap generation (Stage 2)

## Output Structure

```
project/
├── .gt/
│   └── memory/
│       ├── semantic.json      # Permanent facts (tech stack, constraints)
│       ├── episodic.json      # Decisions with TTL (optional)
│       └── procedural.json    # Learned patterns (optional)
└── [existing project files]
```

## Discovery Procedure

Scan these locations in priority order:

### 1. Package Files (Tech Stack Detection)

| File | Detects |
|------|---------|
| `package.json` | Node.js runtime, framework, dependencies |
| `Cargo.toml` | Rust projects |
| `go.mod` | Go projects |
| `requirements.txt` | Python dependencies |
| `pyproject.toml` | Python projects (modern) |
| `Gemfile` | Ruby projects |
| `pom.xml` | Java/Maven projects |
| `build.gradle` | Java/Gradle projects |

### 2. Configuration Files (Service Detection)

| File | Detects |
|------|---------|
| `.firebaserc`, `firebase.json` | Firebase |
| `wrangler.toml` | Cloudflare Workers |
| `vercel.json` | Vercel deployment |
| `netlify.toml` | Netlify deployment |
| `docker-compose.yml` | Containerization |
| `Dockerfile` | Container build |
| `*.env.example` | Environment variables |
| `.github/workflows/` | CI/CD (GitHub Actions) |

### 3. Documentation (Project Understanding)

| File | Provides |
|------|----------|
| `README.md` | Project description, setup |
| `docs/` | Architecture docs, ADRs, PRDs |
| `CONTRIBUTING.md` | Development workflow |
| `CHANGELOG.md` | Project history |
| `LICENSE` | License type |

### 4. Source Structure (Codebase Understanding)

| Directory | Indicates |
|-----------|-----------|
| `src/`, `lib/`, `app/` | Main code location |
| `tests/`, `__tests__/`, `spec/` | Test location |
| `schemas/`, `migrations/` | Database schemas |
| `components/` | UI component library |
| `api/`, `routes/` | API structure |

## Tech Stack Extraction

### Framework Detection

Look for these patterns in dependencies:

| Dependency | Framework |
|------------|-----------|
| `next` | Next.js |
| `react` | React |
| `vue` | Vue.js |
| `@angular/core` | Angular |
| `express` | Express.js |
| `fastify` | Fastify |
| `django` | Django |
| `flask` | Flask |
| `fastapi` | FastAPI |
| `rails` | Ruby on Rails |
| `gin-gonic/gin` | Gin (Go) |

### Database Detection

| Indicator | Database |
|-----------|----------|
| `pg`, `postgres` | PostgreSQL |
| `mysql2` | MySQL |
| `mongodb`, `mongoose` | MongoDB |
| `redis` | Redis |
| `prisma` | Prisma ORM |
| `drizzle-orm` | Drizzle ORM |
| `typeorm` | TypeORM |

### Auth Detection

| Indicator | Auth System |
|-----------|-------------|
| `firebase-admin` | Firebase Auth |
| `@auth0/` | Auth0 |
| `next-auth` | NextAuth.js |
| `passport` | Passport.js |
| `clerk` | Clerk |
| `supabase` | Supabase Auth |

## Output: semantic.json

```json
{
  "$schema": "semantic-memory-v1",
  "project": {
    "name": "my-app",
    "type": "web-application",
    "primary_language": "TypeScript",
    "description": "A task management app for teams"
  },
  "tech_stack": {
    "runtime": "Node.js 20",
    "framework": "Next.js 14",
    "database": "Neon PostgreSQL",
    "auth": "Firebase Auth",
    "deployment": "Vercel",
    "styling": "Tailwind CSS",
    "testing": "Vitest",
    "orm": "Drizzle"
  },
  "personas": [
    {"name": "Team Lead", "needs": ["assign tasks", "track progress"]},
    {"name": "Developer", "needs": ["see my tasks", "update status"]}
  ],
  "constraints": [
    "Must support offline mode",
    "GDPR compliant data handling"
  ],
  "non_goals": [
    "Mobile native app (web-only for MVP)",
    "Enterprise SSO (future phase)"
  ],
  "evidence": {
    "last_scan": "2026-01-27T10:00:00Z",
    "files_analyzed": ["package.json", "README.md", "docs/PRD.md"]
  }
}
```

## Memory Types

### Semantic Memory (Required)

Permanent facts that don't change:
- Project name and type
- Primary programming language
- Tech stack components
- Architectural constraints
- Non-goals

### Episodic Memory (Optional)

Decisions with time-to-live (~30 days):
- Architecture decisions
- Library choices with rationale
- Trade-offs made

### Procedural Memory (Optional)

Learned patterns:
- Code conventions
- Testing patterns
- Deployment procedures

## Quality Gates

| Gate | Requirement |
|------|-------------|
| `semantic_valid` | semantic.json is valid JSON |
| `project_identified` | project.name is not null or empty |
| `tech_stack_detected` | At least 2 tech_stack fields populated |
| `evidence_recorded` | evidence.files_analyzed has 1+ entries |

## Validation

```bash
python plugins/lisa/hooks/validate.py --stage discover
```

## Error Handling

If unable to detect something:
- Set field to `null` rather than guessing
- Add to `evidence.unresolved` list (if pattern exists)
- Document what was searched and why it failed

## Next Steps

After discover completes:
- Proceed to Stage 2 (Plan) → `skills/plan/SKILL.md`
- Or proceed directly to Stage 3 (Structure) if roadmap exists
