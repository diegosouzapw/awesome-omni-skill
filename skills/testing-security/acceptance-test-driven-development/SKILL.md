---
name: acceptance-test-driven-development
description: Write acceptance tests before unit tests to ensure you're building the right thing
globs: "**/*.ts,**/*.tsx,**/*.js,**/*.jsx"
---

# Acceptance Test-Driven Development (ATDD)

## Workflow

ATDD extends TDD by adding an outer acceptance test loop:

1. **Write an acceptance test** that describes the desired behavior from the user's perspective
2. **Run it — it should fail** (the feature doesn't exist yet)
3. **Enter the TDD inner loop:**
   - Write a failing unit test (RED)
   - Write minimal code to pass (GREEN)
   - Refactor (REFACTOR)
   - Repeat until the acceptance test passes
4. **Verify the acceptance test passes** — the feature is complete

## Acceptance Test Patterns

Acceptance tests should be named with these patterns:
- `*.acceptance.test.ts` — acceptance/integration tests
- `*.e2e.test.ts` — end-to-end tests

## Key Principles

- **Acceptance tests describe WHAT**, not HOW — they test observable behavior
- **Unit tests describe HOW** — they test internal mechanics
- **Write acceptance tests in the language of the user/stakeholder**
- **One acceptance test per user story or feature**
- **Multiple unit tests per acceptance test** — the inner TDD loop

## Example Workflow

```
Feature: User registration

1. Write acceptance test: user-registration.acceptance.test.ts
   - Test: "User can register with email and password"
   - Test: "Registration fails with duplicate email"
   - Run → FAIL (no registration endpoint)

2. TDD inner loop:
   a. Unit test: validate-email.test.ts → RED → implement → GREEN
   b. Unit test: hash-password.test.ts → RED → implement → GREEN
   c. Unit test: create-user.test.ts → RED → implement → GREEN
   d. Unit test: registration-handler.test.ts → RED → implement → GREEN

3. Run acceptance test → PASS → Feature complete!
```

## Anti-Patterns to Avoid

- ❌ Writing unit tests without an acceptance test that frames the feature
- ❌ Writing acceptance tests that test implementation details
- ❌ Skipping the acceptance test because "it's a small feature"
- ❌ Writing all acceptance tests upfront (write them one feature at a time)
