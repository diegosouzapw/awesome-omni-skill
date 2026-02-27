---
name: backend-development
description: Build robust backend systems with modern technologies (Node.js, Python, Go, Rust), frameworks (NestJS, FastAPI, Django), databases (PostgreSQL, MongoDB, Redis), APIs (REST, GraphQL, gRPC), authentication (OAuth 2.1, JWT), testing strategies, security best practices (OWASP Top 10), performance optimization, scalability patterns (microservices, caching, sharding), DevOps practices (Docker, Kubernetes, CI/CD), and monitoring. Use when designing APIs, implementing authentication, optimizing database queries, setting up CI/CD pipelines, handling security vulnerabilities, building microservices, or developing production-ready backend systems.
license: MIT
version: 2.0.0
---

# Backend Development Skill

Production-ready backend development with 2025 best practices and code patterns.

## Quick Decision Matrix

| Need | Choose |
|------|--------|
| Fast development | Node.js + NestJS |
| Data/ML/AI | Python + FastAPI |
| High concurrency | Go + Gin |
| Max performance | Rust + Axum |
| Edge/Serverless | Hono / Elysia |
| ACID transactions | PostgreSQL 17 |
| Flexible schema | MongoDB |
| Caching/Sessions | Redis |
| Internal APIs | gRPC |
| Public APIs | REST / GraphQL |
| Type-safe fullstack | tRPC |

## Implementation Checklist

```
[ ] API Design (references/api-design.md)
[ ] Authentication (references/authentication.md)
[ ] Database setup (references/databases.md)
[ ] Security hardening (references/security.md)
[ ] Observability (references/observability.md)
[ ] Testing strategy (references/testing.md)
[ ] DevOps/CI-CD (references/devops.md)
```

## 2025 Key Updates

| Area | 2025 Best Practice |
|------|-------------------|
| Auth | OAuth 2.1 + PKCE mandatory, Passkeys/WebAuthn |
| Security | OWASP 2025, Zero Trust Architecture |
| DB | PostgreSQL 17, pgvector for AI, serverless DBs |
| Observability | OpenTelemetry standard |
| Frameworks | Hono, Elysia for edge; Effect-TS for FP |
| AI Integration | LLMs as probabilistic components |

## Core Patterns (2025)

**API:** REST (CRUD) → GraphQL (complex) → gRPC (microservices) → tRPC (TypeScript)

**Auth:** OAuth 2.1 + PKCE → Passkeys → JWT (short-lived) → Refresh rotation

**DB:** Connection pooling → Indexing → Read replicas → Sharding

**Security:** Zero Trust → Input validation → Parameterized queries → Rate limiting

**Testing:** Unit 70% → Integration 20% → E2E 10% → Contract tests

## Reference Navigation

**API & Architecture:**
- `references/api-design.md` - REST, GraphQL, gRPC, tRPC patterns
- `references/architecture.md` - Microservices, serverless, event-driven

**Security & Auth:**
- `references/security.md` - OWASP 2025, Zero Trust, input validation
- `references/authentication.md` - OAuth 2.1, Passkeys, JWT, sessions

**Data & Performance:**
- `references/databases.md` - PostgreSQL, MongoDB, Redis patterns
- `references/performance.md` - Caching, optimization, scaling

**Quality & Operations:**
- `references/testing.md` - Testing pyramid, frameworks, CI integration
- `references/devops.md` - Docker, K8s, CI/CD, monitoring
- `references/observability.md` - OpenTelemetry, tracing, logging

**Code Patterns:**
- `references/code-patterns.md` - Latest approach snippets for planners

## Resources

- OWASP Top 10 2025: https://owasp.org/Top10/
- OAuth 2.1: https://oauth.net/2.1/
- OpenTelemetry: https://opentelemetry.io/
- PostgreSQL 17: https://www.postgresql.org/docs/17/
