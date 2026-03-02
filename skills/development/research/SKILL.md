---
name: research
description: Research the codebase to find and explain specific topics, answering questions about architecture, configuration, data flows, and implementation details
license: MIT
compatibility:
  - runtime:any
allowed-tools:
  - Read
  - Glob
  - Grep
metadata:
  author: thoreinstein
  version: 1.0.0
---

# Research

Research the codebase to find and explain specific topics. This is a **read-only skill** - it discovers and documents information but does not modify code.

## When to Use

- Understanding where something is configured (e.g., "Where is Stripe configured?")
- Learning how a feature works (e.g., "How does authentication work?")
- Finding implementation details (e.g., "Where are RSS feeds fetched and parsed?")
- Locating infrastructure definitions (e.g., "Where are the Kubernetes manifests for the API?")
- Mapping dependencies and data flows

## Input

A research question about the codebase. Examples:
- "Where is Stripe configured?"
- "How does authentication work?"
- "Where are RSS feeds fetched and parsed?"
- "Where do we define Kubernetes manifests for the API service?"

If no question is provided, ask for clarification before proceeding.

## Workflow

### 1. Interpret the Research Topic

Rewrite the question as a clear, concrete research goal:
- "Where is Stripe configured?" → "Find where and how Stripe API keys are configured and used"
- "How does authentication work?" → "Identify the main entry points for authentication and session handling"

If the topic is ambiguous, make a reasonable assumption and state it in the report.

### 2. Explore the Codebase

Use a top-down search strategy:

1. **High-level scan**: Identify relevant directories and key modules
2. **Pattern matching**: Use Glob and Grep to find likely matches by name, keywords, and patterns
3. **Deep reading**: Read the most relevant files in detail

### 3. Build an Organized Understanding

Identify:
- Which files, directories, and modules are involved
- Main functions, types, or components that implement the feature
- Related configuration (YAML, env, Terraform, Kubernetes, CI, etc.)
- Important relationships:
  - Call chains or data flows
  - Entry points and public APIs
  - Integration with external services or other modules

### 4. Report Back Clearly

Produce a structured report including:
- **Topic**: The research question answered
- **Locations**: List of relevant paths (files/directories)
- **Summary**: Short explanation for each location
- **Architecture notes**: Key interactions or data flows
- **Follow-ups**: Suggestions for next investigations

## Output Format

Use the template at `references/templates/research-findings.md` for comprehensive reports.

For quick answers, use this minimal format:

```markdown
## Research: [Topic]

### Locations
- `path/to/file` — [what it does]
- `path/to/dir/` — [what it contains]

### Summary
[How these pieces work together]

### Architecture Notes
[Key data flows or interactions]

### Follow-ups
- [Suggestion for next investigation]
```

## Constraints

- **Read-only**: Do NOT modify any files
- **No implementation**: Do NOT create tests or code changes
- **Focused**: Keep the report focused on the requested topic
- **High-value**: Prefer insights over exhaustive file listings
- **Clear**: Structure findings so a human can quickly follow

## Example

```
Research Question: "How does authentication work?"

Research Goal: Identify the main entry points for authentication
and session handling.

Locations:
- `pkg/auth/handler.go` — HTTP handlers for login/logout/refresh
- `pkg/auth/jwt.go` — JWT token generation and validation
- `pkg/auth/middleware.go` — Authentication middleware for protected routes
- `pkg/auth/session/` — Session storage and management
- `config/auth.yaml` — Authentication configuration (token TTL, providers)

Summary:
Authentication uses JWT tokens with refresh token rotation. The
middleware validates tokens on protected routes and injects user
context. Sessions are stored in Redis with configurable TTL.

Architecture Notes:
- Login flow: handler.go → jwt.go (generate) → session/ (store)
- Request flow: middleware.go → jwt.go (validate) → inject user context
- Supports OAuth providers via config/auth.yaml

Follow-ups:
- Review token refresh logic for security
- Document the OAuth provider setup process
- Add integration tests for session expiration
```

Begin by interpreting the research question and planning the exploration strategy.
