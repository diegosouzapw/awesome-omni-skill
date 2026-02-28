---
name: advanced-git
description: Advanced Git - interactive rebase, cherry-pick, bisect, reflog, and power user operations
sasmp_version: "1.3.0"
bonded_agent: git-expert
bond_type: SECONDARY_BOND
category: development
version: "2.0.0"
triggers:
  - git reflog
  - git bisect
  - cherry pick
  - interactive rebase
---

# Advanced Git Skill

> **Production-Grade Development Skill** | Version 2.0.0

**Power user Git operations and debugging.**

## Skill Contract

### Input Schema
```yaml
input:
  type: object
  properties:
    operation:
      type: string
      enum: [reflog, bisect, cherry-pick, rebase-interactive, stash, worktree]
    target:
      type: string
      description: Commit hash, branch, or range
    options:
      type: object
      properties:
        dry_run:
          type: boolean
          default: true  # Safe default
        confirm_destructive:
          type: boolean
          default: false
```

### Output Schema
```yaml
output:
  type: object
  required: [result, success, risk_level]
  properties:
    result:
      type: string
    success:
      type: boolean
    risk_level:
      type: string
      enum: [low, medium, high, critical]
    rollback_command:
      type: string
```

## Error Handling

### Retry Logic
```yaml
retry_config:
  max_attempts: 2
  backoff_ms: [1000, 2000]
  retryable:
    - timeout
  non_retryable:
    - conflict
    - invalid_reference
```

### Safety Checks
```yaml
pre_operation_checks:
  - verify_clean_working_tree
  - verify_not_on_shared_branch
  - create_backup_ref
```

---

## Recovery Operations

### Reflog: Your Safety Net
```bash
git reflog
# abc1234 HEAD@{0}: commit: Add feature
# def5678 HEAD@{1}: rebase: finished

# Recover "lost" commit
git branch recovery-branch abc1234
```

## Interactive Rebase

```bash
git rebase -i HEAD~5

# Commands:
# pick   - keep commit
# reword - edit message
# squash - merge with previous
# drop   - remove commit
```

## Cherry-Pick

```bash
git cherry-pick abc1234           # Single commit
git cherry-pick abc1234^..def5678 # Range
git cherry-pick --abort           # On conflict
```

## Git Bisect

```bash
git bisect start
git bisect bad
git bisect good abc1234
# Test and mark good/bad until found
git bisect reset
```

## Stash Operations

```bash
git stash push -m "WIP feature"
git stash list
git stash apply stash@{0}
git stash pop
```

---

## Troubleshooting Guide

### Debug Checklist
```
□ 1. Current HEAD? → git rev-parse HEAD
□ 2. Reflog available? → git reflog
□ 3. Operation in progress? → ls .git/
```

### Common Issues

| Error | Cause | Solution |
|-------|-------|----------|
| "could not apply" | Cherry-pick conflict | Resolve and continue |
| "refusing to rebase" | Uncommitted changes | Stash first |

---

## Command Risk Matrix

| Command | Risk Level |
|---------|------------|
| `git reflog` | LOW |
| `git stash` | LOW |
| `git cherry-pick` | MEDIUM |
| `git rebase -i` | HIGH |
| `git filter-repo` | CRITICAL |

---

## Observability

```yaml
logging:
  level: DEBUG
  events:
    - operation_started
    - backup_created
    - conflict_detected

metrics:
  - rebase_success_rate
  - recovery_operations
```

---

*"With great power comes great responsibility - and the ability to undo mistakes."*
