---
name: getting-pr-review-comments
description: Use when needing Claude bot review recommendations from PR - automatically waits for workflows (including Claude review) to complete, then extracts recommendations and returns concise summary without loading full review into context
---

# Getting PR Review Comments

## Overview

Retrieve Claude Code review bot comments from a PR and extract actionable recommendations.

**Core principle:** Run in subagent to save context. Return only recommendations, not full comment.

**Automatic dependency:** This skill automatically waits for workflows (including Claude review) to complete before fetching comments.

## CRITICAL: Scope Guardrails

**This skill ONLY extracts and reports review recommendations. It NEVER fixes issues.**

When you find review recommendations:
- ✅ Extract the recommendations from the Claude bot comment
- ✅ Format them as a numbered list with severity and location
- ✅ Return the concise summary
- ❌ DO NOT investigate the recommendations
- ❌ DO NOT read the code files mentioned
- ❌ DO NOT propose fixes
- ❌ DO NOT make any code changes

**Why:** This skill runs in a subagent to save tokens. Addressing recommendations should happen in main
  context with user approval, not automatically in the subagent. Return the recommendations and let the
  caller decide what to do.

## When to Use

Use when you need:
- Claude bot's review recommendations for a PR
- Code review feedback after workflows complete
- Action items from automated review

**When NOT to use:**
- Need human reviewer comments (different workflow)

## Workflow

### Step 0: Wait for Workflows to Complete (Automatic)

Before fetching review comments, ensure all workflows are complete (Claude review is a workflow job).

**Skill location:** `~/.claude/skills/awaiting-pr-workflows/SKILL.md`

Read and execute the `awaiting-pr-workflows` skill's workflow to:
- Check for unpushed commits
- Verify PR exists and commit correlation
- Wait for workflows to start (up to 30s)
- Wait for workflows to complete (up to 20 minutes)

Once all workflows (including Claude Code Review) are complete, proceed to Step 1 below.

### Step 1: Get PR Issue Comments (Not Review Comments)

**Important:** Claude bot posts general PR comments, not inline code review comments.
Use the `comments` field from `gh pr view`, not `gh api .../pulls/.../comments`.

```bash
# Get PR comments (issue-level comments, not inline review comments)
gh pr view $PR_NUM --json comments \
  --jq '.comments[] | {author: .author.login, body: .body, createdAt: .createdAt}'
```

### Step 2: Filter for Most Recent Claude Bot Comment

**Critical:** Only use the MOST RECENT comment from Claude bot.
The bot may post multiple iterations during review - we only want the latest.

```bash
# Get only the most recent Claude bot comment
gh pr view $PR_NUM --json comments \
  --jq '.comments
    | map(select(.author.login == "claude"))
    | sort_by(.createdAt)
    | reverse
    | .[0]'
```

**Common bot usernames to check:**
- `claude` (most common in this repo)
- `github-actions[bot]`
- `claude-code-bot`
- `anthropic-claude[bot]`

### Step 3: Extract Recommendations

Parse the comment body for:
- **Recommendation blocks** - Usually markdown sections like "## Recommendations" or "### Suggestions"
- **File references** - Format: `file.ts:123` or `file.ts#L123`
- **Severity markers** - "⚠️", "❌", "💡", or labels like "[HIGH]", "[MEDIUM]"

## Return Format

**Return to main context:**

```markdown
Claude Code Review Recommendations for PR #{number}:

1. **[Severity]** File:Line - Brief description
   - Detailed recommendation
   - Link to comment

2. **[Severity]** File:Line - Brief description
   - Detailed recommendation
   - Link to comment

[Total: N recommendations]
[Review comment URL: https://...]
```

## Example Implementation

```bash
#!/bin/bash
# Run this in subagent

PR_NUM=$1

# Get only the most recent Claude bot comment
LATEST_COMMENT=$(gh pr view $PR_NUM --json comments \
  --jq '.comments
    | map(select(.author.login == "claude"))
    | sort_by(.createdAt)
    | reverse
    | .[0]')

# Extract the body from the most recent comment
COMMENT_BODY=$(echo "$LATEST_COMMENT" | jq -r '.body')

# Parse and extract recommendations from the body
# Look for sections like "## 🔴 Critical Issues", "## ⚠️ Warnings", etc.
# Format as numbered list with severity, location, and description
# (Actual parsing depends on Claude bot's comment format)

# Return concise summary (limit to key recommendations)
```

## Common Mistakes

**Returning all Claude bot comments instead of just the latest**
- **Problem:** Bot may post multiple review iterations; returning all creates confusion
- **Fix:** Sort by `createdAt`, reverse, take `.[0]` to get only most recent

**Loading full comment into main context**
- **Problem:** Claude reviews can be 10k+ tokens
- **Fix:** Extract just the recommendations in subagent

**Not waiting for review to complete**
- **Problem:** Review job may still be running
- **Fix:** Use `awaiting-pr-workflows` first

**Returning raw JSON/markdown**
- **Problem:** Hard to parse in main context
- **Fix:** Format as numbered list with clear structure

**Using inline review comments API instead of PR comments**
- **Problem:** `gh api repos/.../pulls/.../comments` returns inline code comments, not bot summaries
- **Fix:** Use `gh pr view --json comments` for issue-level comments

## Quick Reference

| Task | Command |
|------|---------|
| List all PR comments | `gh pr view $PR --json comments` |
| Get most recent Claude comment | `gh pr view $PR --json comments --jq '.comments \| map(select(.author.login == "claude")) \| sort_by(.createdAt) \| reverse \| .[0]'` |
| Extract comment body | `jq -r '.body'` |
| Parse recommendations | Look for "## 🔴 Critical Issues", "## ⚠️ Warnings", "## 💡 Suggestions" sections |

## Use Subagents

**CRITICAL:** Always run in subagent.

```
Use Task tool with subagent_type='general-purpose'.
Give them this skill and the PR number.
They return just the recommendations.
```

**Why:** Saves 100k+ tokens by not loading full review into main context.
