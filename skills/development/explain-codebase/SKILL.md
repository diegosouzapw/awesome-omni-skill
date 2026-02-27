---
name: explain-codebase
description: Generates a comprehensive overview of a codebase for onboarding, knowledge transfer, or architecture understanding. Use when the user says "explain this project", "how does this codebase work?", "onboard me", "give me an overview", "I'm new to this project", "walk me through the architecture", or "map this codebase".
allowed-tools: Read, Grep, Glob, Bash
---

# Explain Codebase Skill

When explaining a codebase, follow this structured process. The goal is to give someone everything they need to understand the project and start contributing confidently.

## 1. Discovery — Scan the Project

Before explaining anything, gather information:

### Project Identity
```bash
# What is this project?
cat README.md 2>/dev/null
cat package.json 2>/dev/null | grep -E '"name"|"description"|"version"'
cat pyproject.toml 2>/dev/null | head -20
cat Cargo.toml 2>/dev/null | head -20
cat go.mod 2>/dev/null | head -5
cat pom.xml 2>/dev/null | head -30
cat Gemfile 2>/dev/null | head -10
cat composer.json 2>/dev/null | grep -E '"name"|"description"'

# What does the directory structure look like?
find . -maxdepth 2 -type d ! -path '*/node_modules/*' ! -path '*/.git/*' ! -path '*/vendor/*' ! -path '*/__pycache__/*' ! -path '*/dist/*' ! -path '*/build/*' | sort
```

### Tech Stack Detection
```bash
# Languages — count lines by file type
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" -o -name "*.go" -o -name "*.rs" -o -name "*.java" -o -name "*.rb" -o -name "*.php" -o -name "*.swift" -o -name "*.kt" \) ! -path '*/node_modules/*' ! -path '*/vendor/*' | sed 's/.*\.//' | sort | uniq -c | sort -rn

# Framework detection
cat package.json 2>/dev/null | grep -E "react|next|vue|nuxt|angular|express|fastify|nest|svelte|remix|astro"
cat requirements.txt pyproject.toml 2>/dev/null | grep -E "django|flask|fastapi|celery|sqlalchemy"
cat Gemfile 2>/dev/null | grep -E "rails|sinatra|sidekiq"
cat go.mod 2>/dev/null | grep -E "gin|echo|fiber|chi"
cat pom.xml build.gradle 2>/dev/null | grep -E "spring|quarkus|micronaut"

# Database
cat docker-compose.yml 2>/dev/null | grep -E "postgres|mysql|mongo|redis|elasticsearch"
grep -rn "DATABASE_URL\|MONGO_URI\|REDIS_URL" .env.example .env.sample 2>/dev/null

# Infrastructure
ls Dockerfile docker-compose.yml k8s/ terraform/ .github/workflows/ .circleci/ 2>/dev/null
```

### Codebase Size
```bash
# File count and lines of code (excluding deps and build)
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" -o -name "*.go" -o -name "*.rs" -o -name "*.java" -o -name "*.rb" -o -name "*.php" \) ! -path '*/node_modules/*' ! -path '*/vendor/*' ! -path '*/dist/*' ! -path '*/build/*' | wc -l

# Lines of code
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" -o -name "*.go" \) ! -path '*/node_modules/*' ! -path '*/dist/*' -exec cat {} + | wc -l

# Test file count
find . -type f \( -name "*.test.*" -o -name "*.spec.*" -o -name "test_*" -o -name "*_test.*" \) ! -path '*/node_modules/*' | wc -l
```

## 2. The 30-Second Overview

Start with a concise summary anyone can understand:
```
[Project Name] is a [type of application] that [what it does] for [who uses it].

Built with [primary tech stack], it [key capability 1], [key capability 2], 
and [key capability 3].

The codebase is [size: small/medium/large] with approximately [X] files 
and [X]k lines of code.
```

Example:
```
Acme Dashboard is a web application that provides real-time analytics 
and reporting for e-commerce merchants.

Built with Next.js, PostgreSQL, and Redis, it processes order data, 
generates sales reports, and sends automated alerts when metrics 
fall outside configured thresholds.

The codebase is medium-sized with approximately 320 source files 
and 45k lines of TypeScript.
```

## 3. Architecture Overview

### System Diagram

Always provide an ASCII diagram showing the major components:
```
┌─────────────────────────────────────────────────────────┐
│                        CLIENT                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │  Pages    │  │Components│  │  Hooks   │              │
│  │ (Routes)  │──│  (UI)    │──│ (State)  │              │
│  └────┬─────┘  └──────────┘  └────┬─────┘              │
│       │                           │                      │
│       └───────────┬───────────────┘                      │
│                   │ API Calls                            │
└───────────────────┼──────────────────────────────────────┘
                    │
┌───────────────────┼──────────────────────────────────────┐
│                   │        SERVER                         │
│  ┌────────────────▼───────────────┐                      │
│  │        API Layer               │                      │
│  │   (Routes / Controllers)       │                      │
│  └────────────────┬───────────────┘                      │
│                   │                                      │
│  ┌────────────────▼───────────────┐                      │
│  │       Service Layer            │                      │
│  │    (Business Logic)            │                      │
│  └───┬────────────┬───────────┬──┘                      │
│      │            │           │                           │
│  ┌───▼───┐  ┌────▼────┐  ┌──▼──────┐                   │
│  │  DB   │  │  Cache  │  │ External│                    │
│  │(Postgres)│ │(Redis) │  │  APIs   │                   │
│  └───────┘  └─────────┘  └─────────┘                    │
└──────────────────────────────────────────────────────────┘
```

### Directory Map

Explain what each top-level directory contains and WHY it's organized that way:
```
project/
├── src/
│   ├── api/          → HTTP route handlers. Validates input, calls services, returns responses.
│   │                    Does NOT contain business logic.
│   ├── services/     → Core business logic. Orchestrates operations across repositories
│   │                    and external services. This is where the "rules" live.
│   ├── db/           → Database access layer. Queries, models, migrations.
│   │                    Only layer that imports the ORM.
│   ├── models/       → TypeScript interfaces and types shared across layers.
│   │                    No runtime code — only type definitions.
│   ├── middleware/    → Express/Fastify middleware: auth, logging, rate limiting, error handling.
│   ├── utils/        → Pure utility functions. No side effects, no external dependencies.
│   ├── config/       → Configuration loading and validation. Reads from env vars.
│   ├── jobs/         → Background job definitions (queue workers, cron jobs).
│   └── lib/          → Third-party service wrappers (Stripe, SendGrid, S3).
│                        Isolates external dependencies behind clean interfaces.
├── tests/
│   ├── unit/         → Fast tests for individual functions and classes.
│   ├── integration/  → Tests that hit the database or external services.
│   └── e2e/          → End-to-end tests simulating real user flows.
├── scripts/          → Developer tooling: seed data, migrations, deploy scripts.
├── docs/             → Architecture decisions, API docs, onboarding guides.
├── infra/            → Infrastructure as code: Terraform, Kubernetes, Docker.
└── .github/          → CI/CD workflows, PR templates, issue templates.
```

## 4. Key Concepts & Domain Model

Explain the core business domain:

### Domain Entities
Identify and explain the main entities/models:
- What are they?
- How do they relate to each other?
- Where are they defined in code?
```
┌──────────┐     ┌──────────┐     ┌──────────┐
│   User   │────▶│  Order   │────▶│  Item    │
│          │ 1:N │          │ 1:N │          │
└──────────┘     └────┬─────┘     └──────────┘
                      │ 1:1
                 ┌────▼─────┐
                 │ Payment  │
                 └──────────┘
```

### Key Business Rules
List the non-obvious rules that drive behavior:
- "Orders over $100 get free shipping"
- "Users can only cancel within 24 hours"
- "Inventory is reserved at checkout, released after 30 minutes if unpaid"

### Glossary
Define project-specific terms:
| Term | Meaning | Where in Code |
|------|---------|---------------|
| Merchant | A business using our platform | `src/models/merchant.ts` |
| Fulfillment | Process of shipping an order | `src/services/fulfillment/` |
| Webhook | Incoming event from payment provider | `src/api/webhooks/` |

## 5. Critical Paths

Trace the most important user flows through the code:

### Example: User Login Flow
```
1. Client: POST /api/auth/login { email, password }
   → src/api/routes/auth.ts:login()

2. Controller validates input
   → src/api/validators/auth.ts:loginSchema

3. Calls AuthService.login(email, password)
   → src/services/auth.ts:login()

4. AuthService looks up user by email
   → src/db/repositories/user.ts:findByEmail()

5. Compares password hash (bcrypt)
   → src/services/auth.ts:verifyPassword()

6. Generates JWT token
   → src/lib/jwt.ts:generateToken()

7. Creates session record
   → src/db/repositories/session.ts:create()

8. Returns token to client
   → { token: "eyJ...", expiresIn: 3600 }
```

### Identify and Trace These Flows
- **Authentication** — signup, login, logout, password reset
- **Core feature #1** — the main thing the app does
- **Core feature #2** — the second most important feature
- **Payment flow** — if applicable
- **Data ingestion** — how data gets into the system
- **Notification flow** — how users get notified

## 6. Data Flow & State Management

### Where Data Lives
| Data | Storage | Location in Code | Notes |
|------|---------|------------------|-------|
| User accounts | PostgreSQL | `src/db/models/user.ts` | Source of truth |
| Sessions | Redis | `src/lib/session.ts` | 24hr TTL |
| Cart | Client state | `src/store/cart.ts` | Lost on page close |
| Product images | S3 | `src/lib/storage.ts` | CDN cached |
| Search index | Elasticsearch | `src/lib/search.ts` | Rebuilt nightly |

### How Data Moves
```
User Action → API Request → Validation → Service Logic → Database
                                              │
                                              ├── Cache Update
                                              ├── Event Emission
                                              └── Notification
```

### State Management (Frontend)
- What state management solution is used (Redux, Zustand, Pinia, Context, etc.)?
- Where is global state defined?
- How does data flow from API to UI?
- Are there any real-time features (WebSocket, SSE)?

## 7. External Dependencies & Integrations

### Third-Party Services
| Service | Purpose | Wrapper Location | Docs |
|---------|---------|-----------------|------|
| Stripe | Payments | `src/lib/stripe.ts` | stripe.com/docs |
| SendGrid | Email | `src/lib/email.ts` | sendgrid.com/docs |
| AWS S3 | File storage | `src/lib/storage.ts` | aws.amazon.com/s3 |
| Sentry | Error tracking | `src/lib/sentry.ts` | sentry.io/docs |

### Internal Services (if microservices)
| Service | Communicates Via | Purpose |
|---------|-----------------|---------|
| Auth Service | REST API | User authentication |
| Notification Service | Message Queue | Email/SMS/Push |
| Analytics Service | Event Stream | Usage tracking |

## 8. Configuration & Environment

### Environment Variables
| Variable | Purpose | Required | Example |
|----------|---------|----------|---------|
| DATABASE_URL | PostgreSQL connection | Yes | postgres://... |
| REDIS_URL | Redis connection | Yes | redis://... |
| STRIPE_SECRET_KEY | Payment processing | Yes | sk_live_... |
| JWT_SECRET | Token signing | Yes | random-256-bit |
| PORT | Server port | No | 3000 |

### Environment Differences
| Feature | Development | Staging | Production |
|---------|------------|---------|------------|
| Database | Local Postgres | Shared RDS | Dedicated RDS |
| Email | Console output | SendGrid sandbox | SendGrid live |
| Payments | Stripe test mode | Stripe test mode | Stripe live |
| Logging | Debug level | Info level | Warn level |

## 9. Development Workflow

### Getting Started
```bash
# Prerequisites
node --version  # >= 18
docker --version  # for local DB

# Setup
git clone <repo>
cp .env.example .env
npm install
docker-compose up -d  # starts Postgres + Redis
npm run db:migrate
npm run db:seed
npm run dev
```

### Common Tasks
| Task | Command | Notes |
|------|---------|-------|
| Run dev server | `npm run dev` | Hot reload on port 3000 |
| Run tests | `npm test` | Unit tests only |
| Run all tests | `npm run test:all` | Unit + integration + e2e |
| Create migration | `npm run db:migrate:create` | Creates empty migration file |
| Lint | `npm run lint` | ESLint + Prettier |
| Build | `npm run build` | Production build to dist/ |
| Deploy | `npm run deploy` | Deploys to staging |

### Git Workflow
- Branch naming: `feature/`, `fix/`, `chore/`
- PR process: draft → review → merge to main
- CI runs: lint, test, build, security scan
- Deploy: auto-deploy to staging on merge, manual promote to production

## 10. Known Gotchas & Tribal Knowledge

List things that aren't obvious from reading the code:

### Common Pitfalls
- "The `OrderService.create()` method is NOT idempotent — calling it twice creates duplicate orders. Always check for existing orders first."
- "Redis keys use the prefix `app:` in production but no prefix in development. This has caused data collisions before."
- "The user migration from 2023 left some accounts with null emails. Always handle this case."

### Historical Context
- "We migrated from MongoDB to PostgreSQL in Q2 2024. Some older code still has Mongo-style patterns."
- "The `legacy/` folder contains the old API that's still used by mobile app versions < 3.0. Do not modify."
- "Rate limiting was added after the January 2024 incident. See ADR-023."

### Areas of Tech Debt
- "The `UserService` is 800 lines and needs to be split into AuthService and ProfileService"
- "Tests in `tests/integration/orders.test.ts` are flaky due to timing issues with the queue"
- "The search indexer runs synchronously — it blocks the API for ~2 seconds on large catalog updates"

## 11. Where to Look When...

Quick reference for common tasks:

| I Need To... | Look At |
|--------------|---------|
| Add a new API endpoint | `src/api/routes/` and copy an existing route |
| Add a new database table | `src/db/migrations/` and `src/db/models/` |
| Change business logic | `src/services/` — find the relevant service |
| Add a new background job | `src/jobs/` — follow the existing pattern |
| Fix a UI component | `src/components/` — organized by feature |
| Update email templates | `src/templates/email/` |
| Change environment config | `.env` and `src/config/` |
| Debug a failing test | `tests/` — mirrors the src/ structure |
| Understand a past decision | `docs/adr/` — architecture decision records |
| Check deployment config | `.github/workflows/` or `infra/` |

## Output Format

Structure the explanation as:

1. **30-Second Overview** — what it is, who it's for, what stack
2. **Architecture Diagram** — ASCII visual of major components
3. **Directory Map** — what each folder does and why
4. **Domain Model** — core entities and their relationships
5. **Critical Paths** — trace 2-3 key user flows through the code
6. **Data Flow** — where data lives and how it moves
7. **External Dependencies** — third-party services and integrations
8. **Getting Started** — how to set up and run locally
9. **Gotchas** — tribal knowledge and common pitfalls
10. **Quick Reference** — where to look for common tasks

## Adaptation Rules

- **Small project (< 50 files)**: Skip sections 6-8, keep it brief
- **Monorepo**: Explain each package/app separately, then show how they connect
- **Microservices**: Start with the service map, then drill into the requested service
- **Frontend-only**: Focus on component hierarchy, state management, and routing
- **Backend-only**: Focus on API structure, database schema, and job processing
- **New contributor**: Emphasize getting started, directory map, and "where to look"
- **Architecture review**: Emphasize design decisions, trade-offs, and tech debt

## Summary

End every codebase explanation with:
1. **Quick start command** — the one command to get running
2. **First file to read** — the best entry point for understanding the code
3. **Key files** — 5-10 most important files to understand
4. **Suggested first task** — a small, safe task to build familiarity
5. **Who to ask** — point of contact or team channel for questions
