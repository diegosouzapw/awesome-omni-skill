---
name: frontend-patterns
version: "2.0.0"
description: Frontend development and API integration patterns for React, TypeScript, and state management
sasmp_version: "1.3.0"
bonded_agent: 06-frontend-integration
bond_type: PRIMARY_BOND

# Skill Configuration
atomic_design:
  single_responsibility: "Frontend API integration and state management"
  boundaries:
    includes: [data_fetching, state_management, error_handling, caching, optimistic_updates]
    excludes: [api_design, backend_implementation, database_queries]

parameter_validation:
  schema:
    type: object
    properties:
      framework:
        type: string
        enum: [react, vue, svelte, nextjs]
      state_library:
        type: string
        enum: [tanstack_query, swr, zustand, redux, apollo]
      api_type:
        type: string
        enum: [rest, graphql]

retry_config:
  enabled: true
  max_attempts: 3
  backoff:
    type: exponential
    initial_delay_ms: 1000
    max_delay_ms: 10000

logging:
  level: INFO
  fields: [query_key, status, duration_ms, cache_hit]

dependencies:
  skills: [rest, graphql]
  agents: [06-frontend-integration]
---

# Frontend Patterns Skill

## Purpose
Build robust frontend applications with proper API integration and state management.

## Data Fetching Patterns

### TanStack Query (React Query)

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Query configuration
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,       // 5 minutes
      gcTime: 30 * 60 * 1000,         // 30 minutes (formerly cacheTime)
      retry: 3,
      retryDelay: (attempt) => Math.min(1000 * 2 ** attempt, 30000),
      refetchOnWindowFocus: false,
    },
  },
});

// Type-safe API client
const api = {
  users: {
    list: async (params: { page: number; limit: number }) => {
      const res = await fetch(`/api/users?${new URLSearchParams(params as any)}`);
      if (!res.ok) throw new ApiError(res);
      return res.json() as Promise<PaginatedResponse<User>>;
    },
    get: async (id: string) => {
      const res = await fetch(`/api/users/${id}`);
      if (!res.ok) throw new ApiError(res);
      return res.json() as Promise<User>;
    },
    create: async (data: CreateUserInput) => {
      const res = await fetch('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      });
      if (!res.ok) throw new ApiError(res);
      return res.json() as Promise<User>;
    },
  },
};

// Query hook with pagination
function useUsers(page: number) {
  return useQuery({
    queryKey: ['users', 'list', { page }],
    queryFn: () => api.users.list({ page, limit: 20 }),
    placeholderData: (prev) => prev,  // Keep previous data while loading
  });
}

// Single user query
function useUser(id: string) {
  return useQuery({
    queryKey: ['users', 'detail', id],
    queryFn: () => api.users.get(id),
    enabled: !!id,
  });
}

// Mutation with optimistic update
function useCreateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: api.users.create,
    onMutate: async (newUser) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['users', 'list'] });

      // Snapshot previous value
      const previous = queryClient.getQueryData(['users', 'list']);

      // Optimistically update
      queryClient.setQueryData(['users', 'list'], (old: any) => ({
        ...old,
        data: [...(old?.data || []), { ...newUser, id: 'temp-id' }],
      }));

      return { previous };
    },
    onError: (err, newUser, context) => {
      // Rollback on error
      queryClient.setQueryData(['users', 'list'], context?.previous);
    },
    onSettled: () => {
      // Refetch after mutation
      queryClient.invalidateQueries({ queryKey: ['users', 'list'] });
    },
  });
}
```

### SWR Pattern

```typescript
import useSWR, { mutate } from 'swr';
import useSWRMutation from 'swr/mutation';

const fetcher = async (url: string) => {
  const res = await fetch(url);
  if (!res.ok) throw new Error('Failed to fetch');
  return res.json();
};

function useUsers() {
  const { data, error, isLoading, isValidating } = useSWR<User[]>(
    '/api/users',
    fetcher,
    {
      revalidateOnFocus: false,
      dedupingInterval: 5000,
    }
  );

  return {
    users: data,
    isLoading,
    isRefreshing: isValidating && data,
    error,
  };
}

// SWR Mutation
function useCreateUser() {
  return useSWRMutation(
    '/api/users',
    async (url: string, { arg }: { arg: CreateUserInput }) => {
      const res = await fetch(url, {
        method: 'POST',
        body: JSON.stringify(arg),
      });
      return res.json();
    },
    {
      onSuccess: () => mutate('/api/users'),
    }
  );
}
```

## State Management

### Zustand (Recommended for most cases)

```typescript
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => void;
  updateUser: (updates: Partial<User>) => void;
}

const useAuthStore = create<AuthState>()(
  devtools(
    persist(
      immer((set, get) => ({
        user: null,
        token: null,
        isAuthenticated: false,

        login: async (credentials) => {
          const response = await api.auth.login(credentials);
          set((state) => {
            state.user = response.user;
            state.token = response.token;
            state.isAuthenticated = true;
          });
        },

        logout: () => {
          set((state) => {
            state.user = null;
            state.token = null;
            state.isAuthenticated = false;
          });
        },

        updateUser: (updates) => {
          set((state) => {
            if (state.user) {
              Object.assign(state.user, updates);
            }
          });
        },
      })),
      { name: 'auth-store' }
    ),
    { name: 'Auth' }
  )
);

// Selectors (prevent unnecessary re-renders)
const useUser = () => useAuthStore((state) => state.user);
const useIsAuthenticated = () => useAuthStore((state) => state.isAuthenticated);
```

### Redux Toolkit (Enterprise)

```typescript
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';

// Async thunk
export const fetchUsers = createAsyncThunk(
  'users/fetchAll',
  async (params: { page: number }, { rejectWithValue }) => {
    try {
      return await api.users.list(params);
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

// Slice
const usersSlice = createSlice({
  name: 'users',
  initialState: {
    items: [] as User[],
    status: 'idle' as 'idle' | 'loading' | 'succeeded' | 'failed',
    error: null as string | null,
    pagination: { page: 1, total: 0 },
  },
  reducers: {
    userAdded: (state, action: PayloadAction<User>) => {
      state.items.push(action.payload);
    },
    userUpdated: (state, action: PayloadAction<User>) => {
      const index = state.items.findIndex((u) => u.id === action.payload.id);
      if (index !== -1) {
        state.items[index] = action.payload;
      }
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.items = action.payload.data;
        state.pagination = action.payload.pagination;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.payload as string;
      });
  },
});

export const { userAdded, userUpdated } = usersSlice.actions;
export default usersSlice.reducer;
```

## Error Handling

```typescript
// Custom error class
class ApiError extends Error {
  constructor(
    public response: Response,
    public data?: { type: string; title: string; detail?: string }
  ) {
    super(data?.title || 'API Error');
    this.name = 'ApiError';
  }

  static async fromResponse(response: Response): Promise<ApiError> {
    const data = await response.json().catch(() => null);
    return new ApiError(response, data);
  }
}

// Error boundary component
function QueryErrorBoundary({ children }: { children: React.ReactNode }) {
  const queryClient = useQueryClient();

  return (
    <QueryErrorResetBoundary>
      {({ reset }) => (
        <ErrorBoundary
          onReset={reset}
          fallbackRender={({ error, resetErrorBoundary }) => (
            <div className="error-container">
              <h2>Something went wrong</h2>
              <p>{error.message}</p>
              <button onClick={resetErrorBoundary}>Try again</button>
            </div>
          )}
        >
          {children}
        </ErrorBoundary>
      )}
    </QueryErrorResetBoundary>
  );
}

// Hook with error handling
function useUsersSafe(page: number) {
  const query = useUsers(page);

  useEffect(() => {
    if (query.error instanceof ApiError) {
      if (query.error.response.status === 401) {
        // Redirect to login
        router.push('/login');
      } else if (query.error.response.status >= 500) {
        // Show toast
        toast.error('Server error. Please try again later.');
      }
    }
  }, [query.error]);

  return query;
}
```

## Optimistic Updates Pattern

```typescript
function useTodoToggle() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (todo: Todo) =>
      api.todos.update(todo.id, { completed: !todo.completed }),

    onMutate: async (todo) => {
      await queryClient.cancelQueries({ queryKey: ['todos'] });

      const previous = queryClient.getQueryData<Todo[]>(['todos']);

      queryClient.setQueryData<Todo[]>(['todos'], (old) =>
        old?.map((t) =>
          t.id === todo.id ? { ...t, completed: !t.completed } : t
        )
      );

      return { previous };
    },

    onError: (err, todo, context) => {
      queryClient.setQueryData(['todos'], context?.previous);
      toast.error('Failed to update todo');
    },

    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
}
```

---

## Unit Test Template

```typescript
import { describe, it, expect, vi } from 'vitest';
import { renderHook, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });
  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );
};

describe('Frontend Patterns', () => {
  describe('useUsers hook', () => {
    it('should fetch and return users', async () => {
      vi.spyOn(global, 'fetch').mockResolvedValue({
        ok: true,
        json: async () => ({ data: [{ id: '1', name: 'Test' }] }),
      } as Response);

      const { result } = renderHook(() => useUsers(1), {
        wrapper: createWrapper(),
      });

      await waitFor(() => expect(result.current.isSuccess).toBe(true));
      expect(result.current.data?.data).toHaveLength(1);
    });

    it('should handle errors', async () => {
      vi.spyOn(global, 'fetch').mockResolvedValue({
        ok: false,
        status: 500,
      } as Response);

      const { result } = renderHook(() => useUsers(1), {
        wrapper: createWrapper(),
      });

      await waitFor(() => expect(result.current.isError).toBe(true));
    });
  });

  describe('Zustand store', () => {
    it('should update auth state on login', async () => {
      const { result } = renderHook(() => useAuthStore());

      await result.current.login({ email: 'test@test.com', password: 'pass' });

      expect(result.current.isAuthenticated).toBe(true);
      expect(result.current.user).toBeDefined();
    });

    it('should clear state on logout', () => {
      const { result } = renderHook(() => useAuthStore());

      result.current.logout();

      expect(result.current.isAuthenticated).toBe(false);
      expect(result.current.user).toBeNull();
    });
  });
});
```

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Infinite refetching | Missing dependency array | Use stable queryKey |
| Stale data shown | staleTime too high | Reduce staleTime or invalidate |
| Memory leak | Unmounted component | Use cleanup in useEffect |
| Too many re-renders | Non-memoized selectors | Use shallow comparison |
| Optimistic rollback fails | Missing previous snapshot | Always capture previous state |

---

## Quality Checklist

- [ ] Data fetching with TanStack Query or SWR
- [ ] Type-safe API client
- [ ] Error boundaries configured
- [ ] Loading states handled
- [ ] Optimistic updates for mutations
- [ ] Cache invalidation strategy
- [ ] State persistence (where needed)
- [ ] Memoization applied
- [ ] Tests for hooks and stores
