---
name: superteam-writing-plans
description: Create structured implementation plans with machine-parseable task blocks
globs: "**/*.md"
---

# Writing Plans

## Structure

A good plan has:
1. **Goal** — what are we building and why
2. **Constraints** — technical limitations, dependencies, requirements
3. **Tasks** — ordered, independent, testable units of work
4. **Task block** — machine-parseable task list for automation

## Task Block Format

Include a `superteam-tasks` fenced block for automated task tracking:

````markdown
```superteam-tasks
- title: Create data models
  description: Define TypeScript types and validation for User and Post entities
  files: [src/models/user.ts, src/models/post.ts]
- title: Implement API endpoints
  description: REST endpoints for CRUD operations on User and Post
  files: [src/routes/users.ts, src/routes/posts.ts]
- title: Add authentication middleware
  description: JWT validation middleware for protected routes
  files: [src/middleware/auth.ts]
```
````

### Task Properties
- **title**: Short, action-oriented (verb + noun)
- **description**: What to implement and any key decisions
- **files**: Expected files to create or modify

## Task Guidelines

### Good Tasks
- ✅ "Create user model with email validation" — clear, testable, small
- ✅ "Add JWT authentication middleware" — focused, independent
- ✅ "Implement pagination for list endpoints" — specific feature

### Bad Tasks
- ❌ "Set up the project" — too vague
- ❌ "Implement the entire backend" — too large
- ❌ "Fix bugs" — not specific enough

## Task Ordering

1. Data models and types first (foundation)
2. Core business logic (depends on models)
3. API/UI layer (depends on business logic)
4. Integration and cross-cutting concerns (auth, logging, etc.)
5. Polish and optimization (last)

## Size Guidelines

- **Ideal task**: 1-3 files, 50-200 lines of code + tests
- **Too small**: Renaming a variable (just do it)
- **Too large**: "Implement the auth system" (break into subtasks)
- **Splitting rule**: If you can't describe it in one sentence, split it
