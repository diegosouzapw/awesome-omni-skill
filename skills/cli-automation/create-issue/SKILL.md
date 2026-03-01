---
name: create-issue
description: Create a well-structured task issue on GitHub that CI will validate and auto-sync to the project board. Use when user asks to create a task, file an issue, plan work, break down a feature, or make a new issue for any Phoenix repo.
---

# Create task issue — Phoenix Agentic Engine Interface

## Overview

Create task issues using the structured `task.yml` template so that the
`issue-validate.yml` CI workflow can parse fields, validate them, add the issue
to the project board, and auto-populate board fields (Status, Priority, Size,
Area, Work mode, Lock Key, Needed Files, Depends On).

## Mandatory first step

Check the project board to avoid duplicate work and understand current context:

- Project board: https://github.com/users/rivie13/projects/3
- Check for existing issues covering the same scope before creating a new one:
  ```
  mcp_github_github_search_issues(owner="rivie13", repo="Phoenix-Agentic-Engine-Interface", query="<keywords>")
  ```

## Required fields

Every task issue MUST include all of these in the issue body using the exact
heading format below (this is what CI parses):

| Field | Heading in body | Required | Example |
|-------|----------------|----------|---------|
| Target repo | `### Target repository` | Yes | `Phoenix-Agentic-Engine-Interface` |
| Parent branch | `### Parent branch (PR target)` | Yes | `feature/phase-3-sdk-client` |
| Working branch | `### Working branch` | Yes | `subfeature/task/sdk-client-auth` |
| Work mode | `### Work mode` | Yes | `Local IDE` / `CLI Agent` / `Cloud Agent` |
| Separate worktree | `### Separate worktree?` | Yes | `No` / `Yes` / `N/A (Cloud Agent)` |
| Priority | `### Priority` | Yes | `P1 — High` |
| Size | `### Size` | Yes | `M — Medium (half day to full day)` |
| Area | `### Area` | Yes | `sdk/client` |
| Description | `### Task description` | Yes | Free-form, detailed enough for agent |
| Acceptance criteria | `### Acceptance criteria` | Yes | Checkboxes |
| Key files | `### Key files` | No | File paths |
| Depends on | `### Depends on` | No | `#42, #57` |
| Lock key | `### Lock key` | No | `sdk/client/auth/*` |

## Valid values

### Priority
- `P0 — Critical / blocker`
- `P1 — High`
- `P2 — Medium`
- `P3 — Low`

### Size
- `XS — Extra small (< 1 hour)`
- `S — Small (1-4 hours)`
- `M — Medium (half day to full day)`
- `L — Large (multi-day)`
- `XL — Extra large (multi-week)`

### Work mode
- `Local IDE` — Human in VS Code
- `CLI Agent` — Codex / Copilot CLI / other in a worktree
- `Cloud Agent` — GitHub Copilot cloud coding agent

### Area (global — applies across all repos)
`module/assistant-ui`, `module/mcp`, `module/agent`, `module/addons`,
`core`, `gateway`, `worker`, `orchestrator`,
`sdk/client`, `sdk/core`, `contracts`, `infra`,
`app/pages`, `components`, `content`, `public`,
`api`, `domain`, `tests`, `docs`, `ci`

### Branch naming convention
- Feature branches: `feature/<description>`
- Sub-feature / task: `subfeature/<type>/<description>`
  - Types: `task`, `fix`, `chore`, `docs`
- Direct fixes: `fix/<description>`, `chore/<description>`, `docs/<description>`

## Creating an issue (step by step)

### 1. Determine execution context

- **Which repo** will the work happen in?
- **What parent branch** does the PR merge into? (Usually a `feature/*` branch, or `main`)
- **What working branch** will be created?
- **What work mode** — Local IDE, CLI Agent, or Cloud Agent?

### 2. Determine board fields

- **Priority**: How urgent? P0 = blocker, P3 = nice-to-have
- **Size**: How big? XS = trivial, L = multi-day
- **Area**: What part of the codebase?

### 3. Write the issue body

Format the body with exactly these markdown headings (CI parses them):

```markdown
### Target repository

Phoenix-Agentic-Engine-Interface

### Parent branch (PR target)

feature/phase-3-sdk-client

### Working branch

subfeature/task/sdk-client-auth

### Work mode

Local IDE

### Separate worktree?

No

### Priority

P2 — Medium

### Size

S — Small (1-4 hours)

### Area

sdk/client

### Task description

Add authentication support to the SDK client...

### Acceptance criteria

- [ ] Auth tokens are passed in headers
- [ ] All tests pass
- [ ] No lint errors

### Key files

- sdk/client/auth.ts
- tests/client/auth.test.ts

### Depends on

_No response_

### Lock key

_No response_
```

### 4. Create via MCP tool

```
mcp_github_github_issue_write(
  method="create",
  owner="rivie13",
  repo="Phoenix-Agentic-Engine-Interface",
  title="[Task]: <concise title>",
  body="<body from step 3>",
  labels=["task"]
)
```

### 5. What happens automatically

CI will:
1. Parse all fields from the issue body headings.
2. Validate required fields are present and well-formed.
3. Add the issue to the project board (if not already there).
4. Set board fields: Status → Ready, Priority, Size, Area, Work mode.
5. Set text fields: Depends On, Lock Key, Needed Files (if provided).
6. Add a `base-branch:<branch>` label for cloud-agent resolution.
7. Comment with a success summary or validation errors.

If validation fails, CI adds a `needs-triage` label and comments with
specific errors. Edit the issue to fix them — CI re-validates on edit.

## Breaking down features into tasks

When a feature is too large for a single issue:

1. Create the parent feature branch: `feature/<name>`
2. Break into sub-tasks, each as a separate issue
3. Each sub-task's **parent branch** = `feature/<name>`
4. Each sub-task's **working branch** = `subfeature/task/<name>`
5. Set `Depends on` to chain sequential tasks

## Related repos

| Repo | GitHub |
|------|--------|
| Engine | `rivie13/Phoenix-Agentic-Engine` |
| Backend | `rivie13/Phoenix-Agentic-Engine-Backend` |
| Website Frontend | `rivie13/Phoenix-Agentic-Website-Frontend` |
| Website Backend | `rivie13/Phoenix-Agentic-Website-Backend` |
| Supervisor | `rivie13/Phoenix-Agentic-Workspace-Supervisor` |
| Command Center | `rivie13/Phoenix-Agentic-VSCode-CommandCenter` |

Use the same `task.yml` template when creating issues in any repo.
The `create-issue` skill exists in all repos with identical guidance.
```
