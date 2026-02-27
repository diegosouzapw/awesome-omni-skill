---
name: api-design-patterns
description: Principles for REST, GraphQL, versioning, and API authentication.
---

# API Design Patterns

## When to use this skill
- Designing new API endpoints.
- Documenting APIs (OpenAPI/Swagger).
- Implementing authentication strategies.

## 1. RESTful Conventions
- **Nouns**: Use nouns for resources (`/users`, not `/getUsers`).
- **Verbs**: Use correct HTTP methods (`GET` read, `POST` create, `PUT` replace, `PATCH` update, `DELETE` remove).
- **Status Codes**: 200 OK, 201 Created, 400 Bad Request, 401 Unauth, 403 Forbidden, 404 Not Found, 422 Validation Error.

## 2. Response Structure
- **Envelope**: Standardize response JSON.
  ```json
  {
    "data": { ... },
    "meta": { "pagination": ... }
  }
  ```
- **Errors**: Return structured error objects, not just plain strings.

## 3. Versioning
- **URL**: `/api/v1/resource` is preferred for explicit versioning.
- **Breaking Changes**: Never introduce breaking changes to an existing version. Create v2.

## 4. Authentication
- **Bearer Token**: Use `Authorization: Bearer <token>` (JWT or Opaque).
- **Stateless**: API should rarely rely on session cookies (CSRF issues) unless it is a first-party SPA.
