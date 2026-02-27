---
name: airtight-plans
description: Write structured multi-step implementation plans in markdown format. Plans use numbered steps with clear titles and detailed instructions. Use when asked to create an implementation plan, development roadmap, or multi-step task breakdown.
---

# Writing Implementation Plans

Create structured plans with numbered steps that can be executed by developers or automated agents.

## Step Format

```markdown
## Step N: Title

Instructions for this step.
```

**Rules:**
- Format: `## Step N: Title` (N = positive integer)
- Sequential numbering: 1, 2, 3...
- No duplicate step numbers
- Descriptive titles indicating what the step accomplishes

## Step Structure

Each step has four sections:

### Status Quo
What exists at the start of this step. This follows logically from the initial state or previous steps. The implementer can rely on these conditions but should verify them before proceeding.

### Objectives
The business requirements and conceptual goals for this step. What do we want to achieve? How does this fit into the overall plan?

### Tech Notes
Implementation hints, code references, architectural decisions, or technical details that help the implementer. Optional if the implementation is straightforward.

### Acceptance Criteria
Verification steps the implementer performs to confirm the task is complete. Not restated requirements—concrete checks like running tests, verifying behavior, or inspecting output.

**Bad** (restates requirements):
```
### Acceptance Criteria
- User model has all fields
- Password is hashed
```

**Good** (verification steps):
```
### Acceptance Criteria
- `just test` passes with new model tests
- Create a user in test, verify passwordHash !== plaintext password
- Test confirms duplicate email throws constraint error
```

## Example

```markdown
# Feature: User Authentication

Implements JWT-based authentication for the API.

## Step 1: Create user model

### Status Quo
- Database is configured with migrations support
- No user-related tables exist yet

### Objectives
Create the foundational User model to store account credentials. This enables all subsequent authentication features.

### Tech Notes
- Use UUID for id to avoid enumeration attacks
- Store passwordHash, never plain passwords
- Consider adding `deletedAt` for soft deletes if needed later

### Acceptance Criteria
- `just test` passes including new user model tests
- Unit test verifies passwordHash !== plaintext password
- Unit test confirms duplicate email throws constraint error

## Step 2: Implement registration endpoint

### Status Quo
- User model exists with email uniqueness constraint
- Password hashing is implemented in the model

### Objectives
Allow new users to create accounts. This is the entry point for user onboarding.

### Tech Notes
- Endpoint: POST /api/auth/register
- Request body: `{ email, password }`
- Response: `{ user: { id, email }, token }`
- Use existing validation middleware pattern from `src/middleware/validate.ts`

### Acceptance Criteria
- `just test` passes with registration endpoint tests
- `curl -X POST /api/auth/register` with valid data returns 201 and JWT
- Invalid email format returns 400 with validation message
- Existing email returns 409

## Step 3: Implement login endpoint

### Status Quo
- User model and registration endpoint exist
- Test users can be created via registration

### Objectives
Allow existing users to authenticate and receive a token for subsequent API calls.

### Tech Notes
- Endpoint: POST /api/auth/login
- Use constant-time comparison for password verification
- JWT payload should include user ID and expiration

### Acceptance Criteria
- `just test` passes with login tests
- Valid credentials return 200 with decodable JWT containing user ID
- Wrong password returns 401

## Step 4: Add auth middleware

### Status Quo
- Login endpoint issues valid JWTs
- Protected routes are not yet defined

### Objectives
Create reusable middleware that protects routes requiring authentication. This centralizes auth logic and ensures consistent security across the API.

### Tech Notes
- Extract token from `Authorization: Bearer <token>` header
- Attach decoded user to `req.user` for downstream handlers
- See existing middleware pattern in `src/middleware/`

### Acceptance Criteria
- `just test` passes with middleware tests
- Request to protected route without token returns 401
- Request with valid token succeeds and handler receives user object
- Expired token returns 401

## Step 5: Write integration tests

### Status Quo
- All auth components (model, registration, login, middleware) are implemented
- Unit tests exist for individual components

### Objectives
Verify the complete authentication flow works end-to-end. Catch integration issues that unit tests might miss.

### Tech Notes
- Test the full flow: register → login → access protected route
- Include edge cases: expired tokens, malformed tokens, missing headers

### Acceptance Criteria
- `just test` passes with all new integration tests
- Test coverage report shows auth module > 80% coverage
```

## Step Granularity

**Too granular:** "Create file X", "Add import Y"
**Too coarse:** "Implement entire feature"
**Right size:** Logical units that can be completed and verified independently

A good step:
- Can be completed in one focused work session
- Is independently verifiable
- Leaves codebase in working state

## Clarity Guidelines

**Vague:**
```markdown
## Step 3: Add validation
Add validation to the form.
```

**Clear:**
```markdown
## Step 3: Add form validation

### Status Quo
- Registration form exists with email, password, and confirm password fields
- Form submits without any client-side validation

### Objectives
Prevent invalid submissions and provide immediate feedback to users, reducing server load and improving UX.

### Tech Notes
- Email: valid format (use existing `isValidEmail` helper)
- Password: min 8 chars, at least one number and one special character
- Confirm password: must match password field
- Display inline errors below each field
- Disable submit button until all validations pass

### Acceptance Criteria
- `just test` passes with form validation tests
- Submit with invalid email shows "Invalid email format" below field
- Submit with short password shows requirements message
- Submit button disabled until all fields valid
```

## Common Patterns

**Feature implementation:**
1. Models/Schema → 2. Service layer → 3. API endpoints → 4. Validation → 5. Tests

**Refactoring:**
1. Add tests → 2. Extract/restructure → 3. Update consumers → 4. Remove old code → 5. Verify

**Bug fix:**
1. Failing test → 2. Identify cause → 3. Fix → 4. Verify → 5. Regression tests

## Output

Save the plan as a markdown file with:
1. Title: `# Feature: Name` or `# Task: Name`
2. Overview (brief description)
3. Steps (numbered, following format above)

---

## Working with This Plan

### For Implementers

1. **Work sequentially.** Complete steps in order—each step builds on the previous one.

2. **Verify status quo.** Before starting a step, confirm the conditions in "Status Quo" are met. If not, something may have gone wrong in a previous step.

3. **Meet acceptance criteria.** A step is complete only when all acceptance criteria pass. Don't proceed to the next step until they do.

4. **Flag blockers early.** If a step is impossible due to missing information, incorrect assumptions, or technical constraints, stop and raise the issue rather than improvising.

5. **Don't over-engineer.** Implement exactly what's specified. Save improvements and refactors for separate tasks.

6. **Keep the codebase working.** After each step, the code should build and tests should pass. Never leave the codebase in a broken state between steps.

### For Reviewers

1. **Review the plan before implementation.** Check that:
   - Steps are correctly ordered (no step depends on something done later)
   - Status quo for each step follows logically from previous steps
   - Acceptance criteria are verifiable, not just restated requirements
   - No critical steps are missing
   - Step granularity is appropriate (not too coarse, not too granular)

2. **Review implementation against the plan.** For each step, verify:
   - The acceptance criteria are actually met
   - No unplanned changes were introduced
   - The implementation matches the tech notes where specified

3. **Track deviations.** If the implementation necessarily diverges from the plan, ensure the deviation is documented and justified. Update the plan if it will be reused.
