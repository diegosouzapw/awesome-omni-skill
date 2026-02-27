---
name: api-design
kind: evaluator
description: Evaluates API endpoint design for RESTfulness, consistency, error handling, and documentation.
metadata:
  categories: ["api", "endpoint", "schema", "rest", "graphql"]
  dimensions:
    - name: restfulness
      weight: 0.25
      description: Does the API follow REST conventions and HTTP semantics?
    - name: consistency
      weight: 0.25
      description: Are naming, structure, and patterns consistent?
    - name: error_responses
      weight: 0.3
      description: Are errors well-structured, informative, and secure?
    - name: documentation
      weight: 0.2
      description: Is the API self-documenting with clear contracts?
---

# API Design Evaluator

Evaluate API design output against these criteria.

## RESTfulness (25%)

- Are HTTP methods used correctly (GET for reads, POST for creation, PUT/PATCH for updates, DELETE for removal)?
- Are resource URLs noun-based and plural (/users, /orders, not /getUsers, /createOrder)?
- Are status codes appropriate (201 for creation, 204 for no content, 404 for not found)?
- Is the URL hierarchy logical (/users/:id/orders, not /user-orders)?
- Are query parameters used for filtering/sorting/pagination (not path segments)?
- Is HATEOAS considered where appropriate?
- For non-REST APIs (GraphQL, RPC): are equivalent conventions followed?

## Consistency (25%)

- Are naming conventions consistent (camelCase vs snake_case, chosen and stuck to)?
- Are response shapes consistent across endpoints (same envelope, same error format)?
- Are pagination patterns consistent (cursor vs offset, same parameter names)?
- Do similar operations behave similarly (all CRUD endpoints for all resources follow same pattern)?
- Are date formats consistent (ISO 8601)?
- Are ID formats consistent (UUID vs integer, same field name)?

## Error Responses (30%)

- Do error responses include a machine-readable error code?
- Do error responses include a human-readable message?
- Are validation errors specific (which field failed, why)?
- Are errors safe (no stack traces, internal paths, or SQL in production responses)?
- Are error codes documented and stable (clients can switch on them)?
- Is there a consistent error envelope (same shape for all errors)?
- Are rate limit errors informative (retry-after header)?
- Do 5xx responses avoid leaking implementation details?

## Documentation (20%)

- Are request/response schemas defined (OpenAPI, JSON Schema, or equivalent)?
- Are required vs optional fields clearly marked?
- Are example requests and responses provided?
- Are authentication requirements documented per endpoint?
- Are rate limits and quotas documented?
- Are breaking change policies documented (versioning strategy)?
- Is the documentation generated from code or kept in sync?

## Severity Guide

- **Blocker**: Wrong HTTP method semantics, error responses leaking internals, missing auth on sensitive endpoint
- **Important**: Inconsistent naming across endpoints, missing error codes, no pagination on list endpoints
- **Suggestion**: Minor naming improvements, additional examples, HATEOAS links
