---
name: work
description: Work incrementally on Issues with Git worktree support for parallel agent workflows. Use when starting a coding session, continuing work, or running parallel agents.
---

# Work - Incremental Implementation

Make incremental progress on Issues, leaving clean state for next session.

Based on [Anthropic's Coding Agent pattern](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents).

## Arguments

This skill accepts an optional issue path:
```
/work [issue-path]
```

Examples:
- `/work` - Continue work on current issue from Epic progress.md
- `/work issues/20251227-task05-config-store` - Work on specific issue

## When to Use

- Starting a new coding session
- User says "continue", "work on issue", "next feature"
- Running AI agents on parallel issues (via worktrees)

## Session Workflow

```
1. GET BEARINGS        →  pwd, git worktree list, cat epics/*/progress.md
       ↓
2. READ ISSUE          →  Read issue.md and feature.json from issue directory
       ↓
3. VERIFY BASELINE     →  Run build/tests, fix any broken state
       ↓
4. IMPLEMENT           →  One feature at a time: implement → test → commit
       ↓
5. END SESSION         →  Use /commit skill, update Epic progress.md
```

## Directory Structure

```
project/
├── epics/
│   ├── cli/
│   │   ├── epic.md
│   │   ├── progress.md               # Check here for current work
│   │   └── references/
│   └── website/
│       ├── epic.md
│       ├── progress.md
│       └── references/
└── issues/
    ├── 20250115-task01-auth-setup/
    │   ├── issue.md                  # Requirements
    │   └── feature.json              # Acceptance criteria
    └── ...
```

## Reading the Issue

Each issue directory contains:
- `issue.md` - Requirements, technical details, API design
- `feature.json` - Testable acceptance criteria with pass/fail status

```bash
# Read issue to understand the work
cat issues/ISSUE_ID/issue.md
cat issues/ISSUE_ID/feature.json
```

## Core Loop: Implement One Feature

```bash
# 1. Implement the feature
# 2. Test end-to-end (browser/HTTP/CLI)
# 3. Update feature.json: "passes": true
# 4. Use /commit skill to commit changes
```

## Rules

**DO:**
- Start every session with "Get Bearings"
- Read issue.md and feature.json before implementing
- Work on ONE feature at a time
- Test BEFORE marking `passes: true`
- Use `/commit` skill for commits (not manual git commit)
- Update Epic's progress.md at session end

**DO NOT:**
- Skip baseline verification
- Mark features passing without testing
- Leave half-implemented code
- Commit without using the commit skill

## Launch Agent in New Terminal

When user says "work on [issue]" with a specific issue, open a new Ghostty terminal and run the agent there:

```bash
# Get current directory first
PROJ_DIR=$(pwd)

# Open Ghostty and run claude agent in that directory
open -a Ghostty --args -e zsh -lc "cd $PROJ_DIR && claude --dangerously-skip-permissions"
```

This allows:
- Running an autonomous agent in a separate terminal
- Continuing work in the current terminal
- Agent runs with full permissions for unattended operation

## Parallel Development (Worktrees)

Use Git worktrees to run agents on separate issues in parallel:

```bash
# Create worktree for agent
git worktree add ../project-task02 -b task02
cd ../project-task02
claude --print "Complete task02 per issue.md"

# Continue your work in main repo
cd ../project
```

See: [worktree-commands.md](references/worktree-commands.md) | [parallel-agents.md](references/parallel-agents.md)

## Handling Issues

| Situation | Action |
|-----------|--------|
| Broken baseline | Fix FIRST, then new work |
| Blocked by dependency | Document, switch issue or end session |
| Feature fails testing | Keep `passes: false`, fix and re-test |

## Progress File Format

Update the Epic's `progress.md`:

```markdown
#### [YYYY-MM-DD HH:MM] Session #N
- Working on: task02-auth
- Completed: F001, F002
- In progress: F003
- Commits: abc123, def456
- Notes:
  - Brief description of what was done
  - Any important decisions or changes
```
