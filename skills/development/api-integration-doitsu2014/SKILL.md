---
name: api-integration
description: Work with REST and GraphQL APIs, authentication, API configuration, and data fetching. Use when implementing API calls, debugging network requests, setting up Apollo Client, or handling authentication.
---

# API Integration

## Overview
This skill helps you work with API integrations in the admin_side application. The project uses both REST API and GraphQL with Apollo Client, secured by Keycloak authentication.

## Key Files

### API Configuration
- `src/config/api.config.ts` - REST API configuration and authenticated fetch
- `src/infrastructure/graphQL/graphql-client.ts` - Apollo Client setup
- `src/infrastructure/utilities.auth.ts` - Authentication utilities
- `src/infrastructure/utilities.ts` - General utility functions

### Authentication
- `src/auth/AuthContext.tsx` - Auth context provider
- `src/auth/ProtectedRoute.tsx` - Route protection
- `src/auth/keycloak.ts` - Keycloak configuration

### Data Access
- `src/infrastructure/das/categories.das.ts` - Category data access service
- `src/infrastructure/graphQL/queries/categories/` - GraphQL queries

## REST API Setup

### Configuration
```typescript
// src/config/api.config.ts
export const getApiUrl = (path: string) => {
  const baseUrl = import.meta.env.PUBLIC_REST_API_URL;
  return `${baseUrl}${path}`;
};
```

### Authenticated Requests
```typescript
export const authenticatedFetch = async (
  url: string,
  token: string | null,
  options: RequestInit = {}
): Promise<Response> => {
  const headers = {
    'Content-Type': 'application/json',
    ...(token && { Authorization: `Bearer ${token}` }),
    ...options.headers,
  };

  return fetch(url, {
    ...options,
    headers,
  });
};
```

### Environment Variables
REST API URL is configured via environment variable:
```env
PUBLIC_REST_API_URL=http://localhost:4000/api
```

## GraphQL Integration

### Apollo Client Setup
```typescript
// src/infrastructure/graphQL/graphql-client.ts
import { ApolloClient, InMemoryCache, createHttpLink } from '@apollo/client';
import { setContext } from '@apollo/client/link/context';

const httpLink = createHttpLink({
  uri: import.meta.env.PUBLIC_GRAPHQL_API_URL,
});

const authLink = setContext((_, { headers }) => {
  const token = getAuthToken(); // Get from storage
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : '',
    },
  };
});

export const graphqlClient = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache(),
});
```

### Writing GraphQL Queries
```typescript
// src/infrastructure/graphQL/queries/categories/get-categories.ts
import { gql } from '@apollo/client';

export const GET_CATEGORIES = gql`
  query GetCategories {
    categories {
      id
      name
      slug
      description
      posts {
        id
        title
      }
    }
  }
`;
```

### Using GraphQL Queries
```typescript
import { graphqlClient } from '@/infrastructure/graphQL/graphql-client';
import { GET_CATEGORIES } from '@/infrastructure/graphQL/queries/categories/get-categories';

const { data, loading, error } = await graphqlClient.query({
  query: GET_CATEGORIES,
  fetchPolicy: 'network-only', // or 'cache-first', 'cache-only'
});
```

## Authentication

### Keycloak Integration
The app uses Keycloak for authentication:
```typescript
// src/auth/keycloak.ts
import Keycloak from 'keycloak-js';

const keycloak = new Keycloak({
  url: import.meta.env.PUBLIC_KEYCLOAK_URL,
  realm: import.meta.env.PUBLIC_KEYCLOAK_REALM,
  clientId: import.meta.env.PUBLIC_KEYCLOAK_CLIENT_ID,
});

export default keycloak;
```

### Auth Context
```typescript
// Usage in components
import { useAuth } from '@/auth/AuthContext';

function MyComponent() {
  const { token, isAuthenticated, login, logout } = useAuth();

  // Use token for API calls
  const response = await authenticatedFetch(url, token);
}
```

### Protected Routes
```typescript
import ProtectedRoute from '@/auth/ProtectedRoute';

<ProtectedRoute>
  <AdminDashboard />
</ProtectedRoute>
```

## API Endpoints Reference

### Blog Posts
- `GET /posts` - List all posts
- `GET /posts/:id` - Get single post
- `POST /posts` - Create new post
- `PUT /posts/:id` - Update post
- `DELETE /posts/:id` - Delete post

### Categories
- `GET /categories` - List all categories
- `GET /categories/:id` - Get single category
- `POST /categories` - Create new category
- `PUT /categories/:id` - Update category
- `DELETE /categories/:id` - Delete category

## Common Patterns

### Fetching Data
```typescript
const [data, setData] = useState(null);
const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);
const { token } = useAuth();

useEffect(() => {
  const fetchData = async () => {
    setLoading(true);
    try {
      const response = await authenticatedFetch(
        getApiUrl('/posts'),
        token
      );
      if (response.ok) {
        const result = await response.json();
        setData(result.data);
      } else {
        throw new Error('Failed to fetch');
      }
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  fetchData();
}, [token]);
```

### Creating Data
```typescript
const handleCreate = async (formData) => {
  try {
    const response = await authenticatedFetch(
      getApiUrl('/posts'),
      token,
      {
        method: 'POST',
        body: JSON.stringify(formData),
      }
    );

    if (response.ok) {
      const result = await response.json();
      navigate('/admin/posts');
    } else {
      const error = await response.json();
      console.error('Create failed:', error);
    }
  } catch (err) {
    console.error('Network error:', err);
  }
};
```

### Updating Data
```typescript
const handleUpdate = async (id, formData) => {
  try {
    const response = await authenticatedFetch(
      getApiUrl(`/posts/${id}`),
      token,
      {
        method: 'PUT',
        body: JSON.stringify({
          ...formData,
          rowVersion, // Include for optimistic locking
        }),
      }
    );

    if (response.ok) {
      navigate('/admin/posts');
    }
  } catch (err) {
    console.error('Update error:', err);
  }
};
```

## Error Handling

### REST API Errors
```typescript
const response = await authenticatedFetch(url, token);

if (!response.ok) {
  if (response.status === 401) {
    // Unauthorized - redirect to login
    logout();
  } else if (response.status === 404) {
    // Not found
    console.error('Resource not found');
  } else {
    // Other errors
    const error = await response.json();
    console.error('API error:', error);
  }
}
```

### GraphQL Errors
```typescript
const { data, error } = await graphqlClient.query({
  query: GET_CATEGORIES,
});

if (error) {
  console.error('GraphQL error:', error.message);
  // Handle network errors, GraphQL errors, etc.
}
```

## Best Practices

1. **Always use authenticatedFetch** for REST API calls
2. **Include Bearer token** in all authenticated requests
3. **Handle loading states** appropriately
4. **Catch and display errors** to users
5. **Use environment variables** for API URLs
6. **Implement retry logic** for failed requests when appropriate
7. **Cache GraphQL queries** strategically
8. **Validate responses** before using data
9. **Handle 401 errors** by redirecting to login
10. **Include rowVersion** in update operations

## Debugging API Issues

1. **Check Network Tab**: Verify request/response in browser DevTools
2. **Validate Token**: Ensure auth token is valid and not expired
3. **Check CORS**: Verify server allows requests from your origin
4. **Review Headers**: Confirm Authorization header is set correctly
5. **Test Endpoints**: Use Postman/curl to test API directly
6. **Check Environment Variables**: Verify API URLs are correct
7. **Review Apollo Cache**: Clear cache if seeing stale data
8. **Monitor Console**: Check for error messages and warnings

## Environment Variables

Required environment variables:
```env
# REST API
PUBLIC_REST_API_URL=http://localhost:4000/api

# GraphQL API
PUBLIC_GRAPHQL_API_URL=http://localhost:4000/graphql

# Keycloak
PUBLIC_KEYCLOAK_URL=http://localhost:8080
PUBLIC_KEYCLOAK_REALM=your-realm
PUBLIC_KEYCLOAK_CLIENT_ID=your-client-id
```

## Example Workflow

When implementing a new API integration:

1. Review existing API configuration
2. Check authentication setup
3. Define TypeScript types for request/response
4. Implement API call with error handling
5. Add loading states to UI
6. Test with valid and invalid data
7. Handle edge cases (network errors, timeouts)
8. Add appropriate user feedback
