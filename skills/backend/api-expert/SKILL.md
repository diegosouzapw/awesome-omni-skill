---
name: api-expert
description: Handles backend integration using best practices like expo/fetch and React Query. Use when the user needs to connect to an API, handle data fetching, or implement caching.
---

# API Expert Skill

This skill provides expertise in integrating backend services into React Native Expo applications. It emphasizes reliability, performance, and clean architecture.

## Core Principles

- **Use `expo/fetch`**: Always prefer `expo/fetch` over standard `fetch` or `axios`.
- **React Query**: Use `@tanstack/react-query` for managing server state, caching, and synchronization.
- **Service Layer**: Keep API logic in the `services/` directory.
- **Type Safety**: Define Zod schemas for runtime validation and TypeScript interfaces for compile-time safety.

## Instructions

1. **Setup Client**: Ensure the React Query client is configured in `app/_layout.tsx`.
2. **Define Services**: Create service files in `services/api/` that use `expo/fetch`.
3. **Custom Hooks**: Wrap service calls in custom React Query hooks.
4. **Error Handling**: Implement early returns and use `ErrorBoundary` for unexpected failures.
5. **Validation**: Use Zod to validate API responses at the network boundary.

## Example

```tsx
// services/api/user-service.ts
import { fetch } from 'expo/fetch';
import { z } from 'zod';

const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
});

export const fetchUser = async (id: string) => {
  const response = await fetch(`https://api.example.com/users/${id}`);
  if (!response.ok) throw new Error('Failed to fetch user');
  const data = await response.json();
  return UserSchema.parse(data);
};

// hooks/use-user.ts
import { useQuery } from '@tanstack/react-query';
import { fetchUser } from '@/services/api/user-service';

export const useUser = (id: string) => {
  return useQuery({
    queryKey: ['user', id],
    queryFn: () => fetchUser(id),
  });
};
```

See [API Best Practices](references/REFERENCE.md) for more details.
