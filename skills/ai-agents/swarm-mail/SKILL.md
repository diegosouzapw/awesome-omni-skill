---
name: swarm-mail
description: Coordinate with other agents using Swarm mail and file locking
globs:
  - "**/*"
alwaysApply: true
---

# Swarm Multi-Agent Coordination

You are part of a Swarm deployment with multiple AI agents working on the same codebase.
Use these patterns to coordinate effectively.

## When to Use Mail vs Queue vs Direct

- Mail: Handoff tasks to specific agents, request reviews, report completion
- Queue: Send work to yourself for later (cooldown, complex sequences)
- Direct: Only for emergency interrupts (use `swarm inject`)

## Writing Actionable Handoff Messages

Good handoff messages require no follow-up questions.

### Good Example
Subject: Review PR #123 - User authentication refactor
Body:
- PR is ready for review at https://github.com/...
- Focus on: error handling in login flow
- Tests pass locally, CI pending
- After review, send results to agent-xyz

### Bad Example
Subject: Please review
Body: The PR is ready.

## Advisory File Locking

Before editing files, claim a lock:

```bash
swarm lock claim --agent $AGENT_ID --path "src/api/auth.go" --ttl 30m
```

Check for conflicts before claiming:

```bash
swarm lock check --path "src/api/auth.go"
```

Always release when done:

```bash
swarm lock release --agent $AGENT_ID
```

## Subject/Body Conventions

Subjects should be:
- Specific: "Fix null pointer in UserService.getById()"
- Searchable: Include file names, function names, issue numbers
- Actionable: Start with verb (Fix, Review, Implement, Update)

## Checking Your Inbox

Poll for new work regularly:

```bash
swarm mail inbox --agent $AGENT_ID --unread
```

## Additional template files

See the templates in `templates/`:
- templates/handoff.md
- templates/review-request.md
- templates/conflict-resolution.md
