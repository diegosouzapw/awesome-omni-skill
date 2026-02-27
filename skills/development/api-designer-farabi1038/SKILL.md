---
name: api-designer
description: REST and GraphQL API design expert following best practices
category: coding
version: 1.0.0
author: farabi
---

# API Designer

Expert in designing clean, scalable, and well-documented APIs.

## REST API Best Practices

### URL Structure
- Use nouns, not verbs: `/users` not `/getUsers`
- Use plural nouns: `/users` not `/user`
- Nest for relationships: `/users/{id}/orders`
- Use query params for filtering: `/users?status=active`

### HTTP Methods
- GET: Read resources
- POST: Create resources
- PUT: Full update
- PATCH: Partial update
- DELETE: Remove resources

### Status Codes
- 200: Success
- 201: Created
- 204: No Content
- 400: Bad Request
- 401: Unauthorized
- 403: Forbidden
- 404: Not Found
- 500: Server Error

### Response Format
```json
{
  "data": {...},
  "meta": {"page": 1, "total": 100},
  "errors": []
}
```

## GraphQL Best Practices

- Use descriptive type names
- Implement pagination with connections
- Use input types for mutations
- Handle errors in response, not exceptions

## API Documentation

- OpenAPI/Swagger for REST
- GraphQL introspection + descriptions
- Include examples for all endpoints
- Document error responses
