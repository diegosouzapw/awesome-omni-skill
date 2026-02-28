---
name: auth0
description: Expert for Auth0 integration across Go backends and React frontends. Use when setting up OIDC authentication, validating JWTs in Go, or implementing Auth0 React SDK patterns.
---
# Auth0 Integration Skill

This skill provides standard patterns for integrating Auth0 into a polyglot stack. It focuses on secure OIDC flows, JWT verification in Go, and efficient React state management.

## Architectural Standards

### 1. Go Backend Integration (JWT Validation)
*   **Verification:** Use `auth0/go-jwt-middleware` and `form3tech-oss/jwt-go`.
*   **JWKS Cache:** Implement a caching mechanism for public keys from the `.well-known/jwks.json` endpoint to reduce latency.
*   **Claims Mapping:** Map Auth0's `https://yourdomain.com/roles` custom claims to internal Go RBAC structures. Validate the `aud` (Audience) and `iss` (Issuer) claims strictly.

### 2. React Frontend Integration
*   **SDK:** Use `@auth0/auth0-react`. Wrap the application root in `Auth0Provider`.
*   **Silent Refresh:** Implement `getAccessTokenSilently` with `useAuth0`. Use `ignoreCache: true` only when a fresh token is absolutely required for mutation.
*   **Multi-tenant:** Handle `organization` parameters in the login flow if using Auth0 Organizations.

### 3. TanStack Query Integration
```typescript
const { getAccessTokenSilently, isAuthenticated } = useAuth0();

const useSecureQuery = (key: any[], fetcher: (token: string) => Promise<any>) => {
  return useQuery({
    queryKey: key,
    queryFn: async () => {
      if (!isAuthenticated) throw new Error("Not authenticated");
      const token = await getAccessTokenSilently();
      return fetcher(token);
    },
    enabled: isAuthenticated,
  });
};
```

### 4. Advanced Security
*   **PKCE:** Always ensure Authorization Code Flow with PKCE is enabled for SPAs.
*   **CORS & Redirects:** Strictly white-list only production and trusted dev URLs (e.g., `http://localhost:5173`).
*   **MFA:** Handle "MFA Required" errors in the frontend by prompting the user to complete the Auth0 MFA challenge.

## Interaction Protocol
*   **Input:** Auth0 Domain, Client ID, Audience, and architectural requirements.
*   **Output:** Go middleware logic and React TanStack Query integration code.

**Tag**: Start your response with `[AUTH0-INTEGRATION]`.
