---
name: senior-backend
description: This skill should be used when the user asks to "design REST APIs", "optimize database queries", "implement authentication", "build microservices", "review backend code", "set up GraphQL", "handle database migrations", or "load test APIs".
---

# Senior Backend Engineer

Backend development patterns, API design, database optimization, and security practices for any language or framework.

---

## Step 1: Detect the Stack

Before writing any code, identify the language and framework from project signals:

| Signal file / pattern | Stack |
|----------------------|-------|
| `package.json` with express/fastify/hono/koa | Node.js |
| `package.json` with nest | NestJS |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `pyproject.toml` / `requirements.txt` + FastAPI/Django/Flask | Python |
| `pom.xml` / `build.gradle` | Java/Spring |
| `*.csproj` / `Program.cs` | .NET/C# |
| `mix.exs` | Elixir/Phoenix |
| `Gemfile` with rails/sinatra | Ruby |

Check: `ls -la`, `cat package.json`, `cat go.mod`, `cat pyproject.toml` — whichever exists.

---

## Step 2: Research Current Practices

Once you know the stack, search for current best practices before making recommendations:

```
WebSearch: "<framework> REST API best practices 2025"
WebSearch: "<framework> authentication JWT best practices 2025"
WebSearch: "<database> query optimization <framework> 2025"
WebSearch: "<framework> microservices patterns 2025"
```

Use the search results to ground your recommendations in current community standards.

---

## Step 3: Universal Backend Principles

These apply regardless of language or framework.

### API Design

- **REST resource naming**: nouns, plural, lowercase (`/users`, `/orders/{id}`)
- **Versioning**: `/api/v1/` prefix or `Accept: application/vnd.api+json;version=1` header
- **Status codes**: 200 OK, 201 Created, 204 No Content, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict, 422 Unprocessable Entity, 429 Too Many Requests, 500 Internal Server Error
- **Pagination**: cursor-based for large/changing datasets, offset-based for small/stable ones
- **Idempotency**: PUT and DELETE must be idempotent; POST uses idempotency keys for critical operations
- **Error shape**: consistent `{ error: { code, message, details } }` across all endpoints

### Database

- **N+1 prevention**: use eager loading / joins — never query in a loop
- **Index strategy**: index foreign keys, columns in WHERE/ORDER BY/GROUP BY; composite indexes for multi-column filters
- **Transactions**: wrap multi-step mutations; use the lowest isolation level that preserves correctness
- **Migrations**: always reversible (up + down); never drop columns in same deploy as code change
- **Connection pooling**: always use a pool; size = (2 x CPU cores) + effective spindle count as starting point

### Authentication and Authorization

- **Password storage**: bcrypt/argon2/scrypt — never MD5, SHA1, or plain SHA256
- **JWT**: short-lived access tokens (15m), long-lived refresh tokens (7d) stored httpOnly cookie; validate `alg`, `iss`, `aud`, `exp`
- **Session tokens**: cryptographically random, 128+ bits, stored server-side
- **RBAC**: check permissions at the service layer, not just the route layer
- **Rate limiting**: per-user and per-IP on auth endpoints; exponential backoff on failures

### Error Handling

- **Never swallow errors**: log with context (user ID, request ID, stack trace)
- **Distinguish errors**: validation (400), auth (401/403), not found (404), conflict (409), server (500)
- **Structured logging**: JSON logs with `level`, `timestamp`, `requestId`, `userId`, `message`, `error`
- **Circuit breakers**: wrap external service calls; fail fast when downstream is degraded

### Security

- **Input validation**: validate at the boundary — type, range, format, length
- **SQL injection**: always use parameterized queries / prepared statements
- **Command injection**: never interpolate user input into shell commands
- **CORS**: explicit allowlist — never `*` in production for credentialed requests
- **Secrets**: environment variables or secret manager — never hardcoded, never in version control
- **Dependencies**: audit regularly (`npm audit`, `cargo audit`, `pip-audit`, `govulncheck`)

### Performance

- **Caching strategy**: cache at the right layer (CDN -> reverse proxy -> app -> DB); set explicit TTLs and cache-control headers
- **Async processing**: move slow work (email, image processing, reports) to a queue
- **Payload size**: paginate large responses; use field selection (GraphQL) or sparse fieldsets (JSON:API)
- **Database reads**: read replicas for reporting queries; connection pooling for all queries

---

## Review Checklist

Before any backend PR:

- [ ] All inputs validated and sanitized at the boundary
- [ ] Auth checked — correct user can do this, others cannot
- [ ] N+1 queries absent — checked with query logging or explain
- [ ] Error paths return correct status codes and structured errors
- [ ] Secrets in env vars, not source
- [ ] DB migrations are reversible
- [ ] Logging includes requestId and userId for traceability
- [ ] Rate limiting on public/auth endpoints
- [ ] No shell command injection surface
- [ ] Dependencies pinned and audited
