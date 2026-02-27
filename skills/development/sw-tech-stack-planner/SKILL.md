---
name: sw-tech-stack-planner
description: Use when user wants a tech stack recommendation, technology choices, docker-compose setup, or architecture decisions for a software project – reads vision.md, user-stories.md, use-cases.md and generates requirements/tech-stack.yaml silently.
version: 1.0.0
author: Lehnert
---

# SW Tech Stack Planner

## Overview

Reads all available requirements files and recommends the best modern, production-ready tech stack for the project. Outputs a clean YAML file at `requirements/tech-stack.yaml`. Operates silently – no file content shown in the chat.

**Language detection:** Read the user's input language and respond in that language. Default to German if undetectable.

---

## Required Input – Check in Order

Read all files that exist and combine the information:

1. `requirements/vision.md` – project type, complexity, target audience
2. `requirements/user-stories.md` – feature scope, integrations needed
3. `requirements/use-cases.md` – flows, actor interactions, edge cases

If none exist and no input is given, stop and respond:

> No requirements files found.
> Run `/sw-idea-analyzer` first, or describe your project directly:
> `/sw-tech-stack-planner <your description>`

---

## Stack Selection Rules

### Always Docker-First

Every recommendation must include:
- Full `docker-compose.yml` ready to run with `docker compose up`
- All services (database, cache, backend, frontend) in containers
- Named volumes for data persistence
- Environment variables via `.env` file (never hardcoded)
- Health checks on database services

### Database

| Project type | Default choice |
|-------------|----------------|
| Relational data, complex queries | PostgreSQL |
| Simple relational, small project | MySQL / MariaDB |
| Cache, sessions, queues | Redis (always add if backend has async work) |
| Document / flexible schema | MongoDB |
| Multi-model / graph | Neo4j or ArangoDB |

Always include **Testcontainers** when the language/framework supports it (Java, Kotlin, Go, .NET, Node.js).

### Frontend

| Signal from requirements | Recommendation |
|--------------------------|----------------|
| Complex UI, dashboard, SPA | Next.js 15 (App Router) + TypeScript |
| Static / content site | Astro or Next.js static export |
| Mobile app | React Native + Expo |
| Admin panel only | shadcn/ui + Next.js |
| Simple form / landing page | Next.js or plain HTML |

### Backend

| Signal from requirements | Recommendation |
|--------------------------|----------------|
| REST API, moderate complexity | NestJS (Node.js) or FastAPI (Python) |
| High performance, concurrency | Go (Gin / Fiber) or Rust (Axum) |
| Enterprise / Java shop | Spring Boot 3 |
| Full-stack in one repo | Next.js API routes or tRPC |
| Real-time / websockets | NestJS + Socket.io |
| Data-heavy / ML pipeline | FastAPI + Celery |

### Auth

| Requirement | Choice |
|------------|--------|
| SaaS, social login, managed | Clerk or Auth.js |
| Self-hosted, full control | Keycloak (Docker) |
| Simple JWT, own users | NestJS JWT + Passport or Spring Security |

### Testing

Always recommend a layered testing strategy:
- **Unit:** Jest / Vitest (TS), pytest (Python), JUnit 5 (Java)
- **Integration:** Testcontainers (real DB in Docker per test run)
- **E2E:** Playwright
- **API:** Supertest (NestJS) or httpx (FastAPI)

### Deployment

| Scale | Choice |
|-------|--------|
| Small / hobby | Coolify (self-hosted) or Railway |
| Production / team | Docker Compose on VPS + Traefik reverse proxy |
| Kubernetes scale | Helm chart on k3s or managed K8s |
| Serverless preference | Vercel (frontend) + Supabase (backend) |

---

## Output Format

Write `requirements/tech-stack.yaml` with these exact sections:

```yaml
project:
  name: ""
  type: ""           # web-app | api | mobile | cli | fullstack
  complexity: ""     # simple | medium | complex

frontend:
  framework: ""
  language: ""
  ui_library: ""
  state_management: ""
  notes: ""

backend:
  framework: ""
  language: ""
  runtime: ""
  notes: ""

database:
  primary: ""
  cache: ""
  orm: ""
  migrations: ""

auth:
  solution: ""
  strategy: ""       # jwt | session | oauth
  notes: ""

testing:
  unit: ""
  integration: ""
  e2e: ""
  api: ""
  testcontainers: true | false

deployment:
  strategy: ""
  reverse_proxy: ""
  ci_cd: ""
  registry: ""

docker:
  compose_version: "3.9"
  services: []       # list all service names
  volumes: []
  networks: []

rationale:
  frontend: ""       # 1-2 sentences why this choice
  backend: ""
  database: ""
  auth: ""
  deployment: ""
```

---

## Output Rules

- **Never output the YAML or any file content in the chat.**
- Do not show tech stack details, code blocks, or configuration in the chat response.
- The only chat output allowed is the confirmation lines.
- Use the user's language in the `rationale` field values.

---

## File Output

**Steps:**
1. If `requirements/` does not exist: `mkdir -p requirements`
2. Analyze all available inputs and select the best stack
3. Write the complete YAML to `requirements/tech-stack.yaml`
4. Print **only** these lines to the chat:

> ✅ File created: `requirements/tech-stack.yaml`
> This is now the official tech stack for all following coding skills.

One optional summary sentence max (e.g. "Recommended Next.js 15 + NestJS + PostgreSQL + Docker stack with Testcontainers."). No YAML, no code blocks, no full output.

---

## Next Skill

> ▶ **Next steps:**
> - Run `/sw-use-case-coder UC-001` to generate production-ready code for a specific use case
> - Run `/sw-use-case-coder all MVP` to scaffold all MVP use cases at once
