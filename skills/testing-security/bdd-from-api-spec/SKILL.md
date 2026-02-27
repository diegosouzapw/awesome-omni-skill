---
name: BDD from API Spec
description: Generate Gherkin BDD feature files from API handler definitions — map endpoints to executable business scenarios
category: testing
version: 1.0.0
triggers:
  - new-handler
  - endpoint-change
  - bdd-request
globs: "**/handler/**,**/features/**"
---

# BDD from API Spec Skill

Automatically generate Gherkin feature files from API handler definitions, producing executable business specifications for each endpoint.

## Trigger Conditions
- New handler file is created
- Existing handler endpoint is modified
- BDD scenarios are requested
- User invokes with "write BDD" or "bdd-from-api-spec"

## Input Contract
- **Required:** Path to handler files
- **Optional:** OpenAPI spec for additional context

## Output Contract
- Gherkin `.feature` files per handler/resource
- Scenario coverage: happy path, error cases, edge cases, auth checks
- Step definition stubs (Go/Godog format)

## Tool Permissions
- **Read:** Handler files, DTO files, service interfaces, OpenAPI spec
- **Write:** Feature files in `features/` directory, step definition stubs
- **Search:** Grep for handler methods, route registrations

## Execution Steps

1. **Extract endpoints**: Parse handler files for all endpoint methods with HTTP method and path
2. **Identify scenarios**: For each endpoint, generate scenarios:
   - Happy path (valid request, successful response)
   - Authentication failure (missing/invalid token)
   - Authorization failure (wrong user/scope)
   - Validation failure (invalid input)
   - Business rule failure (e.g., insufficient funds, account frozen)
   - Idempotency (duplicate request returns same result)
3. **Generate Gherkin**: Write `.feature` files with Given/When/Then format
4. **Generate step stubs**: Create step definition file stubs for Godog
