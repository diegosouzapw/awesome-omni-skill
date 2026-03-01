---
name: api-codegen
description: API client code generation workflow. Use when modifying backend routes, response schemas, or request models. Automatically regenerates TypeScript API client from OpenAPI schema.
---

# API Client Code Generation

This skill manages the workflow for keeping the frontend TypeScript API client in sync with the backend.

## When to use this skill

- After modifying Litestar routes in `backend/app/`
- After changing request/response schemas (msgspec)
- After adding new API endpoints
- When frontend TypeScript errors mention missing types
- Before committing changes that affect the API contract

## Quick Command

```bash
make codegen  # Generate TypeScript API client from backend OpenAPI schema
```

## Complete Workflow

1. **Modify backend** (routes, schemas, endpoints)
   - Edit Litestar route handlers in `backend/app/server/`
   - Update msgspec schemas in `backend/app/domain/*/schemas.py`
   - Add new endpoints or modify existing ones

2. **Generate API client**
   ```bash
   make codegen
   ```
   This will:
   - Start the backend server temporarily
   - Fetch OpenAPI schema from `/schema/openapi.json`
   - Generate TypeScript client in `frontend/src/openapi/`
   - Stop the backend server

3. **Verify generation**
   - Check `frontend/src/openapi/` for new types
   - Review generated request/response interfaces
   - Look for any generation warnings

4. **Update frontend code**
   - Import new types from `@/openapi`
   - Use generated API client methods
   - Fix any TypeScript errors

5. **Type check**
   ```bash
   make check-frontend  # Verify TypeScript compilation
   ```

## What Gets Generated

The codegen process creates:
- **Type definitions**: Request/response interfaces
- **API client methods**: Type-safe API calls
- **Enums and constants**: From backend schemas
- **Path parameters**: Type-safe route params

Location: `frontend/src/openapi/`

## Example Usage

### Backend Schema (msgspec)
```python
# backend/app/domain/users/schemas.py
from msgspec import Struct

class UserCreateRequest(Struct):
    email: str
    name: str

class UserResponse(Struct):
    id: int
    email: str
    name: str
    created_at: str
```

### Generated TypeScript
```typescript
// frontend/src/openapi/models.ts
export interface UserCreateRequest {
  email: string;
  name: string;
}

export interface UserResponse {
  id: number;
  email: string;
  name: string;
  created_at: string;
}
```

### Frontend Usage
```typescript
import { api } from '@/openapi';
import type { UserCreateRequest, UserResponse } from '@/openapi';

const createUser = async (data: UserCreateRequest): Promise<UserResponse> => {
  const response = await api.users.create(data);
  return response;
};
```

## Troubleshooting

### Generation fails
- **Backend not starting**: Check for syntax errors in Python code
- **Port 8000 in use**: Stop existing backend process
- **OpenAPI schema errors**: Verify all route handlers have proper schemas

### Types missing after generation
- **Endpoint not in OpenAPI**: Ensure route is registered in Litestar app
- **Schema not exported**: Check msgspec Struct definitions
- **Route tags**: Verify API grouping/tagging

### Frontend TypeScript errors after codegen
- **Incompatible changes**: Breaking API changes require frontend updates
- **Missing imports**: Update imports to use new generated types
- **Deprecated types**: Remove references to old type names

## Best Practices

1. **Run codegen frequently**: After any backend schema changes
2. **Commit generated files**: Include `frontend/src/openapi/` in commits
3. **Review changes**: Check git diff for generated code changes
4. **Type safety**: Let TypeScript catch API contract mismatches
5. **Breaking changes**: Coordinate with frontend team for major API changes

## Integration with Development Workflow

### Making API Changes
1. Modify backend route/schema
2. Run `make codegen`
3. Update frontend code to use new types
4. Run `make check-frontend`
5. Test changes locally
6. Commit all changes together (backend + frontend + generated)

### Before Pull Request
```bash
make codegen         # Regenerate API client
make check-all       # Type check everything
make test            # Run backend tests
```

This ensures the API contract is in sync across the full stack.
