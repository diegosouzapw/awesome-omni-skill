---
name: smoke-test-authenticated-api-routes
description: Create a minimal smoke-test plan for authenticated API routes (happy path, one negative case, and persistence checks).
---

# Smoke Test Authenticated API Routes

## Purpose
Provide a repeatable approach to quickly validate that protected API endpoints work end-to-end: authentication, request handling, and persistence side effects.

## When to use
Use this skill when you are:
- Testing a newly added endpoint
- Verifying endpoint behavior after refactors
- Debugging auth-related failures (`401`, `403`)
- Confirming that POST/PUT/PATCH/DELETE endpoints create or update the right records


Avoid using this skill when:
- You need a full end-to-end test execution record across many endpoints, or you need to produce evidence for a release gate.
- You primarily need a code/implementation review rather than a smoke test plan and quick validation run.

## Inputs
- Endpoint(s): method + URL path
- Authentication method:
  - cookie-based session
  - bearer token
  - API key
  - mock/dev bypass (development only)
- Required request body/query/params
- Expected response shape and status code
- Expected persistence side effects (tables/collections/records)

## Outputs
- A minimal smoke test plan per endpoint:
  - valid request
  - invalid request (one representative)
  - expected status codes and response shapes
- Verification notes for persistence side effects
- A short debug report if a test fails (observations + likely causes)

## Core rules
- Smoke tests SHOULD prioritize the happy path and core behavior.
- Tests MUST NOT use real production credentials.
- Any "mock auth" bypass MUST be limited to non-production environments.
- Persisted side effects MUST be verified for write endpoints.

## Steps
1. **Locate the endpoint contract**
   - method + path
   - request fields (required/optional)
   - response shape

2. **Prepare authentication**
   - Obtain a valid session/token for a test user, or use a non-production auth bypass if available.
   - Record the user/role used for the test.

3. **Run a valid request**
   - Use an HTTP client (curl, Postman, a test harness, or automated integration test).
   - Record status code and response body.

4. **Verify side effects (write endpoints)**
   - Query the database or inspect logs to confirm expected changes.
   - Verify idempotency expectations if the endpoint is retried.

5. **Run one invalid request**
   - Example: missing required field, invalid enum, or invalid identifier.
   - Verify validation status code and error shape.

6. **If the request fails, triage by class**
   - `401` unauthorized: missing/invalid auth
   - `403` forbidden: user lacks permission
   - `404` not found: wrong URL/prefix/route registration
   - `5xx` server error: unhandled exception or downstream dependency

## Verification

- [ ] Status codes match the contract
- [ ] Success response contains required fields
- [ ] Validation errors return stable error codes and details
- [ ] Persistence side effects match expectations
- [ ] No secrets or tokens were logged or committed to docs
- [ ] At least one valid and one invalid request were tested

## Boundaries

- MUST NOT use real production credentials in tests
- MUST NOT enable mock auth bypasses in production environments
- MUST NOT skip persistence verification for write endpoints
- MUST NOT log or commit secrets/tokens to documentation
- SHOULD NOT test only happy paths (include at least one negative case)
- SHOULD NOT assume auth context without verifying it is attached

## Included assets
- Templates: `./templates/` includes a route test matrix and a JSON test spec schema.
- Examples: `./examples/` includes curl patterns for cookie and bearer auth (placeholders only).
