---
name: spec-driven-development
description: Guide for implementing Specification-Driven Development in any project using GitHub's spec-kit. Use when users want to start spec-driven development, need to initialize spec-kit in a project, or want guidance on the spec-kit workflow (constitution, specify, clarify, plan, tasks, implement). Covers installation, initialization, and step-by-step prompts for each phase.
---

# Spec-Driven Development

Guide for implementing Specification-Driven Development where specifications become executable artifacts that generate implementation plans and code.

## Prerequisites Check

Before starting, verify spec-kit CLI is installed:

```bash
# Check if installed
specify --help

# If not installed, install globally with uv
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git

# If uv is not installed
curl -LsSf https://astral.sh/uv/install.sh | sh
```

## Project Initialization

Initialize spec-kit in your project:

```bash
# In a new project directory
specify init <project-name> --ai claude

# In existing project (current directory)
specify init . --ai claude
# or
specify init --here --ai claude

# Skip confirmation for non-empty directories
specify init . --force --ai claude
```

This creates:
- `.specify/` directory with templates, scripts, memory
- `CLAUDE.md` with slash commands enabled
- Feature specification structure

Verify slash commands are available: `/speckit.constitution`, `/speckit.specify`, `/speckit.clarify`, `/speckit.plan`, `/speckit.tasks`, `/speckit.implement`

## The Workflow

### Step 1: Constitution (`/speckit.constitution`)

Establish project principles that govern all technical decisions.

**Prompt**:
```
/speckit.constitution Create principles for this [project type] focusing on:
- Code quality and testing standards (test-first, integration-first)
- Architectural constraints (library-first, avoid premature abstraction)
- User experience and performance requirements
- Simplicity guidelines (YAGNI, avoid over-engineering)
- Deployment and operational constraints

Include governance for how these principles should guide implementation choices.
```

**Output**: `.specify/memory/constitution.md`

**Example for Rails app**:
```
/speckit.constitution Create principles for this Rails application focusing on:
- Rails conventions and RESTful design
- Test-first development with RSpec
- Simple, maintainable code over clever abstractions
- PostgreSQL data modeling best practices
- Performance: N+1 query prevention, caching strategy
- Security: strong parameters, authentication/authorization standards
```

---

### Step 2: Specification (`/speckit.specify`)

Transform feature ideas into structured specifications. Be explicit about WHAT and WHY, not HOW (no tech stack yet).

**Prompt template**:
```
/speckit.specify [Feature description with user workflows, data, and behaviors]
```

**Example**:
```
/speckit.specify Build a real-time chat system. Users can create chat rooms, invite members, send messages, and see typing indicators. Messages should have timestamps and support basic formatting (bold, italic, links). Users see presence status (online/offline/away). Include message history with pagination (50 messages per page). Support file attachments up to 10MB. Users can edit their own messages within 15 minutes, delete their own messages anytime, and react to any message with emoji. Room creators can moderate (remove messages, ban users). Include three sample rooms with 5-20 messages each across different states.
```

**What it does**:
- Creates feature branch (e.g., `001-chat-system`)
- Generates `specs/001-chat-system/spec.md` with user stories, acceptance criteria
- Sets up feature directory structure

**Guidelines**:
- Describe user workflows in detail
- Specify data relationships and constraints
- Include edge cases (empty states, error conditions)
- Define success criteria
- Mention sample data needs

---

### Step 3: Clarification (`/speckit.clarify`)

**REQUIRED before planning.** Resolves ambiguities through structured questioning.

**Prompt**:
```
/speckit.clarify
```

The AI will ask sequential questions about unclear areas. Answer them thoroughly. Then refine with follow-up:

**Follow-up prompt example**:
```
For chat rooms, clarify: Can users leave rooms they didn't create? What happens to messages when a user is banned? Can banned users see the room or messages? Should we support direct messages between users, or only room-based chat?
```

**Validate specification**:
```
Review the spec.md acceptance checklist. Check off items that are complete and clear. For unchecked items, explain what's missing and update the spec to address gaps.
```

**Iterate until specification is unambiguous.**

---

### Step 4: Implementation Plan (`/speckit.plan`)

Translate functional requirements into technical architecture. NOW specify tech stack.

**Prompt template**:
```
/speckit.plan [Tech stack, architecture decisions, and technical requirements]
```

**Example**:
```
/speckit.plan Use Ruby on Rails 7.2 with Hotwire Turbo and Action Cable for real-time features. PostgreSQL database with rooms, messages, memberships, and reactions tables. REST API for room/message CRUD. WebSocket via Action Cable for real-time message delivery and typing indicators. Active Storage for file attachments with S3 backend. RSpec for tests with system tests for real-time features. Redis for presence tracking and caching.
```

**What it creates**:
- `specs/[feature]/plan.md` - Implementation strategy
- `specs/[feature]/data-model.md` - Database schemas
- `specs/[feature]/contracts/` - API specs, WebSocket events
- `specs/[feature]/research.md` - Technology investigation
- `specs/[feature]/quickstart.md` - Validation scenarios

**Validation steps**:

1. **Check tech stack**:
```
Review research.md. Verify correct versions and compatibility. For rapidly-changing frameworks (like Hotwire, React, .NET Aspire), research specific implementation patterns we'll use.
```

2. **Audit completeness**:
```
Read plan.md and implementation details. Verify there's a clear task sequence. Ensure core implementation steps reference specific sections in detail documents. Identify anything obvious that's missing.
```

3. **Check against constitution**:
```
Compare plan.md against our constitution. Flag any over-engineered components, unnecessary abstractions, or violations of our principles. Suggest simplifications.
```

4. **Validate checklist**:
```
Review the plan's acceptance checklist. Check off completed items. For unchecked items, update the plan to address them.
```

---

### Step 5: Task Breakdown (`/speckit.tasks`)

Generate executable task list from the implementation plan.

**Prompt**:
```
/speckit.tasks
```

**What it does**:
- Reads plan.md, data-model.md, contracts/, research.md
- Generates ordered task list with dependencies
- Marks parallel tasks with `[P]`
- Specifies exact file paths
- Includes test tasks if TDD specified
- Creates validation checkpoints

**Output**: `specs/[feature]/tasks.md`

**Optional - Advanced validation**:
```
/speckit.analyze
```
Performs cross-artifact consistency check (run before implementation).

---

### Step 6: Implementation (`/speckit.implement`)

Execute tasks systematically.

**Prompt**:
```
/speckit.implement
```

**What it does**:
- Validates prerequisites (constitution, spec, plan, tasks exist)
- Executes tasks in dependency order
- Respects parallel execution markers
- Follows TDD if specified
- Provides progress updates

**Notes**:
- AI will run local CLI commands (ensure tools installed)
- Monitor progress, intervene if stuck
- Test incrementally as features are built
- Check for runtime errors (browser console, logs)

**Post-implementation**:
```
Test the application against acceptance criteria in spec.md. Document any issues found. For bugs, copy error details and logs to me for fixes.
```

---

### Step 7: Iterate

Update specifications based on feedback, then regenerate.

**When requirements change**:
```
Update specs/[feature]/spec.md with [changes]. Then regenerate plan and tasks:
/speckit.plan [updated tech requirements if any]
/speckit.tasks
/speckit.implement
```

**When bugs found**:
```
The spec requires [expected behavior] but the implementation [actual behavior]. Error: [paste error]. Update the spec or plan as needed, then fix implementation.
```

---

## Common Prompts by Project Type

### Rails Application
```
/speckit.plan Ruby on Rails [version] with PostgreSQL. Use Rails conventions, RESTful routing, Hotwire for interactivity. RSpec for testing with FactoryBot. Devise for authentication. Pundit for authorization. Deploy to [Heroku/AWS/etc].
```

### React SPA
```
/speckit.plan React 18 with TypeScript and Vite. TanStack Query for data fetching. React Router for navigation. Tailwind CSS for styling. Vitest and React Testing Library for tests. REST API backend at [URL]. Deploy to Vercel.
```

### Python/Django
```
/speckit.plan Django [version] with Python [version]. PostgreSQL database. Django REST Framework for API. Celery for background jobs. pytest for testing. Docker for deployment.
```

### .NET Application
```
/speckit.plan .NET 8 with ASP.NET Core. Entity Framework Core with SQL Server. Minimal APIs or MVC. xUnit for testing. Blazor Server/WASM for UI if needed. Deploy to Azure.
```

---

## Troubleshooting

**AI over-engineering**:
```
Review this plan against our constitution, specifically simplicity and anti-abstraction principles. Remove components not justified by current requirements. Simplify the approach.
```

**Specification too vague**:
```
/speckit.clarify
```

**AI researching wrong things**:
```
Stop. List specific implementation tasks you're uncertain about. For each, create a targeted research question. Research those specific questions, not general overviews.
```

**Tasks out of order**:
```
/speckit.tasks
Ensure proper dependencies: database models → services → controllers → views → tests. Mark independent tasks with [P] for parallel execution.
```

**Implementation stuck**:
- Provide specific error messages
- Break stuck task into smaller steps
- Verify prerequisites installed
- Check if plan needs refinement

---

## Key Principles

1. **Specs are source of truth** - Code serves specifications
2. **Clarify before planning** - Use `/speckit.clarify` to prevent rework
3. **Constitution guides decisions** - Reference principles when validating plans
4. **Iterate systematically** - Update specs → regenerate plans → re-implement
5. **Validate continuously** - Check acceptance criteria at every phase
6. **Deliver incrementally** - Break large features into multiple specs

---

## Quick Reference

| Step | Command | Input | Output |
|------|---------|-------|--------|
| **1. Principles** | `/speckit.constitution` | Project constraints, standards | `constitution.md` |
| **2. Requirements** | `/speckit.specify` | Feature description (what/why) | `spec.md` |
| **3. Clarify** | `/speckit.clarify` | Answer questions | Updated `spec.md` |
| **4. Planning** | `/speckit.plan` | Tech stack (how) | `plan.md`, `data-model.md`, `contracts/` |
| **5. Tasks** | `/speckit.tasks` | - | `tasks.md` |
| **6. Build** | `/speckit.implement` | - | Working code |
| **7. Iterate** | Update spec → replan → rebuild | Changes | Updated implementation |

---

## Non-Git Workflow

If not using Git branches, set environment variable:

```bash
export SPECIFY_FEATURE=001-feature-name
```

Set in AI agent context before `/speckit.plan` and subsequent commands.