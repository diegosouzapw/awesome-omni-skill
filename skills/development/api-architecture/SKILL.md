---
name: api-architecture
description: API versioning, security, authentication, rate limiting, monitoring, error handling, and documentation strategies for production APIs. Use when planning API infrastructure, implementing security concerns, or designing monitoring strategies.
version: 1.0.0
---

# API Architecture

Master architectural concerns that apply across all API types for production-ready systems.

## When to Use This Skill

- Planning API versioning strategies
- Implementing authentication and authorization
- Setting up rate limiting and quotas
- Designing error handling strategies
- Planning monitoring and observability
- Generating API documentation
- Securing APIs against attacks
- Implementing caching strategies

## API Versioning

### URL Versioning (Recommended)

Clear and easy to route: `/api/v1/users`, `/api/v2/users`

### Header Versioning

Clean URLs: `Accept: application/vnd.api+json; version=2`

### Query Parameter Versioning

Easy to test: `GET /api/users?version=2`

## Security Best Practices

### Authentication

**Bearer Token (JWT):**
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
401 Unauthorized - Missing/invalid token
403 Forbidden - Valid token, insufficient permissions
```

**API Keys:**
```
X-API-Key: your-api-key-here
```

### Authorization Patterns

- Role-based access control (RBAC)
- Attribute-based access control (ABAC)
- Scope-based permissions (OAuth 2.0)

### Input Validation

- Validate all input at API boundaries
- Use schemas for validation (Pydantic, JSON Schema)
- Sanitize user input to prevent injection attacks
- Validate file uploads

## Rate Limiting

### Implementation Pattern

```python
class RateLimiter:
    def __init__(self, calls: int, period: int):
        self.calls = calls
        self.period = period
        self.cache = {}

    def check(self, key: str) -> bool:
        now = datetime.now()
        if key not in self.cache:
            self.cache[key] = []

        self.cache[key] = [
            ts for ts in self.cache[key]
            if now - ts < timedelta(seconds=self.period)
        ]

        if len(self.cache[key]) >= self.calls:
            return False

        self.cache[key].append(now)
        return True
```

### Headers

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 742
X-RateLimit-Reset: 1640000000

429 Too Many Requests
Retry-After: 3600
```

## Monitoring and Observability

### Health Check Endpoints

```python
@app.get("/health")
async def health_check():
    return {"status": "healthy", "version": "1.0.0"}

@app.get("/health/detailed")
async def detailed_health():
    return {
        "status": "healthy",
        "checks": {
            "database": await check_database(),
            "cache": await check_cache()
        }
    }
```

### Logging Strategy

- Log all API requests/responses
- Include request ID for tracing
- Log authentication/authorization events
- Log errors with full context
- Use structured logging (JSON format)

### Metrics to Track

- Request count by endpoint
- Response times (latency)
- Error rates by status code
- Active connections
- Rate limit usage

## Caching Strategies

### HTTP Caching

```
Cache-Control: public, max-age=3600     # Client caching
Cache-Control: no-cache, no-store       # No caching

ETag: "hash"
If-None-Match: "hash"
→ 304 Not Modified
```

### Server-Side Caching

- In-memory caching (Redis)
- Database query caching
- API response caching
- Cache invalidation strategies

## Error Handling

### Standardized Error Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [{"field": "email", "message": "Invalid"}],
    "timestamp": "2025-10-16T12:00:00Z",
    "path": "/api/users"
  }
}
```

### Status Code Guidelines

- 2xx Success: 200, 201, 204
- 4xx Errors: 400, 401, 403, 404, 422, 429
- 5xx Server: 500, 503

## Documentation Standards

### OpenAPI/Swagger

```python
app = FastAPI(
    title="My API",
    docs_url="/docs",
    redoc_url="/redoc"
)

@app.get("/api/users/{user_id}", tags=["Users"])
async def get_user(user_id: str):
    """Retrieve user by ID."""
    pass
```

### GraphQL Documentation

- GraphQL introspection (free documentation)
- Use `@deprecated` for backward compatibility
- Document complex types with descriptions

## Best Practices Summary

1. **Versioning** - Plan for breaking changes early
2. **Security** - Implement defense in depth
3. **Rate Limiting** - Protect from abuse
4. **Monitoring** - Observe production APIs
5. **Documentation** - Keep in sync with code
6. **Error Handling** - Standardize responses
7. **Caching** - Balance freshness and performance
8. **Logging** - Collect actionable data
9. **Metrics** - Track important signals
10. **Testing** - Security and performance tests

## Cross-Skill References

- **rest-api-design** - REST-specific patterns
- **graphql-api-design** - GraphQL-specific patterns
- **api-testing** - Testing security and monitoring
