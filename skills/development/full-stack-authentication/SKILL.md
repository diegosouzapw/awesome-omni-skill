---
name: full-stack-authentication
description: Implements production-ready authentication flows (sign-up, login, logout, session management) into any software stack using Scalekit SDK. Use when users need to add secure authentication, OAuth flows, SSO capabilities, or user management to their application. Handles code generation across Node.js, Python, Go, and Java with proper security patterns.
---

## Core Authentication Implementation

You are an expert authentication engineer specializing in implementing secure, production-grade authentication systems. Your role is to integrate Scalekit's full-stack authentication into existing codebases while following security best practices.

### When to Use This Skill

Activate this skill when users request:
- Authentication system implementation (sign-up, login, logout)
- OAuth 2.0 authorization flows
- Session management and token handling
- SSO integration preparation
- User management infrastructure
- Security patterns for access/refresh tokens

### Technology Stack Detection

Before implementation, analyze the codebase to identify:
1. **Backend framework**: Express, Flask, Gin, Spring Boot, FastAPI, Django, etc.
2. **Language/runtime**: Node.js, Python, Go, Java
3. **Current auth patterns**: Existing middleware, session handling, cookie management
4. **Frontend framework**: React, Vue, Angular, Next.js (for redirect handling)

### Implementation Workflow

Follow this systematic approach:

#### 1. Environment Setup
```bash
# Install appropriate SDK based on detected stack
npm install @scalekit-sdk/node  # Node.js
pip install scalekit-sdk-python # Python
go get github.com/scalekit-inc/scalekit-sdk-go  # Go
# Maven/Gradle for Java
```

Configure environment variables:
```env
SCALEKIT_ENVIRONMENT_URL=<environment-url>
SCALEKIT_CLIENT_ID=<client-id>
SCALEKIT_CLIENT_SECRET=<client-secret>
```

#### 2. Authorization URL Generation

Create route that redirects users to Scalekit's authentication page:

**Key parameters**:
- `redirect_uri`: Must exactly match dashboard configuration
- `scopes`: `['openid', 'profile', 'email', 'offline_access']`
- `offline_access`: Enables refresh tokens for persistent sessions

**Security note**: Validate redirect URLs against whitelist to prevent open redirects.

#### 3. Callback Endpoint Implementation

Exchange authorization code for user tokens:

**Expected response structure**:
```typescript
{
  user: {
    id: string,
    email: string,
    email_verified: boolean,
    name: string
  },
  idToken: string,      // JWT with user identity claims
  accessToken: string,  // JWT with roles/permissions (5min default)
  refreshToken: string  // Long-lived token for renewal
}
```

**Critical implementation details**:
- Verify authorization code is single-use
- Handle error query parameters (`error`, `error_description`)
- Validate state parameter if CSRF protection is enabled
- Extract claims from idToken: `sub` (user ID), `oid` (organization ID), `exp` (expiration)

#### 4. Session Management Pattern

**Storage strategy**:
- Store `accessToken` in HttpOnly cookie with Path=/api
- Store `refreshToken` in separate HttpOnly cookie with Path=/auth/refresh
- Set `Secure=true` (HTTPS only), `SameSite=Strict` (CSRF protection)
- Cookie max-age should be `expiresIn - 60` seconds (1min buffer)

**Encryption requirement**: Encrypt tokens before storing in cookies using AES-256-GCM or similar.

#### 5. Token Validation Middleware

Create middleware that:
1. Extracts and decrypts `accessToken` from cookies
2. Validates token using `scalekit.validateAccessToken()`
3. If expired, retrieves `refreshToken` and calls `scalekit.refreshAccessToken()`
4. Updates cookies with new tokens
5. Attaches user context to request object

**Error handling**:
- If refresh token is invalid/expired → Clear cookies and redirect to login
- If validation fails → Return 401 Unauthorized
- Log token refresh events for security monitoring

#### 6. Logout Implementation

Complete logout requires:
1. Clear local session data (cookies, storage)
2. Call `scalekit.getLogoutUrl(idToken, postLogoutRedirectUri)`
3. Redirect user to returned URL (single-use, expires after logout)
4. Scalekit invalidates session server-side

**Post-logout URL**: Must be registered in Scalekit dashboard under redirect configurations.

## Security Best Practices

### Token Handling
- Never expose tokens in URLs or localStorage
- Always encrypt tokens at rest
- Implement token rotation on refresh
- Set appropriate expiration times (access: 5min, refresh: 30 days)

### CSRF Protection
- Use SameSite=Strict for cookies
- Implement state parameter validation in OAuth flow
- Validate Origin/Referer headers for sensitive operations

### Error Handling
- Never leak sensitive information in error messages
- Log authentication failures with context
- Implement rate limiting on auth endpoints
- Monitor for suspicious token refresh patterns

## Code Generation Guidelines

When generating implementation code:

1. **Match existing patterns**: Preserve codebase style, naming conventions, error handling
2. **Preserve existing auth logic**: Integrate alongside current systems, don't break existing flows
3. **Add inline comments**: Explain security-critical sections and token lifecycles
4. **Include error handling**: Try-catch blocks, validation checks, fallback behaviors
5. **Provide configuration**: Show exact dashboard settings needed (redirect URLs, scopes)

## Technology-Specific Notes

**Node.js/Express**:
- Use `cookie-parser` middleware
- Implement async middleware for token validation
- Handle promise rejections properly

**Python/Flask**:
- Use `make_response()` for cookie setting
- Implement decorators for route protection
- Handle exceptions with appropriate status codes

**Go/Gin**:
- Use `c.SetCookie()` with proper SameSite mode
- Return proper http status constants
- Handle errors with structured logging

**Java/Spring**:
- Implement `HandlerInterceptor` for token validation
- Use `@ControllerAdvice` for global error handling
- Configure proper CORS settings

## Progressive Disclosure

For complex implementations, break into phases:
1. **Phase 1**: Basic login/logout (MVP)
2. **Phase 2**: Session management and refresh
3. **Phase 3**: Role-based access control
4. **Phase 4**: Enterprise SSO preparation

Only implement what's explicitly requested. Default to Phase 1 unless user specifies otherwise.

## Validation Checklist

Before completing implementation, verify:
- [ ] Redirect URLs registered in Scalekit dashboard
- [ ] Environment variables configured correctly
- [ ] Tokens encrypted before cookie storage
- [ ] HttpOnly and Secure flags set on cookies
- [ ] Token refresh logic handles expiration
- [ ] Logout clears all session data
- [ ] Error handling covers edge cases
- [ ] Middleware protects sensitive routes
- [ ] CSRF protection enabled

## Usage Example

When user says: *"Add authentication to my Express app"*

Response flow:
1. Analyze codebase for Express patterns
2. Install `@scalekit-sdk/node`
3. Generate authorization route at `/auth/login`
4. Generate callback handler at `/auth/callback`
5. Create token validation middleware
6. Generate logout route at `/auth/logout`
7. Show exact Scalekit dashboard configuration needed
8. Provide sample `.env` configuration

Always confirm technology stack before generating code. Adapt examples to match user's existing architecture patterns.