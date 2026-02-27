---
name: Authorization Matrix
description: Build and verify an authorization matrix mapping roles to endpoints — detect unprotected routes, missing ownership checks, and scope misconfigurations
category: security
version: 1.0.0
triggers:
  - route-change
  - middleware-change
  - auth-change
globs: "**/cmd/api/main.go,**/middleware/**,**/handler/**"
---

# Authorization Matrix Skill

Build a complete authorization matrix from the codebase and verify every endpoint has appropriate authentication, authorization, and ownership checks.

## Trigger Conditions
- Route configuration changes
- Auth middleware changes
- New handlers are added
- User invokes with "auth matrix" or "authorization-matrix"

## Input Contract
- **Required:** Path to route configuration (main.go or router files)
- **Required:** Path to middleware directory
- **Optional:** Path to handler files for ownership check verification

## Output Contract
- Complete route → auth requirement matrix
- List of unprotected routes (missing auth middleware)
- List of routes missing account ownership verification
- Comparison against the whitelist in rule 121

## Tool Permissions
- **Read:** Route config, middleware, handler files
- **Write:** None (read-only analysis)
- **Search:** Grep for `Use(middleware.Auth`, `GetUserID`, `RequireScope`, `AccountOwnership`

## Execution Steps

1. **Extract all routes**: Parse route registration to build complete endpoint list with HTTP method and path
2. **Identify auth middleware**: Find which route groups use auth middleware
3. **Check whitelist**: Compare unauthenticated routes against the whitelist in rule 121
4. **Verify ownership**: For account-scoped routes (`/accounts/:id/*`), verify ownership middleware is applied
6. **Report**: Produce authorization matrix with pass/fail per check

## Success Criteria
- All routes outside the whitelist are authenticated
- All account-scoped routes verify ownership
- All financial endpoints require appropriate scopes
- No route bypasses auth through misconfiguration

## References
- `.cursor/rules/121-route-auth-enforcement.mdc`
- `.cursor/rules/042-security-authorization.mdc`
