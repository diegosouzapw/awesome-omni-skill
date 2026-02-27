---
name: rest-api
description: "WHAT: REST API data fetching with TanStack Query and OpenTelemetry tracing. WHEN: fetching non-GraphQL endpoints, implementing pagination, cache invalidation. KEYWORDS: REST, api, useFetch, useQuery, useInfiniteQuery, query keys, pagination, tracing, fetch."
---

# REST API Standards

Standards for fetching REST API data using TanStack Query with OpenTelemetry tracing.

## When to Use

Use REST API **only when data is NOT available in GraphQL schema**. Always verify GraphQL schema first.

**Use these standards when:**
- Creating new API endpoints not in GraphQL
- Fetching from legacy REST endpoints
- Integrating with external APIs
- Implementing infinite scroll pagination
- Setting up cache invalidation

**Why:** GraphQL provides better type safety and reduces over-fetching, but REST is needed for legacy endpoints and external APIs.

## Core Principles

### useFetch Hook with OpenTelemetry

**Always use `useFetch` hook instead of native `fetch()`. It provides automatic OpenTelemetry tracing, request ID generation, and authentication middleware.**

✅ **Good:**
```typescript
import { useFetch } from '@libs/networking-client';

export const useGetProducts = (params: GetProductsRequest) => {
  const fetch = useFetch(); // Provides traced fetch with request IDs

  return useQuery({
    queryKey: PRODUCT_QUERY_KEYS.list(params),
    queryFn: () => getProducts(params, queryKey, fetch),
  });
};
```

❌ **Bad:**
```typescript
// Never use native fetch directly
export const useGetProducts = (params: GetProductsRequest) => {
  return useQuery({
    queryKey: ['products', params],
    queryFn: async () => {
      const response = await fetch('/api/products'); // ❌ No tracing
      return response.json();
    },
  });
};
```

**Why:** `useFetch` automatically:
- Creates OpenTelemetry spans for distributed tracing
- Adds unique request IDs for debugging
- Handles authentication middleware
- Provides consistent error handling

**Real-world usage:** 200+ usages in shared-mobile-modules codebase.

### Hierarchical Query Keys

Use pattern: `[domain, operation, params]`

✅ **Good:**
```typescript
export const PRODUCT_QUERY_KEYS = {
  all: ['products'] as const,
  list: (params?) => ['products', 'list', params] as const,
  detail: (id: string) => ['products', 'detail', id] as const,
} as const;

// Usage
queryKey: PRODUCT_QUERY_KEYS.list({ category: 'electronics' })
// Result: ['products', 'list', { category: 'electronics' }]
```

❌ **Bad:**
```typescript
// Flat keys - hard to invalidate
queryKey: ['products']
queryKey: ['productList']
queryKey: ['product', id, 'random'] // Inconsistent structure
```

**Why:** Hierarchical keys enable precise cache invalidation:
- `queryClient.invalidateQueries({ queryKey: PRODUCT_QUERY_KEYS.all })` - Invalidates all product queries
- `queryClient.invalidateQueries({ queryKey: PRODUCT_QUERY_KEYS.list() })` - Invalidates only list queries

### File Organization

Organize by domain:
```
src/data-access/query/{domain}/
├── constants.ts      # Query keys & endpoints
├── service.ts        # Fetch functions
├── schema.ts         # TypeScript types
├── hooks.ts          # TanStack Query hooks
└── index.ts          # Exports
```

**Why:** Domain-based organization keeps API logic together and makes it easy to find related code.

## Type Definitions

Define explicit interfaces for all requests and responses.

```typescript
// schema.ts

/**
 * Product entity from API
 */
export interface Product {
  /** Unique identifier */
  id: string;
  /** Product name */
  name: string;
  /** Price in cents */
  price: number;
  /** Product category */
  category: 'electronics' | 'clothing' | 'books';
  /** Availability status */
  status: 'active' | 'inactive';
  /** Creation timestamp (ISO 8601) */
  createdAt: string;
}

/**
 * Request parameters for fetching products
 */
export interface GetProductsRequest {
  cursor?: string;
  limit?: number;
  category?: Product['category'];
  search?: string;
}

/**
 * Response for product list with pagination
 */
export interface GetProductsResponse {
  data: Product[];
  pagination: {
    nextCursor?: string;
    hasMore: boolean;
  };
}
```

**Why:** Explicit types provide autocomplete, catch errors at compile time, and serve as documentation.

## Service Functions

Implement fetch functions using the traced `fetch` parameter from `useFetch`.

```typescript
// service.ts
import type { DataAccessGet } from './schema';

export const getProducts: DataAccessGet<GetProductsResponse, GetProductsRequest> = async (
  params,
  queryKey,
  fetch
) => {
  // Build query params
  const searchParams = new URLSearchParams();
  if (params.cursor) searchParams.append('cursor', params.cursor);
  if (params.limit) searchParams.append('limit', params.limit.toString());
  if (params.category) searchParams.append('category', params.category);

  // fetch parameter comes from useFetch() - includes tracing
  const response = await fetch(`${PRODUCT_ENDPOINTS.LIST}?${searchParams}`, queryKey, {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
    },
  });

  if (!response.ok) {
    throw new Error(`Failed to fetch products: ${response.status}`);
  }

  return await response.json();
};
```

**Why:** Centralized service functions make API calls consistent and easier to test.

## Custom Hooks

Create domain-specific hooks using TanStack Query.

```typescript
// hooks.ts
import { useQuery } from '@tanstack/react-query';
import { useFetch, useLocalizeParams } from '@/hooks/data-access';

/**
 * Hook for fetching products list
 */
export const useGetProducts = (params: GetProductsRequest, options?) => {
  const fetch = useFetch();
  const localizeParams = useLocalizeParams();
  const requestParams = { ...params, ...localizeParams };

  return useQuery({
    queryKey: PRODUCT_QUERY_KEYS.list(requestParams),
    queryFn: () => getProducts(requestParams, queryKey, fetch),
    staleTime: 5 * 60 * 1000, // 5 minutes
    ...options,
  });
};

/**
 * Hook for fetching single product
 */
export const useGetProduct = (productId: string, options?) => {
  const fetch = useFetch();
  const localizeParams = useLocalizeParams();

  return useQuery({
    queryKey: PRODUCT_QUERY_KEYS.detail(productId),
    queryFn: () => getProduct({ id: productId, ...localizeParams }, queryKey, fetch),
    enabled: !!productId,
    staleTime: 10 * 60 * 1000, // 10 minutes
    ...options,
  });
};
```

**Why:** Custom hooks abstract TanStack Query details, include localization automatically, and provide consistent patterns.

## Infinite Queries for Pagination

Use `useInfiniteQuery` for paginated lists with infinite scroll.

```typescript
// hooks.ts
import { useInfiniteQuery } from '@tanstack/react-query';

export const useGetProductsInfinite = (params: Omit<GetProductsRequest, 'cursor'>, options?) => {
  const fetch = useFetch();
  const localizeParams = useLocalizeParams();
  const requestParams = { ...params, ...localizeParams };

  return useInfiniteQuery({
    queryKey: [...PRODUCT_QUERY_KEYS.list(requestParams), 'infinite'],
    queryFn: ({ pageParam }) => {
      return getProducts({ ...requestParams, cursor: pageParam }, queryKey, fetch);
    },
    initialPageParam: undefined,
    getNextPageParam: (lastPage) => lastPage.pagination.nextCursor,
    staleTime: 5 * 60 * 1000,
    ...options,
  });
};
```

**Why:** `useInfiniteQuery` manages pagination state automatically and enables smooth infinite scroll UX.

## Component Integration

### Standard Query Usage

```typescript
// ProductList.tsx
import { useGetProducts } from '@/data-access/query/products';

export const ProductList = () => {
  const { data, isLoading, error, refetch } = useGetProducts({
    limit: 20,
    sort: 'createdAt',
  });

  if (isLoading && !data) return <LoadingSpinner />;
  if (error && !data) return <ErrorMessage error={error} onRetry={refetch} />;
  if (!data?.data.length) return <EmptyState />;

  return (
    <FlatList
      data={data.data}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => <ProductItem product={item} />}
    />
  );
};
```

### Infinite Scroll Usage

```typescript
// ProductInfiniteList.tsx
import { useGetProductsInfinite } from '@/data-access/query/products';

export const ProductInfiniteList = ({ category }) => {
  const {
    data: infiniteData,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isLoading,
    error,
  } = useGetProductsInfinite({ category, limit: 20 });

  // Flatten pages into single array
  const products = infiniteData?.pages.flatMap((page) => page.data) || [];

  if (isLoading && !infiniteData) return <LoadingSpinner />;
  if (error && !infiniteData) return <ErrorMessage error={error} />;
  if (!products.length) return <EmptyState />;

  return (
    <FlatList
      data={products}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => <ProductItem product={item} />}
      onEndReached={() => {
        if (hasNextPage && !isFetchingNextPage) {
          fetchNextPage();
        }
      }}
      onEndReachedThreshold={0.5}
      ListFooterComponent={isFetchingNextPage ? <LoadingSpinner /> : null}
    />
  );
};
```

**Why:** Flattening pages simplifies FlatList integration. onEndReached triggers pagination automatically.

## Mutations with Cache Invalidation

```typescript
// hooks.ts
export const useCreateProduct = (options?) => {
  const queryClient = useQueryClient();
  const fetch = useFetch();

  return useMutation({
    mutationFn: (params: CreateProductRequest) => {
      return createProduct(params, queryKey, fetch);
    },
    onSuccess: () => {
      // Invalidate list to refetch
      queryClient.invalidateQueries({ queryKey: PRODUCT_QUERY_KEYS.all });
    },
    ...options,
  });
};
```

**Why:** Proper cache invalidation keeps UI in sync with server state after mutations.

## Performance Optimization

Configure appropriate stale times based on data volatility.

```typescript
// List data changes frequently
useQuery({
  queryKey: PRODUCT_QUERY_KEYS.list(params),
  queryFn: () => getProducts(params, queryKey, fetch),
  staleTime: 5 * 60 * 1000, // 5 minutes
  gcTime: 10 * 60 * 1000, // 10 minutes
});

// Detail data is more stable
useQuery({
  queryKey: PRODUCT_QUERY_KEYS.detail(id),
  queryFn: () => getProduct({ id }, queryKey, fetch),
  staleTime: 10 * 60 * 1000, // 10 minutes
  gcTime: 15 * 60 * 1000, // 15 minutes
});
```

**Why:** Proper cache timing reduces unnecessary API calls while keeping data fresh.

## Testing

Mock service functions and test hooks in isolation.

```typescript
import { renderHook, waitFor } from '@testing-library/react-native';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useGetProducts } from './hooks';
import * as service from './service';

jest.mock('./service');
jest.mock('@/hooks/data-access', () => ({
  useFetch: () => jest.fn(),
  useLocalizeParams: () => ({ locale: 'en' }),
}));

const mockGetProducts = service.getProducts as jest.MockedFunction<typeof service.getProducts>;

test('returns products data', async () => {
  const mockResponse = {
    data: [{ id: '1', name: 'Test Product', price: 2999 }],
    pagination: { hasMore: false },
  };

  mockGetProducts.mockResolvedValue(mockResponse);

  const queryClient = new QueryClient({ defaultOptions: { queries: { retry: false } } });
  const wrapper = ({ children }) => (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );

  const { result } = renderHook(() => useGetProducts({ limit: 10 }), { wrapper });

  await waitFor(() => expect(result.current.isSuccess).toBe(true));
  expect(result.current.data).toEqual(mockResponse);
});
```

**Why:** Mocking enables testing without actual API calls and validates query structure.

## Common Mistakes

### ❌ Don't Use REST When GraphQL Exists
```typescript
// GraphQL query exists - use that instead
const products = useQuery({
  queryKey: ['products'],
  queryFn: fetchProductsRest, // Wrong
});
```

### ❌ Don't Call Fetch Directly in Components
```typescript
useEffect(() => {
  fetch('/api/products').then(setData); // Wrong - no caching, no tracing
}, []);
```

### ❌ Don't Use Inconsistent Query Keys
```typescript
// Inconsistent structure
const queryKey = ['products', id, 'random'];
```

### ✅ Do Use Custom Hooks with Standard Patterns
```typescript
const { data, isLoading, error } = useGetProducts(params);
```

## Quick Reference

**Query Keys:**
```typescript
const KEYS = {
  all: ['domain'] as const,
  list: (params?) => ['domain', 'list', params] as const,
  detail: (id) => ['domain', 'detail', id] as const,
} as const;
```

**Standard Hook:**
```typescript
export const useGet = (params, options?) => {
  const fetch = useFetch();
  return useQuery({
    queryKey: KEYS.list(params),
    queryFn: () => service(params, queryKey, fetch),
    ...options,
  });
};
```

**Infinite Query:**
```typescript
useInfiniteQuery({
  queryKey: [...KEYS.list(params), 'infinite'],
  queryFn: ({ pageParam }) => service({ ...params, cursor: pageParam }, queryKey, fetch),
  initialPageParam: undefined,
  getNextPageParam: (lastPage) => lastPage.pagination.nextCursor,
});
```

## Additional Resources

For detailed examples and patterns, see:
- [references/examples.md](references/examples.md) - Real API hook implementations
- [references/patterns.md](references/patterns.md) - REST API patterns and anti-patterns
