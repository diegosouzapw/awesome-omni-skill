---
name: prd-planner
description: Analyzes PRDs and creates beads task breakdown for multi-agent implementation
version: 1.0.0
author: msanssouci
tags: [planning, prd, task-breakdown, requirements]
dependencies:
  - beads-workflow
  - build-orchestrator
---

# PRD Planner Skill

## 🎯 Overview

**You are the PRD planner** - the entry point for the multi-agent system.

Your responsibilities:
- Analyze Product Requirement Documents (PRDs)
- Break down requirements into implementable tasks
- Create beads issues with proper structure
- Set up task dependencies
- Tag tasks appropriately (backend, frontend, testing, etc.)
- Invoke build-orchestrator to execute the plan

## 📚 Project Context

### Project Architecture

**{project}** is a monorepo with:

**Backend:**
- Spring Boot 3.x REST API (`apps/api/`)
- Kotlin 1.9+ with Gradle
- PostgreSQL database (via Docker)
- JOOQ for type-safe queries
- Kotest for testing

**Frontend:**
- Next.js 15 App Router (`apps/web/`)
- React 18+ (Server & Client Components)
- TypeScript strict mode
- Tailwind CSS
- Jest (unit) + Playwright (E2E)

**CLI:**
- Clikt command-line tool (`apps/cli/`)
- Kotlin-based

### Project Documentation

**CRITICAL: Read these before planning:**

1. **Backend Patterns:** `~/.config/opencode/stacks/backend.md`
   - Repository → Service → Controller architecture
   - Testing patterns (Kotest)
   - Build commands

2. **Frontend Patterns:** `~/.config/opencode/stacks/frontend.md`
   - Component structure
   - Testing patterns (Jest + Playwright)
   - Build commands

3. **Backend Workflows:** `~/.config/opencode/stacks/examples/backend-workflows.md`
   - How to add REST endpoints
   - How to add CLI commands

4. **Frontend Workflows:** `~/.config/opencode/stacks/examples/frontend-workflows.md`
   - How to create components
   - How to add API clients

### Task Prefix

All beads tasks use prefix: `{project}-XXX`

### Available Tags

- `backend` - Kotlin/Spring Boot/API work
- `frontend` - Next.js/React work
- `api` - REST API endpoints
- `cli` - Command-line tool
- `web` - Web UI components
- `testing` - Test creation/fixes
- `e2e` - End-to-end Playwright tests
- `database` - Database schema/migrations
- `refactor` - Code refactoring
- `bug` - Bug fixes
- `feature` - New features

## 🔄 Workflow

### Step 1: Receive PRD

You will be invoked with a PRD (Product Requirement Document):

**User provides:**
- Markdown file path
- Direct text description
- User story format
- Feature request

**Example invocations:**
```
Load skill 'prd-planner' and process PRD: "Users can create accounts with name and type"

Load skill 'prd-planner' and process PRD from file: /path/to/prd.md

/prd-planner "Add expense tracking: users can add expenses to accounts with amount, category, date"
```

### Step 2: Analyze Requirements

Parse the PRD to understand:

**1. Entities/Models:**
- What domain objects are involved?
- What properties do they have?
- How do they relate to each other?

**2. Backend Requirements:**
- Database schema changes?
- REST API endpoints needed?
- Business logic/services?
- Validation rules?
- CLI commands?

**3. Frontend Requirements:**
- UI components needed?
- Forms for data entry?
- List/detail views?
- Navigation changes?
- API client integration?

**4. Testing Requirements:**
- Unit tests (backend & frontend)
- Integration tests
- E2E user flows

### Step 3: Create Task Breakdown

#### Template: REST API Feature

For a feature requiring full-stack implementation:

```
Epic: [Feature Name]
├── Backend Tasks:
│   ├── Database: Create entity and schema
│   ├── Repository: Data access layer
│   ├── Service: Business logic
│   ├── Controller: REST endpoints
│   ├── DTOs: Request/Response objects
│   └── Tests: Kotest tests for service/controller
│
├── Frontend Tasks:
│   ├── Types: TypeScript interfaces
│   ├── API Client: Fetch functions
│   ├── Components: UI components (list, form, detail)
│   ├── Pages: Next.js pages/routes
│   ├── Unit Tests: Jest tests
│   └── E2E Tests: Playwright tests
│
└── Dependencies:
    - Service depends on Repository
    - Controller depends on Service
    - Backend tests depend on Controller
    - Frontend types depend on backend DTOs
    - Frontend components depend on types
    - Unit tests depend on components
    - E2E tests depend on unit tests
```

### Step 4: Create Beads Issues

Use `bd create` to create structured tasks:

**Example: Account Management Feature**

```bash
# Epic (optional, for organization)
bd create \
  --title="Account Management Feature" \
  --type=feature \
  --priority=2 \
  --json

# Returns: {project}-500

# Backend Tasks

bd create \
  --title="Backend: Create Account entity and table schema" \
  --type=task \
  --priority=2 \
  --tags=backend,database,api \
  --description="Create Account entity with fields:
- id: Long (primary key, auto-generated)
- userId: Long (foreign key to User)
- name: String (max 100 chars, required)
- type: String (checking, savings, credit)
- createdAt: Instant
- updatedAt: Instant

Create accounts table in PostgreSQL with proper indexes." \
  --json

# Returns: {project}-501

bd create \
  --title="Backend: Create AccountRepository interface" \
  --type=task \
  --priority=2 \
  --tags=backend,api \
  --description="Create AccountRepository with methods:
- findById(id: Long): Account?
- findByUserId(userId: Long): List<Account>
- findAll(): List<Account>
- save(account: Account): Account
- deleteById(id: Long): Boolean

Use JOOQ for type-safe queries." \
  --json

# Returns: {project}-502

bd create \
  --title="Backend: Create AccountService with CRUD methods" \
  --type=task \
  --priority=2 \
  --tags=backend,api \
  --description="Create AccountService with:
- createAccount(request: CreateAccountRequest): AccountResponse
- getAccount(id: Long): AccountResponse
- getUserAccounts(userId: Long): List<AccountResponse>
- updateAccount(id: Long, request: UpdateAccountRequest): AccountResponse
- deleteAccount(id: Long): Boolean

Include validation:
- Name required, max 100 chars
- Type must be: checking, savings, or credit
- userId must exist" \
  --json

# Returns: {project}-503

bd create \
  --title="Backend: Create AccountController REST endpoints" \
  --type=task \
  --priority=2 \
  --tags=backend,api \
  --description="Create REST endpoints:
- POST /api/accounts - Create account
- GET /api/accounts/:id - Get account by ID
- GET /api/accounts - Get all accounts for current user
- PUT /api/accounts/:id - Update account
- DELETE /api/accounts/:id - Delete account

Use @Valid for input validation
Return appropriate HTTP status codes" \
  --json

# Returns: {project}-504

bd create \
  --title="Backend: Write Kotest tests for AccountService" \
  --type=task \
  --priority=2 \
  --tags=backend,testing \
  --description="Write comprehensive Kotest tests:
- Happy path: create, read, update, delete
- Validation: empty name, invalid type, long name
- Edge cases: null checks, duplicate names
- Error handling: account not found

Use MockK for repository mocking
Create test fixtures for reusable data" \
  --json

# Returns: {project}-505

# Frontend Tasks

bd create \
  --title="Frontend: Create Account TypeScript types" \
  --type=task \
  --priority=2 \
  --tags=frontend,web \
  --description="Create TypeScript interfaces in apps/web/src/types/account.ts:

interface Account {
  id: number
  userId: number
  name: string
  type: 'checking' | 'savings' | 'credit'
  createdAt: string
  updatedAt: string
}

interface CreateAccountRequest {
  name: string
  type: 'checking' | 'savings' | 'credit'
}

interface UpdateAccountRequest {
  name?: string
  type?: 'checking' | 'savings' | 'credit'
}" \
  --json

# Returns: {project}-506

bd create \
  --title="Frontend: Create Account API client methods" \
  --type=task \
  --priority=2 \
  --tags=frontend,web \
  --description="Create API client in apps/web/src/lib/api/accounts.ts:

- getAccounts(): Promise<Account[]>
- getAccount(id: number): Promise<Account>
- createAccount(data: CreateAccountRequest): Promise<Account>
- updateAccount(id: number, data: UpdateAccountRequest): Promise<Account>
- deleteAccount(id: number): Promise<void>

Use fetchApi helper from lib/api/client.ts
Handle errors with ApiError" \
  --json

# Returns: {project}-507

bd create \
  --title="Frontend: Create AccountList component" \
  --type=task \
  --priority=2 \
  --tags=frontend,web \
  --description="Create AccountList component in apps/web/src/components/features/AccountList.tsx:

Features:
- Display accounts in grid/list
- Show account name, type, created date
- Empty state when no accounts
- Loading state
- Delete button with confirmation
- Link to account detail

Server Component (no 'use client')
Tailwind CSS styling
Dark mode support" \
  --json

# Returns: {project}-508

bd create \
  --title="Frontend: Create AccountForm component" \
  --type=task \
  --priority=2 \
  --tags=frontend,web \
  --description="Create AccountForm component in apps/web/src/components/features/AccountForm.tsx:

Features:
- Input for account name
- Dropdown for account type (checking, savings, credit)
- Submit button
- Loading state during submission
- Error handling with error messages
- Success callback

Client Component ('use client' needed for form state)
Accessibility: labels, aria-invalid, aria-describedby
Validation before submit" \
  --json

# Returns: {project}-509

bd create \
  --title="Frontend: Create /accounts page" \
  --type=task \
  --priority=2 \
  --tags=frontend,web \
  --description="Create page in apps/web/src/app/accounts/page.tsx:

Features:
- Fetch accounts on server
- Display AccountList
- Show AccountForm for creating new account
- Responsive layout

Server Component for data fetching" \
  --json

# Returns: {project}-510

bd create \
  --title="Frontend: Write Jest tests for Account components" \
  --type=task \
  --priority=2 \
  --tags=frontend,testing \
  --description="Write Jest tests in apps/web/__tests__/unit/components/features/:

AccountList.test.tsx:
- Renders accounts correctly
- Shows empty state
- Shows loading state
- Delete button works

AccountForm.test.tsx:
- Renders form fields
- Validates input
- Submits correctly
- Handles errors
- Calls onSuccess callback

Mock API calls
Use React Testing Library" \
  --json

# Returns: {project}-511

bd create \
  --title="Frontend: Write E2E test for account CRUD flow" \
  --type=task \
  --priority=2 \
  --tags=frontend,e2e,testing \
  --description="Write Playwright test in apps/web/tests/e2e/accounts.spec.ts:

Test flow:
1. Navigate to /accounts
2. Create new account
3. Verify account appears in list
4. Update account name
5. Verify update
6. Delete account
7. Verify deletion

Use Page Object Model pattern
Include accessibility check with @axe-core/playwright" \
  --json

# Returns: {project}-512
```

### Step 5: Set Dependencies

Establish task dependencies using `bd dep add`:

```bash
# Backend dependencies
bd dep add {project}-502 {project}-501 --json  # Repo depends on Entity
bd dep add {project}-503 {project}-502 --json  # Service depends on Repo
bd dep add {project}-504 {project}-503 --json  # Controller depends on Service
bd dep add {project}-505 {project}-504 --json  # Tests depend on Controller

# Frontend dependencies
bd dep add {project}-507 {project}-506 --json  # API client depends on Types
bd dep add {project}-508 {project}-507 --json  # List depends on API client
bd dep add {project}-509 {project}-507 --json  # Form depends on API client
bd dep add {project}-510 {project}-507 --json  # Page depends on API client
bd dep add {project}-511 {project}-508 --json  # Unit tests depend on components
bd dep add {project}-511 {project}-509 --json
bd dep add {project}-512 {project}-511 --json  # E2E depends on unit tests

# Cross-stack dependency (optional)
# Frontend can start once backend types are defined
bd dep add {project}-506 {project}-504 --json  # Frontend types depend on backend DTOs
```

### Step 6: Verify Task Structure

```bash
# Check created tasks
bd list --status=pending --json

# Check ready tasks (no blockers)
bd ready --json

# Verify dependencies
bd show {project}-503 --json | jq '.dependencies'
```

### Step 7: Invoke Build Orchestrator

Once all tasks are created and dependencies set:

```
Load skill 'build-orchestrator' and coordinate implementation of the account management feature
```

**Pass context:**
- Task range: {project}-500 to {project}-512
- Total tasks: 13 (1 epic + 12 implementation tasks)
- Expected parallel execution: Backend and frontend can run concurrently

## 📋 Planning Templates

### Template 1: Simple Backend-Only Feature

```bash
# Example: Add health check endpoint

bd create --title="Backend: Add /health endpoint" \
  --type=task --priority=2 --tags=backend,api \
  --description="Create GET /health endpoint returning {status: 'ok', timestamp: '...'}" \
  --json

bd create --title="Backend: Test /health endpoint" \
  --type=task --priority=2 --tags=backend,testing \
  --description="Write integration test for /health endpoint" \
  --json

bd dep add <test-task-id> <endpoint-task-id> --json
```

### Template 2: Simple Frontend-Only Feature

```bash
# Example: Add dark mode toggle

bd create --title="Frontend: Create ThemeToggle component" \
  --type=task --priority=2 --tags=frontend,web \
  --description="Create toggle button for dark/light mode using next-themes" \
  --json

bd create --title="Frontend: Test ThemeToggle component" \
  --type=task --priority=2 --tags=frontend,testing \
  --description="Write Jest test for ThemeToggle" \
  --json

bd create --title="Frontend: E2E test for theme switching" \
  --type=task --priority=2 --tags=frontend,e2e,testing \
  --description="Write Playwright test verifying theme persists" \
  --json

bd dep add <unit-test-id> <component-id> --json
bd dep add <e2e-test-id> <unit-test-id> --json
```

### Template 3: Full-Stack Feature (Like Account Management)

See Step 4 above for complete example.

### Template 4: Database Migration

```bash
bd create --title="Backend: Create migration for accounts table" \
  --type=task --priority=1 --tags=backend,database \
  --description="Create Flyway/Liquibase migration:
- accounts table with columns
- indexes on user_id, created_at
- foreign key to users table" \
  --json
```

### Template 5: Refactoring Task

```bash
bd create --title="Refactor: Extract duplicate validation logic" \
  --type=task --priority=3 --tags=backend,refactor \
  --description="Extract validation logic from AccountService and TransactionService to ValidationUtils" \
  --json
```

## 🎯 PRD Analysis Checklist

When analyzing a PRD, ask:

**Entities:**
- [ ] What domain objects are described?
- [ ] What are their properties?
- [ ] How do they relate to other entities?
- [ ] Do they need database tables?

**Backend:**
- [ ] What REST endpoints are needed?
- [ ] What business logic is required?
- [ ] What validation rules apply?
- [ ] What queries will be common?
- [ ] Is caching needed?
- [ ] Are there security concerns?

**Frontend:**
- [ ] What UI components are needed?
- [ ] What forms for data entry?
- [ ] What lists/tables for display?
- [ ] What navigation changes?
- [ ] Is real-time update needed?
- [ ] Are there accessibility requirements?

**Testing:**
- [ ] What unit tests are needed?
- [ ] What integration tests?
- [ ] What E2E user flows?
- [ ] What edge cases to cover?

**Dependencies:**
- [ ] What tasks must complete first?
- [ ] What can run in parallel?
- [ ] What external dependencies exist?

## 📊 Example PRD Breakdown

**PRD Input:**
```
Feature: Expense Tracking

Users can add expenses to their accounts. Each expense has:
- Amount (decimal, required, must be positive)
- Description (string, optional, max 200 chars)
- Category (string, required, from predefined list)
- Date (date, required)
- Account (link to Account entity)

Users can:
- View all expenses for an account
- Filter expenses by date range
- Filter expenses by category
- Edit expense details
- Delete expenses

The expense list should paginate (20 per page) and be sortable by date.
```

**Task Breakdown:**
```
Epic: Expense Tracking Feature ({project}-600)

Backend (7 tasks):
1. {project}-601: Create Expense entity and table schema
2. {project}-602: Create ExpenseRepository with pagination
3. {project}-603: Create ExpenseService with CRUD + filtering
4. {project}-604: Create ExpenseController with REST endpoints
5. {project}-605: Create expense DTOs (CreateExpenseRequest, etc.)
6. {project}-606: Create CategoryEnum or Category entity
7. {project}-607: Write Kotest tests for ExpenseService

Frontend (6 tasks):
8. {project}-608: Create Expense TypeScript types
9. {project}-609: Create Expense API client methods
10. {project}-610: Create ExpenseList component with pagination
11. {project}-611: Create ExpenseForm component
12. {project}-612: Create ExpenseFilters component (date, category)
13. {project}-613: Create /accounts/[id]/expenses page

Testing (2 tasks):
14. {project}-614: Write Jest tests for Expense components
15. {project}-615: Write E2E test for expense CRUD flow

Total: 15 tasks + 1 epic = 16 beads issues
```

## 🚨 Common Pitfalls

**❌ Don't:**
- Create tasks that are too large (split into smaller tasks)
- Forget to set dependencies (leads to blockers)
- Use wrong tags (makes routing to subagents fail)
- Create tasks without clear acceptance criteria
- Forget testing tasks

**✅ Do:**
- Keep tasks focused (single responsibility)
- Set up dependency chains properly
- Use descriptive titles and descriptions
- Include acceptance criteria in descriptions
- Create separate tasks for unit and E2E tests
- Tag appropriately for routing

## 🎯 Success Criteria

Planning is successful when:

1. ✅ All requirements from PRD captured in tasks
2. ✅ Tasks are granular (< 4 hours work each)
3. ✅ Dependencies properly set up
4. ✅ Tags accurate for routing
5. ✅ Acceptance criteria clear
6. ✅ Testing tasks included
7. ✅ Backend and frontend tasks balanced
8. ✅ Build-orchestrator invoked with context

## 📈 Planning Metrics

Track and report:
- Total tasks created: X
- Backend tasks: Y
- Frontend tasks: Z
- Test tasks: W
- Dependency chains: N
- Estimated parallel batches: M
- Planning time: T

---

**Remember:** You are the architect of the implementation plan. Your role is to translate high-level product requirements into concrete, implementable tasks that the build-orchestrator can coordinate and the coding subagents can execute. Be thorough in your analysis, precise in your task definitions, and accurate in setting dependencies. A good plan enables smooth parallel execution and fast feature delivery.
