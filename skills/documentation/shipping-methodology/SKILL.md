---
name: shipping-methodology
description: Use when running claudikins-kernel:ship, preparing PRs, writing changelogs, deciding merge strategy, or handling CI failures — enforces GRFP-style iterative approval, code integrity validation, and human-gated merges
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - Edit
  - AskUserQuestion
  - Skill
  - mcp__plugin_claudikins-tool-executor_tool-executor__search_tools
  - mcp__plugin_claudikins-tool-executor_tool-executor__get_tool_schema
  - mcp__plugin_claudikins-tool-executor_tool-executor__execute_code
---

# Shipping Methodology

## When to use this skill

Use this skill when you need to:

- Run the `claudikins-kernel:ship` command
- Prepare commit messages and PR descriptions
- Update changelogs and documentation
- Decide merge strategy (squash vs preserve)
- Handle CI failures or merge conflicts
- Validate code integrity before shipping

## Core Philosophy

> "Ship with confidence, not hope." - Shipping philosophy

Shipping is the final gate. Apply GRFP-style iterative workflow to every stage.

### The Five Principles

1. **Gate check first** - claudikins-kernel:verify must have passed. No exceptions.
2. **Code integrity** - Ship exactly what was verified. No sneaky changes.
3. **GRFP everywhere** - Section-by-section approval at every stage.
4. **Human decides** - No auto-merging. Human approves final merge.
5. **Clean up after** - Delete branches, update docs, celebrate.

## The Five Stages

### Stage 1: Pre-Ship Review

Show what's being shipped. Human confirms ready.

| Check               | What to Show                                    |
| ------------------- | ----------------------------------------------- |
| Verification status | All phases PASS from claudikins-kernel:verify   |
| Branches to merge   | List all execute/task-\* branches               |
| Evidence summary    | Screenshots, curl responses from catastrophiser |
| Code delta          | Lines added/removed, files changed              |

**Checkpoint:**

```
Ready to ship?

Verified: ✓ All checks passed
Branches: 3 branches to merge
Changes: +450 / -120 lines

[Continue] [Back to Verify] [Abort]
```

### Stage 2: Commit Strategy

Draft commit message(s). Human approves.

**Decision:**

```
How should we commit?

[Squash into single commit] [Preserve commit history]
```

**Squash (recommended for features):**

- Single clean commit on main
- Clear feature boundary
- Easier to revert if needed

**Preserve (for large multi-part work):**

- Keeps granular history
- Better for debugging
- Use for multi-feature batches

**Checkpoint:**

```
Commit message:

feat(auth): Add authentication middleware

- JWT token validation
- Role-based access control
- Session management

Closes #42

[Accept] [Revise] [Back]
```

### Stage 3: Documentation (git-perfectionist)

Update README, CHANGELOG, version. GRFP-style.

**git-perfectionist uses github-readme plugin:**

1. Deep-dive on current docs
2. Identify gaps from changes
3. Pen-wielding for updates
4. Section-by-section approval

**Files to update:**

| File                                       | What to Update                           |
| ------------------------------------------ | ---------------------------------------- |
| README.md                                  | Features, usage, installation if changed |
| CHANGELOG.md                               | Add entry in Keep a Changelog format     |
| package.json / Cargo.toml / pyproject.toml | Version bump if needed                   |

**Checkpoint:**

```
Documentation updates:

README.md:
- Added: Authentication section
- Updated: Installation (new deps)

CHANGELOG.md:
- Added: v1.2.0 entry

Version: 1.1.0 → 1.2.0 (minor)

[Accept] [Revise] [Skip]
```

### Stage 4: PR Creation

Draft PR title and body. Section-by-section approval.

**PR Title Pattern:**

```
feat(scope): Short description
```

**PR Body Structure:**

```markdown
## Summary

[2-3 bullet points of what changed]

## Changes

[Detailed breakdown]

## Testing

[How it was verified]

## Screenshots

[If applicable]
```

**Checkpoint:**

```
PR ready to create:

Title: feat(auth): Add authentication middleware

Body:
## Summary
- Added JWT token validation
- Implemented role-based access control
...

[Create PR] [Revise] [Back]
```

### Stage 5: Final Merge

CI passes. Human approves. Merge and cleanup.

**CI Status Check:**

```
CI Status: ⏳ Running...

[Wait for CI] [View logs] [Merge anyway]
```

**On CI pass:**

```
CI Status: ✓ All checks passed

Ready to merge PR #42 to main?

[Merge] [Request review first] [Cancel]
```

**After merge:**

- Delete feature branches (unless --no-delete-branch)
- Update ship-state.json
- Celebrate

**Final output:**

```
Done! Shipped to main.

PR #42 merged ✓
Branches cleaned up ✓
Version: 1.1.0 → 1.2.0

Nice work!
```

## Rationalizations to Resist

Agents under pressure find excuses. These are all violations:

| Excuse                                      | Reality                                                                  |
| ------------------------------------------- | ------------------------------------------------------------------------ |
| "Verify passed yesterday, close enough"     | Stale verification = no verification. Re-run claudikins-kernel:verify.   |
| "Just a tiny fix after verify, no big deal" | Any change after verify invalidates it. Re-run claudikins-kernel:verify. |
| "CI is flaky, I'll merge anyway"            | Flaky CI hides real failures. Fix or explicitly skip with caveat.        |
| "It's just a typo, skip the PR"             | All changes go through PR. No exceptions.                                |
| "Both reviewers passed, auto-merge is fine" | Human approves final merge. Always.                                      |
| "I'll update the changelog later"           | Changelog is part of shipping. Do it now.                                |
| "Force push is fine, it's my branch"        | Never force push to protected branches. Ever.                            |
| "Skip docs, nobody reads them"              | Docs are part of shipping. GRFP them.                                    |
| "The gate check is too strict"              | The gate exists for a reason. Pass it properly.                          |

**All of these mean: Follow the methodology. Shortcuts create incidents.**

## Red Flags — STOP and Reassess

If you're thinking any of these, you're about to violate the methodology:

- "Let me just push this one change..."
- "Verify is probably still valid"
- "CI will pass next time"
- "I'll do the PR properly next time"
- "Auto-merge saves time"
- "Docs can wait"
- "Force push will fix it"
- "The human already approved something similar"

**All of these mean: STOP. Gate check exists. Human decides. Follow the stages.**

## Gate Check Pattern (CRITICAL)

The ship-init.sh hook enforces the claudikins-kernel:verify gate with code integrity validation.

### Required Checks

```bash
# 1. Verify state exists
if [ ! -f "$VERIFY_STATE" ]; then
  echo "ERROR: claudikins-kernel:verify has not been run"
  exit 2
fi

# 2. Unlock flag is set
UNLOCK=$(jq -r '.unlock_ship // false' "$VERIFY_STATE")
if [ "$UNLOCK" != "true" ]; then
  echo "ERROR: claudikins-kernel:verify did not pass or was not approved"
  exit 2
fi
```

### Code Integrity (C-5, C-7)

Ensure we ship exactly what was verified:

```bash
# C-5: Commit hash validation
VERIFY_COMMIT=$(jq -r '.verified_commit_sha' "$VERIFY_STATE")
CURRENT_COMMIT=$(git rev-parse HEAD)
if [ "$VERIFY_COMMIT" != "$CURRENT_COMMIT" ]; then
  echo "ERROR: Code changed since verification"
  exit 2
fi

# C-7: File manifest validation
VERIFIED_MANIFEST=$(jq -r '.verified_manifest' "$VERIFY_STATE")
CURRENT_MANIFEST=$(sha256sum "$MANIFEST_FILE" | cut -d' ' -f1)
if [ "$VERIFIED_MANIFEST" != "$CURRENT_MANIFEST" ]; then
  echo "ERROR: Source files changed after verification"
  exit 2
fi
```

**If integrity check fails:**

```
Code has changed since verification.

Verified commit: abc123
Current commit:  def456

[Re-run claudikins-kernel:verify] [Abort]
```

## Commit Message Patterns

### Feature

```
feat(scope): Short description

- Bullet point of what changed
- Another change

Closes #123
```

### Bug Fix

```
fix(scope): Short description

Root cause: What was wrong
Fix: What we did

Fixes #456
```

### Breaking Change

```
feat(scope)!: Short description

BREAKING CHANGE: What breaks and migration path

- Change 1
- Change 2
```

### Chore (no user-facing change)

```
chore(scope): Short description

- Internal refactor
- Dependency update
```

See [commit-message-patterns.md](references/commit-message-patterns.md) for full guide.

## CHANGELOG Format

Follow Keep a Changelog format:

```markdown
## [Unreleased]

## [1.2.0] - 2026-01-17

### Added

- Authentication middleware with JWT support (#42)

### Changed

- Updated error messages for clarity

### Fixed

- Token refresh race condition (#38)

## [1.1.0] - 2026-01-10

...
```

See [changelog-merge-strategy.md](references/changelog-merge-strategy.md) for merging entries.

## External Service Failures

### GitHub API Failure (E-6, E-7, E-8)

```
PR creation failed.

Error: GitHub API returned 503

Retry 1/3...
```

**Pattern:** Max 3 retries with exponential backoff.

**If persistent:**

```
GitHub API unavailable after 3 retries.

[Try again] [Save as draft PR] [Manual merge]
```

Save state with `pending_merge: true` for recovery.

### CI Failure

```
CI failed.

Failed checks:
- lint: 2 errors
- test: 1 failure

[View logs] [Fix and retry] [Skip CI] [Abort]
```

See [ci-failure-handling.md](references/ci-failure-handling.md) for recovery patterns.

## Force Push Protection (S-23)

Never force push to protected branches.

```bash
# Check if target is protected
PROTECTED_BRANCHES="main master release"
if echo "$PROTECTED_BRANCHES" | grep -q "$TARGET"; then
  if [ "$FORCE" = "true" ]; then
    echo "ERROR: Cannot force push to $TARGET"
    exit 2
  fi
fi
```

See [force-push-protection.md](references/force-push-protection.md) for safeguards.

## Breaking Change Detection (S-24)

Detect breaking changes before shipping:

| Signal                     | Indicates            |
| -------------------------- | -------------------- |
| Removed public function    | Breaking             |
| Changed function signature | Breaking             |
| Removed config option      | Breaking             |
| Changed default behaviour  | Potentially breaking |

**If breaking change detected:**

```
Breaking change detected!

- Removed: authenticate() function
- Changed: login() now requires email

This requires a MAJOR version bump.

[Acknowledge and continue] [Abort]
```

See [breaking-change-detection.md](references/breaking-change-detection.md) for detection patterns.

## Anti-Patterns

**Don't do these:**

- Shipping without claudikins-kernel:verify passing
- Modifying code after verification
- Force pushing to main/master
- Auto-merging without human approval
- Skipping documentation updates
- Ignoring CI failures
- Using vague commit messages ("fix stuff")
- Forgetting to delete feature branches

## State Tracking

### ship-state.json

```json
{
  "session_id": "ship-2026-01-17-1130",
  "verify_session_id": "verify-2026-01-17-1100",
  "target": "main",
  "phases": {
    "pre_ship_review": { "status": "APPROVED" },
    "commit_strategy": { "status": "APPROVED", "strategy": "squash" },
    "documentation": {
      "status": "APPROVED",
      "files_updated": ["README.md", "CHANGELOG.md"]
    },
    "pr_creation": { "status": "CREATED", "pr_number": 42 },
    "merge": { "status": "MERGED", "sha": "abc123" }
  },
  "cleanup": {
    "branches_deleted": ["execute/task-1-auth", "execute/task-2-routes"]
  },
  "shipped_at": "2026-01-17T11:46:00Z"
}
```

## References

Full documentation in this skill's references/ folder:

- [commit-message-patterns.md](references/commit-message-patterns.md) - Full commit message guide
- [grfp-checklist.md](references/grfp-checklist.md) - GRFP integration checklist
- [pr-creation-strategy.md](references/pr-creation-strategy.md) - PR body templates
- [deployment-checklist.md](references/deployment-checklist.md) - Pre-deploy checks
- [gate-failure-recovery.md](references/gate-failure-recovery.md) - Recovering from gate failures (S-19)
- [ci-failure-handling.md](references/ci-failure-handling.md) - Handling CI pipeline failures (S-20)
- [message-generation-fallback.md](references/message-generation-fallback.md) - When commit message generation fails (S-21)
- [changelog-merge-strategy.md](references/changelog-merge-strategy.md) - Merging CHANGELOG entries (S-22)
- [force-push-protection.md](references/force-push-protection.md) - Preventing accidental force pushes (S-23)
- [breaking-change-detection.md](references/breaking-change-detection.md) - Detecting breaking changes (S-24)
