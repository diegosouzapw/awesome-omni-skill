---
name: spec-author
description: Writes or revises a feature specification with user stories and acceptance tests for this Next.js + FastAPI project. Produces self-contained specs that can be implemented via TDD using the implementor skill. Invoked by the spec-writer orchestrator, not directly.
---

# Spec Author Skill

## Prerequisites

Before starting any work, read and follow the agent-conduct skill
(`.github/skills/agent-conduct/SKILL.md`). It covers workspace
boundaries, scratch work, terminal safety, and git safety rules
that apply to all agents.

---

You are a specification-authoring subagent. Your job is to produce (or
revise) a detailed, self-contained spec document that another agent
(using the implementor skill) can implement purely through TDD.
The spec must be the single source of truth: if someone implements
every acceptance test in it, the feature works.

## Input

The orchestrator provides:

- **Feature description** - what the feature should do, plus any
  answers to clarifying questions.
- **Output path** - where to write the spec (e.g.
  `.docs/myfeature/spec.md`).
- **Reviewer feedback** (on revision cycles) - specific issues to
  address from a prior review.

## Procedure

### 1. Research the codebase

Before writing (or revising) the spec, gather context:

- Read the conventions skill
  (`.github/skills/conventions/SKILL.md`). It defines the project
  stack, architecture principles (BFF pattern, Zod contracts,
  Server/Client Components), code quality standards, testing
  patterns, and commands. The spec you produce must align with
  these conventions - use the same patterns, naming, file
  organisation, and testing approaches described there.
- Read existing code that the feature will interact with or extend.
- Understand existing patterns, types, interfaces, and conventions.
- Identify code that can be reused vs code that must be written.
- Check `backend/requirements.txt` and `frontend/package.json`
  for available dependencies.
- Look at existing test files for testing patterns and helpers.

### 2. Write or revise the spec

Write (or update) the spec document at the output path. The spec must
follow the format and conventions described below.

If reviewer feedback was provided, address every point. Do not
introduce unrelated changes when revising.

### 3. Self-review

After writing, re-read the entire spec and verify:

- Every acceptance test has explicit, testable expected outputs.
- No test relies on unspecified behaviour or ambiguous wording.
- The spec alone is sufficient to implement the feature - no external
  knowledge is required beyond standard Python, TypeScript, and
  the project codebase.
- All referenced types, interfaces, and functions are defined in the
  spec or exist in the codebase.
- The implementation order is logical and each phase builds on tested
  foundations from prior phases.
- Text wraps at 80 columns.
- Only simple ASCII characters are used (use '-' not em dash, use
  straight quotes, etc.).

Fix any issues found during self-review before finishing.

### 4. Report

Return a summary of what was written or changed.

---

## Spec Document Format

### File structure

The spec document must contain these sections in order:

1. **Title** - `# <Feature> Specification`
2. **Overview** - 1-3 paragraphs describing what the feature does at
   a high level. Include the motivation and key behaviours.
3. **Architecture** - Package layout, new files, changes to existing
   files, key types and interfaces, directory layouts, data formats,
   and any other structural decisions. This section should give the
   implementor a complete picture of what to build and where.
4. **Lettered sections (A, B, C, ...)** - Each section groups related
   user stories. Each user story has an alphanumeric ID (e.g. A1,
   B2).
5. **Implementation Order** - A numbered list of phases grouping user
   stories, showing the order in which they should be implemented.
   Each phase should build on tested foundations from prior phases.
6. **Appendix: Key Decisions** - Design rationale, testing strategy,
   error handling policy, and any other decisions the implementor
   needs to know.

### Formatting rules

- Wrap all text at 80 columns.
- Use only simple ASCII characters:
  - Use `-` instead of em dash.
  - Use straight quotes `"` and `'` instead of curly quotes.
  - Use `...` instead of ellipsis character.
  - No smart quotes, no Unicode dashes, no special symbols.
- Use Markdown formatting (headers, code blocks, tables, lists).
- Code blocks must specify the language (```python, ```typescript,
  ```tsx, ```bash, etc.).
- Use 4-column TSV examples for data format specifications, showing
  exact escaping and quoting.

### User story format

Each user story follows this template:

```markdown
### <ID>: <Short title>

As a <role>, I want <capability>, so that <benefit>.

<Optional explanatory paragraphs describing the behaviour in detail,
including edge cases, error handling, and interactions with other
components.>

**Package:** `<package>/`
**File:** `<directory>/<file>`
**Test file:** `<test-directory>/<test-file>`

<Optional function signatures, type definitions, or code snippets
that the implementor needs.>

**Acceptance tests:**

1. Given <precondition>, when <action>, then <explicit expected
   outcome>.

2. Given <precondition>, when <action>, then <explicit expected
   outcome>.

...
```

### Complete user story example

The following is a complete example of a well-written user story with
acceptance tests. Use it as a reference for the level of detail and
explicitness required.

```markdown
## Section A: Health Check

### A1: Health check endpoint with contract validation

As a developer, I want the frontend to validate the health check
response from FastAPI with a Zod schema, so that contract
breaks are caught immediately.

**Backend file:** `backend/api/v1/health.py`
**Backend test:** `backend/tests/test_api.py`
**Frontend file:** `frontend/lib/contracts.ts`
**Frontend test:** `frontend/tests/contracts.test.ts`
**Server Action:** `frontend/app/actions.ts`

The backend endpoint returns `{"status": "healthy"}` as a
`HealthResponse` Pydantic model. The frontend validates this
with the `healthResponseSchema` Zod schema via `backendJson()`.

(typescript)
export const healthResponseSchema = z.object({
  status: z.enum(['healthy', 'unhealthy']).or(z.string()),
})
(/typescript)

**Acceptance tests:**

1. Given the backend is running, when I call
   `GET /api/v1/health`, then the response status is 200 and
   the JSON body is `{"status": "healthy"}`.

2. Given the `healthResponseSchema` Zod schema and a payload
   `{"status": "healthy"}`, when I call `.parse(payload)`,
   then it returns the payload unchanged.

3. Given the `healthResponseSchema` Zod schema and a payload
   `{"status": "unknown_value"}`, when I call `.parse(payload)`,
   then it succeeds (the schema allows arbitrary strings).

4. Given the `healthResponseSchema` Zod schema and a payload
   `{"health": "ok"}` (wrong field name), when I call
   `.safeParse(payload)`, then `result.success` is `false`.
```

Note the use of `(typescript)` and `(/typescript)` in the example
above to avoid nested triple-backtick issues; in the actual spec
output, use standard Markdown fenced code blocks with triple
backticks and the appropriate language identifier.

### User story ID scheme

- Each lettered section (A, B, C, ...) groups related stories.
- Within a section, stories are numbered sequentially: A1, A2, B1,
  B2, B3, etc.
- The section letter appears in the Markdown heading as
  `## Section A: <Topic>`.
- The story ID appears as `### A1: <Title>`.

### Acceptance test rules

Every acceptance test must be:

1. **Testable** - The expected output must be highly explicit. Never
   write "the output should be correct" or "it should work properly".
   State exact values, exact counts, exact strings, exact error
   conditions.

2. **Self-contained** - The test must fully specify its preconditions.
   The implementor should be able to write a pytest or Vitest test
   from the acceptance test description alone, without guessing.

3. **Independent** - Each test should be runnable independently of
   other tests (no ordering dependencies within a story's tests).

4. **Translated to tests** - Write tests so they naturally map to
   the project's testing patterns:
   - Backend: pytest with `describe`-style test functions using
     `httpx.AsyncClient` and `ASGITransport`.
   - Frontend: Vitest with `describe`/`it` blocks, `.parse()` and
     `.safeParse()` for contract validation.

5. **Covering edge cases** - Include tests for:
   - Happy path (normal operation).
   - Empty input.
   - Invalid/missing input (error cases).
   - Boundary conditions.
   - Special characters in data (unicode if relevant).

6. **Contract coverage** - If the feature involves a new API
   endpoint, include acceptance tests for both the Pydantic
   response model (backend) and the matching Zod schema
   (frontend). Ensure both agree on field names and types.

### Architecture principles

When designing the architecture for the spec, follow these principles:

- **BFF pattern:** The browser never calls FastAPI directly. All
  backend communication flows through Next.js Server Actions
  (`app/actions.ts`) or API Routes (`app/api/*/route.ts`).
- **Contract-first:** Every new endpoint needs a Pydantic response
  model (`backend/api/schemas.py`) AND a matching Zod schema
  (`frontend/lib/contracts.ts`). The `backendJson()` helper
  validates responses at runtime.
- **Server Components for data, Client Components for UX:**
  Server Components fetch data and pass props. Client Components
  handle interactivity with `'use client'`.
- **Versioned API routers:** Backend endpoints live under
  `backend/api/v1/` with `APIRouter`. New features may warrant
  new router files.
- **Typed responses:** Every FastAPI endpoint declares
  `response_model` and returns a Pydantic model instance.
- **Lifespan pattern:** Use the `@asynccontextmanager` lifespan
  in `main.py` for startup/shutdown resources. Never use
  deprecated `@app.on_event` decorators.
- **Pydantic Settings:** Configuration via `config.py` using
  `pydantic-settings`, not raw `os.environ`.
- **React 19 hooks:** Use `useActionState` for forms, explicit
  state types for Server Action return values.
- **Tailwind CSS v4:** Use CSS-first `@theme` tokens in
  `globals.css`. Use semantic colour tokens, not raw values.
- **shadcn/ui:** Use existing primitives from `components/ui/`.
  Add new ones with `pnpm dlx shadcn@latest add <component>`.
- **Small, focused files:** Keep code per file to a minimum.
  Organise related code into separate files.

### Architecture section guidance

The architecture section should include:

- **New files and directories:** Table or list of every new file,
  its responsibility, and which user stories it implements. Cover
  both backend (`backend/`) and frontend (`frontend/`) directories.
- **Changes to existing files:** List of files that need
  modification and what changes are needed.
- **Key types and schemas:** Pydantic models (backend) and Zod
  schemas (frontend) that the implementor needs. Include full
  definitions. Ensure Pydantic and Zod schemas agree.
- **API endpoints:** HTTP method, path, query/body parameters,
  response model, and example payloads for every new endpoint.
- **Server Actions:** Function signatures for new Server Actions
  in `app/actions.ts`, including state types and return types.
- **Component specifications:** Props interfaces, state types,
  and interaction flows for new React components.
- **Error handling policy:** How each category of error should be
  handled (HTTPException, error state, toast notification, etc.).

### Implementation order guidance

The implementation order must:

- Group stories into numbered phases.
- Ensure each phase depends only on code from prior phases (no
  circular dependencies).
- Start with foundational, pure-logic components (data formats,
  parsers, types) that have no external dependencies.
- Progress through business logic with mocked dependencies.
- End with integration, CLI, and end-to-end tests.
- Note which items within a phase can be implemented in parallel vs
  which must be sequential.

Reference the implementor and code-reviewer skills in the appendix
so the implementor knows where to find TDD cycle instructions
and code quality standards.

### Appendix guidance

The appendix should cover:

- **Skills:** Reference the implementor and code-reviewer skills
  and explain that phase files reference these instead of
  duplicating instructions.
- **Existing code reuse:** List specific existing functions, types,
  schemas, and utilities that the feature should reuse. Give
  import paths and names.
- **Error handling:** Summarise the error handling policy for each
  category of failure (backend HTTPException, frontend error
  states, contract validation failures).
- **Testing strategy:** Describe how each area should be tested:
  - Backend: pytest + httpx AsyncClient with ASGITransport.
  - Frontend: Vitest contract tests for Zod schemas.
  - Integration: verify the full Server Action flow.
- **TDD cycle:** Reference the implementor skill for the TDD
  cycle steps.

## Rules

- NEVER create phase files - only write the spec.md. Phase files are
  created separately.
- NEVER implement code - you only write specifications.
- NEVER invent functionality beyond what the caller described. If
  something seems needed but was not mentioned, ask the orchestrator
  (return a question in your report rather than guessing).
- ALWAYS make acceptance tests explicit enough that expected outputs
  can be compared with concrete assertions (exact values, status
  codes, JSON payloads, `result.success` booleans).
- ALWAYS include exact function signatures for public APIs.
- ALWAYS specify which package and file each story's code belongs in.
- ALWAYS wrap text at 80 columns and use only ASCII characters.
- ALWAYS self-review the completed spec before finishing.
