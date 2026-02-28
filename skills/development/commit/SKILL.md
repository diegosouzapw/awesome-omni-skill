---
name: commit
description: Analyze unstaged and staged changes, suggest atomic commit groups with conventional commit messages. NEVER pushes to remote.
license: MIT
compatibility:
  - vcs:git
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash(git:status)
  - Bash(git:diff)
  - Bash(git:log)
  - Bash(git:add)
  - Bash(git:commit)
metadata:
  author: thoreinstein
  version: 1.0.0
---

# Commit

Analyze unstaged changes and organize them into atomic, well-documented commits.

## When to Use This Skill

- After completing work that spans multiple logical changes
- When working tree has mixed changes (features, fixes, refactors)
- To ensure clean, reviewable git history
- Before creating a pull request

## Workflow

1. **Analyze current state:**
   - Check `git status` to see all staged and unstaged changes
   - Review `git diff` to understand what changed

2. **Group changes into atomic units:**
   - Each group should represent one logical change (feature, fix, refactor, docs, etc.)
   - Group related files and hunks together
   - Keep changes that depend on each other in the same commit or note the dependency

3. **For each group, provide:**
   - List of files/hunks to stage
   - Commit message following project conventions
   - Brief rationale for the grouping

4. **Note dependencies:**
   - If commits must be applied in a specific order, document this
   - Identify any commits that could be applied independently

5. **Execute commits:**
   - Stage and commit each group in the appropriate order
   - Proceed unless there are questions or ambiguities

## Optional Focus

You can provide a focus to filter which changes to consider:

- `"frontend only"` - only consider frontend-related changes
- `"exclude tests"` - skip test file changes
- `"api changes"` - focus on API-related files
- Any other relevant filter for the codebase

## Constraints

- **NEVER push to remote** - this skill only creates local commits
- **Atomic commits** - each commit should be a single logical unit that could be reverted independently
- **Respect project conventions** - match existing commit message style in the repository
- **Ask before proceeding** - if grouping is ambiguous, ask for clarification rather than guessing

## Examples

### Example: Mixed Working Tree

**Situation:** Working tree has changes to API handler, its tests, a bug fix in an unrelated util, and a README update.

**git status output:**
```
modified:   internal/api/users.go
modified:   internal/api/users_test.go
modified:   internal/util/strings.go
modified:   README.md
```

**Proposed commit groups:**

**Group 1: Bug fix (independent)**
```
Files: internal/util/strings.go
Message: fix: handle empty string edge case in TrimSafe

Rationale: Standalone fix, no dependencies on other changes
```

**Group 2: Feature (with tests)**
```
Files: internal/api/users.go, internal/api/users_test.go
Message: feat: add email validation to user creation endpoint

Rationale: Handler and its tests belong together as one logical unit
```

**Group 3: Documentation (independent)**
```
Files: README.md
Message: docs: add API usage examples

Rationale: Documentation update, independent of code changes
```

**Suggested order:** Group 1 → Group 2 → Group 3 (no strict dependencies)

---

Begin by running git status and git diff to analyze the current working tree state.
