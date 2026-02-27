---
name: multi-claude
description: Run parallel Claude instances for writer+reviewer patterns, git worktrees, and specialist agents. Use for independent verification and parallel progress.
---

# Multi-Claude Workflows Skill

## Trigger
Use when you need parallel progress, independent verification, or separation of concerns between different aspects of a task.

## The Insight
From Anthropic's Claude Code best practices: "Run separate Claude instances in parallel—one writing code, another reviewing. Use git worktrees to enable simultaneous independent tasks."

## Workflow Patterns

### Pattern 1: Writer + Reviewer
Run two Claude instances with different roles:

**Terminal 1 - Writer:**
```bash
cd ~/project
claude
# "Implement the new caching layer"
```

**Terminal 2 - Reviewer:**
```bash
cd ~/project
claude
# "Review the changes being made to src/cache/.
#  Look for bugs, edge cases, and improvements."
```

Benefits:
- Fresh eyes catch issues writer misses
- Reviewer isn't biased by implementation decisions
- Parallel thinking on same problem

### Pattern 2: Git Worktrees for Parallel Tasks
Create multiple working directories for the same repo:

```bash
# Create worktrees for parallel work
git worktree add ../project-feature-a feature-a
git worktree add ../project-feature-b feature-b
git worktree add ../project-bugfix bugfix-123
```

Now run separate Claude instances:

**Terminal 1:**
```bash
cd ../project-feature-a
claude
# "Implement feature A"
```

**Terminal 2:**
```bash
cd ../project-feature-b
claude
# "Implement feature B"
```

**Terminal 3:**
```bash
cd ../project-bugfix
claude
# "Fix bug #123"
```

Benefits:
- No git conflicts between parallel work
- Each Claude has clean working state
- Merge when each is complete

### Pattern 3: Specialist Agents
Different Claude instances for different expertise:

**Terminal 1 - Backend:**
```bash
claude
# "Implement the API endpoints for user management"
```

**Terminal 2 - Frontend:**
```bash
claude
# "Implement the React components for user management UI"
```

**Terminal 3 - Tests:**
```bash
claude
# "Write integration tests for the user management feature"
```

### Pattern 4: Research + Implementation
One instance explores, another implements:

**Terminal 1 - Research:**
```bash
claude
# "Research how authentication is handled in this codebase.
#  Document the patterns, key files, and flows."
```

**Terminal 2 - Implementation (after research):**
```bash
claude
# "Using the auth patterns documented in RESEARCH.md,
#  implement OAuth2 login for the mobile app"
```

## Setup Guide

### Using Git Worktrees

```bash
# List existing worktrees
git worktree list

# Add a new worktree
git worktree add <path> <branch>

# Remove a worktree when done
git worktree remove <path>

# Prune stale worktree info
git worktree prune
```

### Terminal Management
Use terminal multiplexer (tmux/screen) or IDE terminals:

```bash
# tmux example
tmux new-session -s claude-writer
# Ctrl-b c to create new window
# Ctrl-b n/p to switch windows
# Ctrl-b d to detach
```

### Coordinating Between Instances
Create a shared document for coordination:

```markdown
# COORDINATION.md

## Instance Assignments
- Terminal 1: API implementation
- Terminal 2: Frontend components
- Terminal 3: Test coverage

## Shared Decisions
- Using REST not GraphQL
- Auth tokens in headers
- Error format: { error: string, code: number }

## Interface Contracts
- POST /api/users returns { id, email, createdAt }
- User component expects props: { user: User, onUpdate: fn }

## Blockers
- [ ] Need DB schema before API work
- [ ] Need API spec before frontend work
```

## When to Use Multi-Claude

| Scenario | Pattern |
|----------|---------|
| Large feature with distinct parts | Specialist agents |
| Need independent code review | Writer + Reviewer |
| Multiple unrelated tasks | Git worktrees |
| Complex research + implementation | Research + Implementation |
| Time-sensitive parallel work | Multiple worktrees |

## Anti-Patterns

**Don't:**
- Have instances edit the same files simultaneously
- Forget to merge/integrate parallel work
- Let instances make conflicting architectural decisions
- Skip coordination document for complex parallel work

**Do:**
- Define clear boundaries between instances
- Use worktrees to avoid file conflicts
- Coordinate on shared interfaces/contracts
- Merge and test integration regularly
