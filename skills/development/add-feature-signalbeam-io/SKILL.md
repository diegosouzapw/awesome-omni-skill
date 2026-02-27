---
name: add-feature
description: Scaffold a new frontend feature module with page, components, API service, and hooks
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
user-invocable: true
---

# Add Frontend Feature

Scaffold a new feature module following the project's React + TypeScript conventions.

## Arguments

- `{feature}` — Feature name in kebab-case (e.g., `alerts`, `rollouts`)

## Structure

Creates the following structure under `web/src/features/{feature}/`:

```
{feature}/
├── pages/
│   └── {feature}-page.tsx
└── components/
    └── {feature}-overview.tsx
```

And adds an API service at `web/src/api/services/{feature}.api.ts`.

## 1. API Service (`web/src/api/services/{feature}.api.ts`)

```typescript
/**
 * {Feature} API client
 */

import { apiRequest } from '../client'
import { appendTenantId } from './tenant'
import type { PaginatedResponse } from '../types'

// Define types inline or in ../types/index.ts
export interface {Entity} {
  id: string
  // fields
}

const BASE_PATH = '/api/{feature}'

export const {feature}Api = {
  /**
   * Get paginated list
   */
  async getAll(page = 1, pageSize = 20): Promise<PaginatedResponse<{Entity}>> {
    const params = new URLSearchParams()
    appendTenantId(params)
    params.set('pageNumber', page.toString())
    params.set('pageSize', pageSize.toString())

    return apiRequest({ url: BASE_PATH, params })
  },

  /**
   * Get by ID
   */
  async getById(id: string): Promise<{Entity}> {
    const params = new URLSearchParams()
    appendTenantId(params)
    return apiRequest({ url: `${BASE_PATH}/${id}`, params })
  },
}
```

## 2. Page (`web/src/features/{feature}/pages/{feature}-page.tsx`)

```tsx
import { {Feature}Overview } from '../components/{feature}-overview'

export function {Feature}Page() {
  return (
    <div className="container mx-auto py-6">
      <{Feature}Overview />
    </div>
  )
}
```

## 3. Overview Component (`web/src/features/{feature}/components/{feature}-overview.tsx`)

```tsx
import { useQuery } from '@tanstack/react-query'
import { {feature}Api } from '@/api/services/{feature}.api'

export function {Feature}Overview() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['{feature}'],
    queryFn: () => {feature}Api.getAll(),
    staleTime: 30_000,
  })

  if (isLoading) return <div>Loading...</div>
  if (error) return <div>Error loading {feature}</div>

  return (
    <div>
      <h1 className="text-2xl font-bold mb-4">{Feature}</h1>
      {/* Render data */}
    </div>
  )
}
```

## 4. Add Route

Remind the user to add the route in `web/src/routes/`:

```tsx
import { {Feature}Page } from '@/features/{feature}/pages/{feature}-page'

// Add to router config:
{ path: '/{feature}', element: <{Feature}Page /> }
```

## Checklist

- [ ] API service uses `apiRequest` and `appendTenantId`
- [ ] Types defined as interfaces, no `any`
- [ ] TanStack Query for server state
- [ ] Functional components only
- [ ] shadcn/ui components with Tailwind CSS
- [ ] Route added to router config

## Guidelines

- Follow existing features (e.g., `devices/`, `bundles/`) as reference
- Keep components small and focused
- Use barrel exports where the project already does

## Related Skills

- `/add-query` to create the backend API endpoint this feature calls
- `/add-command` to create mutation endpoints
