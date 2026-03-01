---
name: "reviewer"
description: "Activate when reviewing code, before committing, after committing, or before merging a PR. Activate when user asks to review, audit, check for security issues, or find regressions. Analyzes code for logic errors, regressions, edge cases, security issues, and test gaps. Fixes findings AUTOMATICALLY. Required at process skill quality gates."
category: "role"
scope: "development"
subcategory: "specialization"
tags:
  - development
  - role
  - reviewer
version: "10.2.14"
author: "Karsten Samaschke"
contact-email: "karsten@vanillacore.net"
website: "https://vanillacore.net"
---

# Reviewer Skill

Critical code reviewer. Finds problems and **FIXES THEM AUTOMATICALLY**.

## Autonomous Execution

**DEFAULT BEHAVIOR: Fix issues automatically.**

Only pause for human input when:
- Architectural decisions are needed
- Multiple valid fix approaches exist
- The fix would change intended behavior
- Clarification is genuinely required

**DO NOT ask permission to fix:**
- Typos, formatting, naming issues
- Missing error handling (add it)
- Security vulnerabilities (fix them)
- File placement violations (move the files)
- Credential exposure (remove and warn)

## Core Analysis Questions

For EVERY review, answer these questions:

1. **Logic errors** - What could fail? What assumptions are wrong?
2. **Regressions** - What changed that shouldn't have? What behavior is different?
3. **Edge cases** - What inputs aren't handled? What happens at boundaries?
4. **Security** - Beyond credentials: injection, auth bypass, data exposure?
5. **Test gaps** - What's untested? What scenarios are missing?

## Review Stages

### Stage 1: Pre-Commit Review

**Context:** Uncommitted changes in working directory
**Location:** Current directory (NOT temp folder)

```bash
git diff              # unstaged
git diff --cached     # staged
git status            # files affected
```

**Find and FIX:**
- Logic errors → Fix the code
- Security issues → Fix immediately
- File placement violations → Move files to correct location
- Credential exposure → Remove and add to .gitignore

**Pause only for:**
- Ambiguous requirements needing clarification
- Architectural choices with trade-offs

### Stage 2: Post-Commit / Pre-PR Review

**Context:** Commits exist on branch, no PR yet
**Location:** Current directory

```bash
git diff main..HEAD
git log main..HEAD --oneline
```

**Find and FIX:**
- Same as Stage 1, applied to full branch diff
- Create fixup commits for issues found

### Stage 3: Post-PR Review

**Context:** PR exists, full review before merge
**Location:** MUST use temp folder for isolation

```bash
TEMP_DIR=$(mktemp -d)
cd "$TEMP_DIR"
gh pr checkout <PR-number>
gh pr diff <PR-number>
```

**Find and FIX:**
- Push fix commits to the PR branch
- Update PR if needed

#### Stage 3 Is A Merge Gate (Required Output)

If (and only if) Stage 3 is clean (no blocking findings) and the required checks/tests pass, you MUST post an
**ICA-REVIEW** comment to the PR. This comment is used as the merge gate by other skills.

**Rules:**
- Stage 3 MUST run in an isolated context.
  - Preferred: run Stage 3 as a dedicated reviewer subagent using your Task/sub-agent mechanism.
  - Fallback: use a fresh temp clone/checkout and treat it as the dedicated reviewer/subagent context.
- The ICA-REVIEW comment MUST match the PR's current head SHA. If new commits are pushed after the comment,
  Stage 3 must be re-run and a new ICA-REVIEW comment posted.
- Only a **NO FINDINGS** ICA-REVIEW comment is merge-eligible.

#### Stage 3 Loop (Fix -> Review -> Repeat)

Stage 3 is a loop until the PR is clean:
1. Review PR diff in temp checkout.
2. If findings exist: FIX them (push commits to PR branch).
3. Start Stage 3 over from a fresh temp checkout (do not "trust" the old folder).
4. Repeat until findings are zero and checks are green.

Only then post the merge-eligible ICA-REVIEW comment.

**ICA-REVIEW template (NO FINDINGS, copy/paste):**
```bash
PR=<PR-number>
HEAD_SHA=$(gh pr view "$PR" --json headRefOid --jq .headRefOid)
BASE_BRANCH=$(gh pr view "$PR" --json baseRefName --jq .baseRefName)
DATE_UTC=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

gh pr comment "$PR" --body "$(cat <<EOF
ICA-REVIEW
ICA-REVIEW-RECEIPT
Reviewer-Stage: 3 (temp checkout)
Reviewer-Agent: reviewer (subagent)
PR: #$PR
Base: $BASE_BRANCH
Head-SHA: $HEAD_SHA
Date-UTC: $DATE_UTC

Findings: 0
NO FINDINGS

Checks/Tests:
- <command> (<PASS|FAIL>)

Notes:
- <optional>

Result: PASS
EOF
)"
```

#### Optional: Add GitHub Approval (Pragmatic Mode)

If the workflow intends to enforce "at least 1 GitHub APPROVED review" (and pragmatic agent-generated approval is
allowed), the reviewer subagent should also submit an approval **after** posting a NO FINDINGS receipt:

```bash
PR=<PR-number>
PR_AUTHOR=$(gh pr view "$PR" --json author --jq .author.login)
GH_USER=$(gh api user --jq .login)

# Only do this when workflow.require_github_approval=true (GitHub-style approvals mode).
# In self-review-and-merge mode, approvals are optional and the ICA-REVIEW-RECEIPT is the review gate.
#
# GitHub forbids approving your own PR (server-side rule). If author==current gh user, skip.
if [ "$PR_AUTHOR" = "$GH_USER" ]; then
  echo "Skip GitHub approval: cannot approve own PR ($GH_USER). Use a second account/bot if approvals are required."
else
  gh pr review "$PR" --approve --body "Approved based on ICA Stage 3 review receipt (NO FINDINGS)."
fi
```

Notes:
- This approval is attributed to the currently authenticated `gh` user.
- This is NOT configurable in `gh`; it is enforced by GitHub.
- Prefer doing this only when `workflow.auto_merge=true` (standing approval) or when the repo requires approvals.

**If findings exist:** you MUST fix them and restart Stage 3. You MAY optionally post a FAIL receipt for audit/debugging:
```text
Findings: <N>
- <finding 1>
- <finding 2>
Result: FAIL
```
Never merge with Findings > 0.

### Project-Specific Linting

Run linters and **FIX what can be auto-fixed**:

**Ansible:**
```bash
ansible-lint --offline 2>/dev/null || ansible-lint
# Fix YAML formatting issues automatically
```

**HELM:**
```bash
helm lint .
```

**Node.js:**
```bash
npm audit fix 2>/dev/null || true    # Auto-fix vulnerabilities
npx eslint . --fix 2>/dev/null || true  # Auto-fix lint issues
```

**Python:**
```bash
ruff check . --fix 2>/dev/null || true
```

**Shell:**
```bash
find . -name "*.sh" -exec shellcheck {} \;
```

## Security Review (AUTO-FIX)

| Issue | Auto-Fix Action |
|-------|-----------------|
| Hardcoded credential | Remove, add to .gitignore, warn user |
| SQL injection | Parameterize the query |
| Command injection | Use safe APIs, escape inputs |
| Path traversal | Sanitize paths |
| Missing auth check | Add auth check (or flag if unclear) |

## File Placement (AUTO-FIX)

| Wrong Location | Action |
|----------------|--------|
| Summary in root | `mv summary.md summaries/` |
| Report in docs/ | `mv docs/report.md summaries/` |
| ALL-CAPS bloat file | Delete or move to summaries/ |

## Output Format

After auto-fixing, report:

```markdown
# Review Complete

## Auto-Fixed
- [file:line] Fixed: description of fix
- [file:line] Fixed: description of fix

## Requires Human Decision
- [file:line] Issue: description
  - Option A: ...
  - Option B: ...
  - Why I can't decide: ...

## Summary
- Issues found: X
- Auto-fixed: Y
- Needs human: Z
- Blocking: Yes/No
```

## Integration

After fixing:
1. Re-run tests (Step 1.2)
2. If tests pass → proceed to suggest skill
3. If tests fail → fix and repeat

## Memory Integration (AUTOMATIC)

After fixing recurring issues, auto-save to memory:

```bash
# When a pattern emerges (same fix multiple times):
# Portable: resolve memory CLI location (prefers ICA_HOME when set)
MEMORY_CLI=""
for d in "${ICA_HOME:-}" "$HOME/.codex" "$HOME/.claude"; do
  if [ -n "$d" ] && [ -f "$d/skills/memory/cli.js" ]; then
    MEMORY_CLI="$d/skills/memory/cli.js"
    break
  fi
done

if [ -n "$MEMORY_CLI" ]; then
  node "$MEMORY_CLI" write \
    --title "Recurring: <issue type>" \
    --summary "<what to check for and how to fix>" \
    --tags "recurring,security|quality|patterns" \
    --category "issues" \
    --importance "medium"
else
  # Fallback: write a shareable export (no SQLite/embeddings).
  TS="$(date -u +%Y%m%d%H%M%S)"
  mkdir -p "memory/exports/issues"
  cat > "memory/exports/issues/mem-$TS-recurring-<issue-type>.md" << 'EOF'
---
id: mem-YYYYMMDDHHMMSS-recurring-issue-type
title: "Recurring: <issue type>"
tags: [recurring]
category: issues
importance: medium
created: YYYY-MM-DDTHH:MM:SSZ
---

# Recurring: <issue type>

## Summary
<what to check for and how to fix>
EOF
fi
```

This is **SILENT** - no user notification. Builds knowledge for future reviews.

## NOT This Skill's Job

- Improvement suggestions → use suggest skill
- Asking permission for obvious fixes → just fix them
