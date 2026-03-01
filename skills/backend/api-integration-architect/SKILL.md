---
name: api-integration-architect
description: Expert in API integration with React Query, SWR, RTK Query, REST/GraphQL clients, WebSocket real-time, optimistic updates, caching strategies, and error handling
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Task
---

# API Integration Architect

Expert skill for integrating APIs in React applications. Specializes in React Query, SWR, GraphQL, WebSockets, caching strategies, and optimistic updates.

## Core Capabilities

### 1. Data Fetching Libraries
- **React Query (TanStack Query)**: Powerful async state management
- **SWR**: Stale-while-revalidate strategy
- **RTK Query**: Redux Toolkit integration
- **Apollo Client**: GraphQL client
- **Axios/Fetch**: HTTP clients

### 2. REST API Integration
- **GET Requests**: Fetch data
- **POST/PUT/PATCH**: Create/update data
- **DELETE**: Remove data
- **Query Parameters**: Filtering, pagination
- **Headers**: Authentication, content-type
- **Error Handling**: Network errors, HTTP errors

### 3. GraphQL Integration
- **Queries**: Fetch data with GraphQL
- **Mutations**: Modify data
- **Subscriptions**: Real-time updates
- **Fragments**: Reusable query parts
- **Code Generation**: Type-safe queries
- **Caching**: Normalized cache

### 4. Real-Time Communication
- **WebSocket**: Bidirectional communication
- **Server-Sent Events**: One-way server push
- **Polling**: Interval-based updates
- **Long Polling**: Efficient polling
- **Socket.io**: WebSocket library

### 5. Caching & Optimization
- **Cache Strategies**: Fresh, stale, refetch
- **Background Refetch**: Update in background
- **Cache Invalidation**: Smart cache updates
- **Prefetching**: Load before needed
- **Pagination**: Infinite scroll, load more
- **Deduplication**: Avoid duplicate requests

### 6. Optimistic Updates
- **Immediate UI**: Update before server response
- **Rollback**: Undo on error
- **Pessimistic Updates**: Wait for server
- **Concurrent Updates**: Handle conflicts
- **Offline Support**: Queue mutations

## React Query Setup

```typescript
// queryClient.ts
import { QueryClient } from '@tanstack/react-query'

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      cacheTime: 10 * 60 * 1000, // 10 minutes
      refetchOnWindowFocus: false,
      retry: 1,
    },
  },
})

// App.tsx
import { QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  )
}
```

## API Client

```typescript
// api/client.ts
import axios from 'axios'

const apiClient = axios.create({
  baseURL: process.env.REACT_APP_API_URL,
  headers: {
    'Content-Type': 'application/json',
  },
})

// Request interceptor
apiClient.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token')
    if (token) {
      config.headers.Authorization = `Bearer ${token}`
    }
    return config
  },
  (error) => Promise.reject(error)
)

// Response interceptor
apiClient.interceptors.response.use(
  (response) => response.data,
  (error) => {
    if (error.response?.status === 401) {
      // Handle unauthorized
      window.location.href = '/login'
    }
    return Promise.reject(error)
  }
)

export default apiClient
```

## React Query Patterns

```typescript
// hooks/useUser.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import apiClient from '@/api/client'

interface User {
  id: string
  name: string
  email: string
}

// Fetch user
export function useUser(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => apiClient.get<User>(`/users/${userId}`),
    enabled: !!userId,
  })
}

// Update user
export function useUpdateUser() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: (data: Partial<User> & { id: string }) =>
      apiClient.put(`/users/${data.id}`, data),

    // Optimistic update
    onMutate: async (newUser) => {
      await queryClient.cancelQueries({ queryKey: ['user', newUser.id] })
      const previousUser = queryClient.getQueryData(['user', newUser.id])
      queryClient.setQueryData(['user', newUser.id], newUser)
      return { previousUser }
    },

    onError: (err, newUser, context) => {
      queryClient.setQueryData(['user', newUser.id], context?.previousUser)
    },

    onSettled: (data, error, variables) => {
      queryClient.invalidateQueries({ queryKey: ['user', variables.id] })
    },
  })
}
```

## GraphQL with Apollo

```typescript
// apollo-client.ts
import { ApolloClient, InMemoryCache, createHttpLink } from '@apollo/client'
import { setContext } from '@apollo/client/link/context'

const httpLink = createHttpLink({
  uri: process.env.REACT_APP_GRAPHQL_URL,
})

const authLink = setContext((_, { headers }) => {
  const token = localStorage.getItem('token')
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : '',
    },
  }
})

export const apolloClient = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache(),
})

// Usage
import { useQuery, useMutation, gql } from '@apollo/client'

const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) {
      id
      name
      email
    }
  }
`

function UserProfile({ userId }: { userId: string }) {
  const { data, loading, error } = useQuery(GET_USER, {
    variables: { id: userId },
  })

  if (loading) return <div>Loading...</div>
  if (error) return <div>Error: {error.message}</div>

  return <div>{data.user.name}</div>
}
```

## WebSocket Integration

```typescript
// hooks/useWebSocket.ts
import { useEffect, useState, useRef } from 'react'

export function useWebSocket<T>(url: string) {
  const [data, setData] = useState<T | null>(null)
  const [error, setError] = useState<Event | null>(null)
  const ws = useRef<WebSocket | null>(null)

  useEffect(() => {
    ws.current = new WebSocket(url)

    ws.current.onmessage = (event) => {
      setData(JSON.parse(event.data))
    }

    ws.current.onerror = (error) => {
      setError(error)
    }

    return () => {
      ws.current?.close()
    }
  }, [url])

  const send = (message: any) => {
    ws.current?.send(JSON.stringify(message))
  }

  return { data, error, send }
}
```

## Infinite Scroll

```typescript
// hooks/useInfiniteScroll.ts
import { useInfiniteQuery } from '@tanstack/react-query'
import apiClient from '@/api/client'

export function useInfinitePosts() {
  return useInfiniteQuery({
    queryKey: ['posts'],
    queryFn: ({ pageParam = 1 }) =>
      apiClient.get(`/posts?page=${pageParam}`),
    getNextPageParam: (lastPage, pages) => {
      return lastPage.hasMore ? pages.length + 1 : undefined
    },
  })
}

// Component
function PostList() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfinitePosts()

  return (
    <div>
      {data?.pages.map((page) =>
        page.posts.map((post) => <Post key={post.id} {...post} />)
      )}

      {hasNextPage && (
        <button onClick={() => fetchNextPage()} disabled={isFetchingNextPage}>
          {isFetchingNextPage ? 'Loading...' : 'Load More'}
        </button>
      )}
    </div>
  )
}
```

## Best Practices

- Type-safe APIs with TypeScript
- Error boundaries for API errors
- Loading and error states
- Optimistic updates for better UX
- Cache invalidation strategies
- Request deduplication
- Retry failed requests
- Authentication handling

## When to Use This Skill

Use when you need to:
- Integrate REST or GraphQL APIs
- Set up React Query or SWR
- Implement real-time updates
- Add optimistic updates
- Handle pagination/infinite scroll
- Manage API caching
- Build offline-capable apps

## Output Format

Provide:
1. **API Client Setup**: Axios/Fetch configuration
2. **Query Hooks**: Type-safe data fetching
3. **Mutation Hooks**: Data modification
4. **Caching Strategy**: How data is cached
5. **Error Handling**: Error boundaries and fallbacks
6. **Testing**: API mocking and tests
