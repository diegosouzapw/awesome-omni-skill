---
name: memory-conventions
description:
  This skill should be used when persisting context between sessions, saving project state, loading
  previous session context, or managing longitudinal memory beyond beads issue tracking.
version: 0.1.0
---

# Memory Conventions

This skill defines conventions for managing longitudinal memory and context across sessions. It
establishes when to use different persistence mechanisms (Serena memories, beads notes, git commits)
and how to keep context organized and accessible.

## Persistence Mechanism Selection

### When to Use Serena Memories

**Serena memories** are for knowledge that needs to persist across multiple sessions and is
discovery-based or decision-oriented. Use memories for:

**Design decisions with rationale:**

- "We chose PostgreSQL over MongoDB because our data is highly relational and we need ACID
  guarantees for financial transactions"
- "API versioning strategy: using URL path versioning (/v1/, /v2/) instead of header-based because
  it's more discoverable and cache-friendly"
- "UI component library decision: Material-UI chosen over Chakra for better TypeScript support and
  larger community"

**API contracts and integration points:**

- "Authentication API contract: POST /auth/login expects {email, password}, returns {token,
  refreshToken, expiresIn}"
- "Third-party service integration: Stripe webhook endpoints require signature verification using
  stripe-signature header"
- "GraphQL schema conventions: all mutations return {success, error, data} envelope"

**Architecture choices:**

- "Microservices communication: using message queue (RabbitMQ) for async operations, REST for sync
  queries"
- "Frontend state management: Redux for global app state, React Query for server state, local
  useState for UI-only state"
- "Deployment architecture: containerized services in Kubernetes, PostgreSQL on managed RDS, Redis
  on ElastiCache"

**Discovered constraints and limitations:**

- "Legacy API rate limit: 100 requests/minute per IP, need to implement client-side throttling"
- "Database performance: user_events table becomes slow above 10M rows, need partitioning strategy"
- "Browser compatibility: Safari doesn't support ResizeObserver, must use polyfill"

**Codebase conventions:**

- "Error handling pattern: all async functions return Result<T, AppError> type, never throw
  exceptions"
- "Test file naming: component.test.tsx for unit tests, component.e2e.tsx for end-to-end tests"
- "Import order: external dependencies, internal modules, types, styles (enforced by ESLint)"

**External service configurations:**

- "AWS S3 bucket policy: public-read for marketing assets, private for user uploads with signed
  URLs"
- "CI/CD pipeline stages: lint -> test -> build -> deploy to staging -> manual approval -> deploy to
  production"
- "Monitoring setup: Datadog for metrics, Sentry for error tracking, LogRocket for session replay"

### When to Use Beads Notes

**Beads notes** are for task-specific context that's relevant while the task is active. Use beads
notes for:

**Implementation progress:**

- "Implemented login endpoint, still need to add rate limiting and email verification"
- "Migrated 3 of 5 database tables to new schema, blocked on foreign key constraints"

**Blocking issues:**

- "Can't deploy to staging, waiting for DevOps to provision SSL certificate"
- "Unit tests failing intermittently, suspected race condition in async test setup"

**Work-in-progress notes:**

- "Refactoring auth module: extracted token generation to separate service, next step is to update
  all call sites"
- "Performance optimization: reduced API response time from 800ms to 200ms by adding database index,
  still need to add caching layer"

**Task-specific context that won't be needed after completion:**

- "Testing locally with user ID 12345, has full dataset for reproduction"
- "Temporary workaround: hardcoded API endpoint to staging while DNS propagates"

### When to Use Git Commits

**Git commits** are for code changes with descriptive messages. Every code change must have a
commit. Commits should:

**Document what changed and why:**

```text
feat(auth): add password reset flow

Implements password reset via email token. Users receive a link
valid for 1 hour. Includes rate limiting (3 requests per hour)
to prevent abuse.

Closes #142
```

**Reference related issues/tasks:**

- Use "Closes #123" or "Fixes #123" for issues being resolved
- Use "Related to #123" for connected but not resolved issues
- Use "Part of #123" for work contributing to larger tasks

**Capture technical details:**

```text
perf(db): add composite index on user_events table

Query performance improved from 2.3s to 45ms for date range
queries. Index covers (user_id, event_type, created_at) which
matches our most common query pattern.

Before: sequential scan on 8.5M rows
After: index scan on ~10k rows per typical query
```

## Memory File Naming

**Use descriptive, kebab-case names** that clearly indicate the topic and scope:

- `api-authentication-design.md` (not `auth.md` or `api_auth.txt`)
- `database-schema-decisions.md` (not `db.md` or `schema_notes.md`)
- `frontend-routing-architecture.md` (not `routes.md` or `FrontendRoutes.md`)
- `stripe-webhook-integration.md` (not `stripe.md` or `webhooks.md`)
- `performance-optimization-results.md` (not `perf.md` or `optimization.md`)

**Include scope and topic in the name:**

- Good: `user-service-api-contract.md` (scope: user-service, topic: API contract)
- Bad: `contract.md` (what contract? which service?)

**Avoid abbreviations unless universally understood:**

- Good: `api-rate-limiting-strategy.md`
- Bad: `api-rl-strat.md`

**Use version suffixes only when maintaining historical records:**

- `database-schema-v2.md` (current version)
- `database-schema-v1-deprecated.md` (archived old version)

## What to Persist

### Persist These Types of Information

**Design decisions with rationale:**

Record not just what was decided, but why. Future sessions (and future developers) need to
understand the trade-offs:

```markdown
# Decision: GraphQL over REST for mobile API

## Context

Mobile app needs to fetch user profile, posts, and comments in a single request to minimize latency
on slow connections.

## Options Considered

1. REST with multiple endpoints (3 requests)
2. REST with composite endpoint (1 request, over-fetching)
3. GraphQL (1 request, precise data)

## Decision

Chose GraphQL. Mobile app can specify exactly what data it needs, reducing payload size by ~60% in
typical cases.

## Trade-offs

- Pro: Reduced network payload, better mobile performance
- Pro: Self-documenting schema with GraphQL introspection
- Con: Increased backend complexity (GraphQL server setup)
- Con: Need to implement N+1 query protection (using DataLoader)
```

**API contracts:**

Document expected request/response formats, especially for external integrations:

```markdown
# Stripe Webhook Integration

## Endpoint

POST /webhooks/stripe

## Headers

- stripe-signature: HMAC signature for verification

## Payload

Standard Stripe event object

## Response

200 OK with empty body (Stripe ignores response body)

## Error Handling

- 400 for invalid signature (Stripe will retry)
- 500 for processing errors (Stripe will retry)
```

**Architecture choices:**

Capture the big picture decisions that affect how the system works:

```markdown
# State Management Architecture

## Global State (Redux)

- User authentication/authorization
- Application-wide settings
- UI theme and preferences

## Server State (React Query)

- API data fetching/caching
- Optimistic updates
- Background refetching

## Local State (useState/useReducer)

- Form inputs
- UI-only state (modals, dropdowns)
- Component-specific state
```

**Discovered constraints:**

Document limits, quirks, and gotchas discovered through experimentation:

```markdown
# AWS Lambda Constraints

## Memory/CPU

- Memory range: 128 MB to 10,240 MB
- CPU scales linearly with memory (1,769 MB = 1 vCPU)
- Our image processing needs 3GB minimum for reliable performance

## Execution Time

- Max execution: 15 minutes
- Our video processing can take 10-12 minutes for 1080p
- Need to split into chunks or use Step Functions for longer videos

## Cold Start

- Cold start with 3GB memory: ~2-3 seconds
- Warm invocations: ~100ms
- Using provisioned concurrency (5 instances) for API endpoints
```

**Performance baselines:**

Record performance measurements to track improvements/regressions:

```markdown
# API Performance Baselines (2026-02-08)

## User Profile Endpoint

- p50: 120ms
- p95: 280ms
- p99: 450ms

## Search Endpoint

- p50: 350ms
- p95: 1200ms
- p99: 2400ms (needs optimization)

## Database Queries

- Most common query (user posts): 45ms avg with index
- Slowest query (analytics aggregation): 1.8s (runs async)
```

### What NOT to Persist

**Session-specific temporary data:**

Don't save information that's only relevant to the current session:

- "I'm currently looking at the login component" (ephemeral)
- "Found 3 TypeScript errors in auth module" (temporary state)
- "User asked me to check the database" (session context)

**File listings and search results:**

Don't save output from searches or file system operations:

- Directory tree listings
- Grep search results
- File content dumps

These can be regenerated on demand and become stale quickly.

**Transient state:**

Don't save things that change frequently or are easily derived:

- Current git branch (changes constantly)
- Number of open PRs (changes constantly)
- Package versions (check package.json instead)

## Session Start Protocol

At the beginning of each session, follow this protocol to load relevant context:

### 1. Check for Existing Memories

```bash
# List all memories to see what context exists
list_memories
```

Review memory titles to identify relevant ones for the current work.

### 2. Read Relevant Memories

```bash
# Read specific memories related to the current task
read_memory("api-authentication-design")
read_memory("database-schema-decisions")
```

Don't read all memories indiscriminately. Be selective based on the task.

### 3. Check Beads Context

```bash
# Check for ready tasks
bd ready

# Check in-progress tasks
bd list --status=in_progress

# View specific task details if needed
bd show <task-id>
```

This gives you the current work context and any blocking issues.

### 4. Synthesize Context

Briefly summarize what you learned:

"I've loaded context from memories: using JWT auth with refresh tokens, PostgreSQL database with
schema version 2. Beads shows task #42 in progress (implement password reset). Previous session got
blocked on email service configuration."

Then proceed with the work, informed by this context.

## Memory Management

### When to Update vs Create New

**Update existing memory when:**

- The topic is the same, information has changed or evolved
- Adding new details to an existing decision
- Correcting or refining previous information
- Appending new discoveries to existing topic

Example: If `api-authentication-design.md` describes JWT auth, and you add refresh token rotation,
update the existing memory rather than creating `api-authentication-refresh-tokens.md`.

**Create new memory when:**

- Addressing a genuinely new topic not covered in existing memories
- Creating a distinct decision document for a different area
- Starting a new epic/project area
- Documenting a different integration or service

Example: If you have `user-service-api-contract.md` and now need to document the payment service
API, create `payment-service-api-contract.md` as a separate memory.

### Memory Hygiene

**Delete obsolete memories when explicitly asked.** Don't proactively delete memories unless:

1. User explicitly asks to remove outdated information
2. Information is completely superseded (e.g., "v1 API" when v2 is fully deployed and v1 is gone)

When deleting, consider archiving instead:

```markdown
# DEPRECATED: GraphQL API Design (v1)

This approach was replaced by REST API in Feb 2026. See `rest-api-design.md` for current
implementation.

[Original content kept for historical reference...]
```

**Keep memories focused and current:**

- Review and update memories when you notice outdated information
- Split large, unfocused memories into topic-specific ones
- Remove implementation details that belong in code comments
- Keep architectural decisions, remove tactical step-by-step instructions

**Avoid memory bloat:**

- Don't save every small decision
- Focus on decisions that affect multiple sessions or multiple people
- Let beads handle task-specific temporary context
- Let git commits handle code change history

## Examples

### Good Memory: Design Decision

```markdown
# API Error Handling Strategy

## Decision

All API endpoints return consistent error format with HTTP status codes and structured error
objects.

## Error Response Format

{ "error": { "code": "VALIDATION_ERROR", "message": "User-friendly error message", "details": [
{"field": "email", "message": "Invalid email format"} ] } }

## HTTP Status Codes

- 400: Client errors (validation, malformed requests)
- 401: Authentication required
- 403: Authenticated but not authorized
- 404: Resource not found
- 409: Conflict (duplicate resource)
- 422: Unprocessable entity (business logic error)
- 429: Rate limit exceeded
- 500: Server error (logged but not exposed to client)

## Error Codes

Standardized error codes in SCREAMING_SNAKE_CASE:

- VALIDATION_ERROR
- AUTHENTICATION_REQUIRED
- PERMISSION_DENIED
- RESOURCE_NOT_FOUND
- etc.

## Rationale

Consistent error handling makes client integration easier and reduces support burden. Structured
errors allow clients to programmatically handle specific error cases.
```

### Good Beads Note

```markdown
Task #42: Implement password reset flow

**Progress:**

- Created email template
- Implemented token generation (1-hour expiry)
- Added database table for reset tokens

**Next:**

- Add rate limiting (3 requests/hour per email)
- Write integration tests
- Update API documentation

**Blockers:**

- Need SMTP credentials for staging environment
- Asked DevOps in Slack #infrastructure
```

### Good Git Commit

```text
feat(auth): implement password reset flow

Adds password reset via email token with 1-hour expiry.

Implementation:
- POST /auth/reset-password/request sends email with token
- POST /auth/reset-password/confirm validates token and updates password
- Rate limiting: 3 requests per hour per email address

Security:
- Tokens are cryptographically random (32 bytes)
- Tokens hashed before storage (SHA-256)
- Old tokens invalidated when new one requested

Closes #142
```

## Summary

Use the right tool for the right job:

- **Memories**: Long-lived knowledge about design, architecture, constraints, decisions
- **Beads notes**: Task-specific, session-relevant, work-in-progress context
- **Git commits**: Code changes with descriptive messages and rationale

Keep context organized, accessible, and current. Don't over-persist or under-persist. Find the
balance.
