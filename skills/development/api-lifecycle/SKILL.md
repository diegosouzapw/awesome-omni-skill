---
name: api-lifecycle
description: Provides API design and lifecycle management guidance including versioning, deprecation, rate limiting, and documentation standards. Triggers on api design, rest api, graphql, api versioning, deprecation, breaking changes, rate limiting, api lifecycle, api documentation, openapi.
version: 1.0.0
user-invocable: false
---

# API Lifecycle

## API Design Principles

- **Resource-oriented design:** Model APIs around resources (nouns), not actions (verbs). Use `GET /orders/123` not `GET /getOrder?id=123`. Nest sub-resources logically: `GET /users/123/orders`. Keep nesting to a maximum of two levels.
- **Idempotency:** GET, PUT, and DELETE must be idempotent. Repeated calls produce the same result. For POST (create) operations, support client-provided idempotency keys (`Idempotency-Key` header) to prevent duplicate resource creation on retries.
- **Stateless requests:** Every request must contain all information needed to process it. Do not store client session state on the server between requests. Use tokens (JWT, opaque) for authentication context.
- **Pagination:** Always paginate list endpoints. Choose the appropriate strategy:
  - **Cursor-based (recommended):** Use an opaque cursor token. Best for real-time data, large datasets, and when items can be inserted/deleted between pages. Return `next_cursor` in the response.
  - **Offset-based:** Use `?offset=20&limit=10`. Simpler to implement but has consistency issues with changing datasets. Acceptable for admin UIs and static data.
  - Always include `total_count` (or indicate if more pages exist), `next` link, and the current page size in the response.
- **Consistent naming:** Use snake_case for JSON fields (`created_at`, not `createdAt`). Use plural nouns for collection endpoints (`/users`, not `/user`). Use kebab-case for URL paths (`/order-items`, not `/orderItems`). Be consistent across the entire API.
- **HTTP methods and status codes:** Use methods correctly: GET (read), POST (create), PUT (full replace), PATCH (partial update), DELETE (remove). Return appropriate status codes: 200 (success), 201 (created), 204 (no content), 400 (bad request), 401 (unauthorized), 403 (forbidden), 404 (not found), 409 (conflict), 422 (unprocessable entity), 429 (rate limited), 500 (server error).

## Versioning Strategies

| Strategy | Format | Pros | Cons | Best For |
|----------|--------|------|------|----------|
| **URL path** | `/v1/users`, `/v2/users` | Explicit, easy to route, easy to document, cacheable | URL pollution, clients must update all URLs | Public APIs, APIs with infrequent breaking changes |
| **Header** | `Accept: application/vnd.api+json;version=2` | Clean URLs, version is metadata not resource identity | Harder to test in browser, less discoverable | Internal APIs, APIs with frequent version changes |
| **Query parameter** | `/users?version=2` | Easy to add, backward-compatible | Breaks caching, pollutes query string, easy to forget | Quick prototyping, transitional use only |

Recommendations:

- For public APIs, use URL path versioning. It is the most explicit and easiest for consumers to understand.
- Only increment the major version for breaking changes. Non-breaking changes are added to the current version.
- Support at most two major versions simultaneously (current and previous).
- Set a sunset date for old versions at the time you release a new version.
- Never version individual endpoints differently from the rest of the API.

## Breaking vs Non-Breaking Changes

**Breaking changes (require a new version):**

- Removing a field from a response body
- Renaming a field in a request or response
- Changing the type of a field (string to integer, object to array)
- Removing or renaming an endpoint
- Changing the HTTP method of an endpoint
- Adding a new required field to a request body
- Changing the meaning or behavior of an existing field
- Changing authentication or authorization requirements
- Changing error response format

**Non-breaking changes (safe to add to the current version):**

- Adding a new optional field to a request body
- Adding a new field to a response body
- Adding a new endpoint
- Adding a new optional query parameter
- Adding a new enum value (if clients handle unknown values gracefully)
- Adding a new HTTP header
- Improving error messages (without changing the error code or structure)
- Increasing rate limits
- Adding support for a new content type

Rule: When in doubt, treat a change as breaking. It is better to version unnecessarily than to break consumers silently.

## Deprecation Process

Follow these five steps for every deprecation:

1. **Announce:** Notify API consumers at least 90 days before removal. Use email, changelog, developer portal, and in-API deprecation notices. For internal APIs, 30 days notice is acceptable.

2. **Mark:** Add the `Sunset` header to deprecated endpoint responses with the planned removal date. Add the `Deprecation` header with the date the endpoint was deprecated. Mark the endpoint as deprecated in OpenAPI documentation.
   ```http
   HTTP/1.1 200 OK
   Deprecation: Sat, 01 Feb 2025 00:00:00 GMT
   Sunset: Sat, 01 May 2025 00:00:00 GMT
   Link: <https://api.example.com/v2/users>; rel="successor-version"
   ```

3. **Monitor usage:** Track how many consumers still use the deprecated endpoint. Set up dashboards showing call volume over time. Contact heavy users directly with migration guidance.

4. **Sunset with notice:** When usage drops below the threshold (or after the sunset date), return `410 Gone` with a response body pointing to the replacement endpoint. Keep this response active for at least 30 days.
   ```json
   {
     "type": "https://api.example.com/errors/gone",
     "title": "Endpoint Removed",
     "detail": "This endpoint was removed on 2025-05-01. Use /v2/users instead.",
     "replacement": "/v2/users"
   }
   ```

5. **Remove and document:** Remove the endpoint code after the 410 period. Update all documentation, SDK examples, and migration guides. Record the deprecation in the changelog with the date and replacement.

## Rate Limiting

Implement rate limiting on every public-facing API:

**Strategies:**

- **Fixed window:** Count requests in fixed time intervals (e.g., 100 requests per minute). Simple to implement but allows burst traffic at window boundaries.
- **Sliding window:** Count requests in a rolling time window. Smoother than fixed window, prevents boundary bursts. Slightly more complex to implement.
- **Token bucket (recommended):** Tokens are added at a fixed rate. Each request consumes a token. Allows controlled bursts while enforcing an average rate. Best balance of flexibility and protection.

**Response headers:** Include rate limit information in every response:
```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 67
X-RateLimit-Reset: 1698278400
Retry-After: 30
```

**429 response format:**
```json
{
  "type": "https://api.example.com/errors/rate-limit",
  "title": "Rate Limit Exceeded",
  "detail": "You have exceeded the rate limit of 100 requests per minute.",
  "retry_after": 30
}
```

**Rules:**

- Set different rate limits per tier: free (60/min), standard (300/min), premium (1000/min).
- Rate limit by API key or authenticated user, not by IP alone (IP-based limiting breaks shared networks).
- Apply stricter limits to authentication endpoints (10/min for login) to prevent brute force.
- Return `429 Too Many Requests` with a `Retry-After` header. Never silently drop requests.
- Log rate limit hits for monitoring and capacity planning.

## Error Standards

Use RFC 7807 Problem Details format for all error responses:

```json
{
  "type": "https://api.example.com/errors/validation-error",
  "title": "Validation Error",
  "status": 422,
  "detail": "The request body contains invalid fields.",
  "instance": "/users/123",
  "errors": [
    {
      "field": "email",
      "code": "INVALID_FORMAT",
      "message": "Must be a valid email address."
    },
    {
      "field": "age",
      "code": "OUT_OF_RANGE",
      "message": "Must be between 0 and 150."
    }
  ]
}
```

Rules:

- **`type`:** A URI that identifies the error type. Use the same URI for the same class of errors. Link to documentation that explains the error.
- **`title`:** A short, human-readable summary. Same for every occurrence of this error type.
- **`status`:** The HTTP status code. Must match the actual response status.
- **`detail`:** A human-readable explanation specific to this occurrence. Include enough context to help the consumer fix the issue.
- **`instance`:** The URI of the specific resource or request that caused the error.
- **`errors`:** For validation errors, include an array of field-level errors with `field`, `code`, and `message`.
- Use consistent error codes across the API: `NOT_FOUND`, `VALIDATION_ERROR`, `UNAUTHORIZED`, `FORBIDDEN`, `RATE_LIMITED`, `INTERNAL_ERROR`.
- Never expose internal implementation details (stack traces, SQL errors, file paths) in error responses.

## Documentation Standards

- **OpenAPI for REST APIs:** Maintain an OpenAPI 3.1 specification file (`openapi.yaml` or `openapi.json`) as the single source of truth. Include descriptions for every endpoint, parameter, request body, and response. Provide realistic examples for every schema.
- **AsyncAPI for event-driven APIs:** Use AsyncAPI 3.0 for documenting message brokers, WebSocket APIs, and event streams. Include message schemas, channel descriptions, and bindings.
- **GraphQL introspection:** Keep the schema as the primary documentation. Add descriptions to every type, field, and argument using schema comments. Expose the introspection endpoint in development; disable in production if not needed by consumers.
- **Examples for every endpoint:** Every documented endpoint must include at least one request/response example with realistic data. Include examples for success cases, common error cases, and edge cases (empty results, pagination).
- **SDK and code samples:** Provide code samples in at least two languages for public APIs. Keep samples up to date with every API version change. Test code samples in CI.
- **Changelog:** Maintain a versioned changelog documenting every API change. Categorize changes as Added, Changed, Deprecated, Removed, Fixed, Security.

## GraphQL Lifecycle

- **Schema evolution over versioning:** GraphQL APIs do not use URL versioning. Instead, evolve the schema by adding new fields and types without removing existing ones. This is possible because clients request only the fields they need.
- **`@deprecated` directive:** Mark fields and enum values for removal using `@deprecated(reason: "Use newField instead")`. The deprecation reason must include the replacement field or migration path.
  ```graphql
  type User {
    name: String!
    fullName: String!
    firstName: String @deprecated(reason: "Use fullName instead. Will be removed 2025-06-01.")
    lastName: String @deprecated(reason: "Use fullName instead. Will be removed 2025-06-01.")
  }
  ```
- **Field usage tracking:** Instrument the GraphQL server to track which fields are requested by which clients. Use this data to determine when deprecated fields can be safely removed. Tools: Apollo Studio, GraphQL Hive, or custom middleware.
- **Schema stitching and federation:** For large APIs, split the schema into domain-specific subgraphs using Apollo Federation or similar. Each team owns its subgraph and can evolve it independently. Define clear entity boundaries and shared types.
- **Breaking change prevention:** Use schema linting (graphql-schema-linter, Apollo rover) in CI to detect breaking changes before they are merged. Block PRs that remove fields, change field types, or remove enum values without deprecation.

## Anti-Patterns

- **Versionless API:** Never shipping a versioned API and being unable to make changes without breaking consumers. Version from day one, even if you start with v1 and never expect to change.
- **Breaking changes without notice:** Removing fields, changing types, or renaming endpoints without going through the deprecation process. Every breaking change must follow the five-step deprecation process.
- **Inconsistent error formats:** Returning different error structures from different endpoints (plain text from one, JSON from another, HTML from a third). Use RFC 7807 Problem Details consistently across the entire API.
- **No pagination:** Returning unbounded lists from collection endpoints. Every list endpoint must support pagination. Set a default page size (20-50) and a maximum page size (100-200).
- **No rate limiting:** Leaving API endpoints without rate limits. Every endpoint must be rate limited. Unauthenticated endpoints need stricter limits than authenticated ones.
- **Undocumented endpoints:** Endpoints that exist in code but not in the API documentation. Every endpoint must be documented in the OpenAPI spec. Use linting tools to detect undocumented endpoints.
- **Chatty APIs:** Requiring multiple round trips to accomplish a single user action. Design endpoints that return the data clients need in a single request. Use field selection, includes, or GraphQL to reduce round trips.
- **Ignoring backward compatibility:** Adding required fields to request bodies, changing field types, or altering business logic without versioning. Always default new fields and maintain backward compatibility within a version.
- **Exposing internal models directly:** Mapping database tables 1:1 to API resources. API resources should represent the domain model, not the storage model. Use DTOs or view models to decouple the API contract from internal implementation.
