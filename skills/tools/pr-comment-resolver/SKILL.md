---
name: pr-comment-resolver
description: This skill should be used when the user asks to "handle PR comments", "resolve PR review comments", "fix PR feedback", "process review comments", "address PR suggestions", or provides a GitHub PR URL with review comments to handle. Automates the workflow of reviewing, fixing, and resolving PR comments with atomic commits.
---

# PR Comment Resolver

Automate the process of handling GitHub PR review comments: evaluate each comment, fix issues with atomic commits, and reply with detailed resolution information.

## Core Principles

1. **Critical Thinking First** - Evaluate whether each comment is correct before acting; reviewers can make mistakes too
2. **Commit by Topic, Not by Comment** - Group commits by logical change, not by comment count; one commit can address multiple related comments
3. **Atomic Commits** - Each commit should be a single logical fix; different concerns require separate commits
4. **Human Collaboration** - Ask the user when uncertain about a fix, interpretation, or when you disagree with a comment
5. **Detailed Replies** - Include fix explanation, commit hash, and link in every resolution
6. **Reply to Thread** - Always reply directly to each review thread, NOT as a general PR comment at the bottom

## Quick Start

### Interactive Mode (Default)

```
User: Handle the comments on this PR: https://github.com/owner/repo/pull/123
```

Workflow:
1. Fetch all unresolved review comments
2. Present each comment for review
3. For each comment, determine whether to fix or explain why no fix is needed
4. Execute fixes with atomic commits
5. Reply and resolve each comment

### Auto Mode

```
User: Auto-resolve all comments on https://github.com/owner/repo/pull/123
```

Process all comments automatically, only pausing for truly ambiguous cases.

## Workflow Overview

### Phase 1: Fetch Comments

Use `gh api graphql` to retrieve all unresolved review comments:

```bash
gh api graphql -f query='
{
  repository(owner: "<OWNER>", name: "<REPO>") {
    pullRequest(number: <PR_NUMBER>) {
      reviewThreads(first: 100) {
        nodes {
          id
          isResolved
          path
          line
          comments(first: 10) {
            nodes {
              body
              author { login }
            }
          }
        }
      }
    }
  }
}' --jq '.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved == false)'
```

Extract key information:
- Comment ID and thread ID
- File path and line number
- Comment body (the feedback)
- Author information (login, isBot flag)

### Phase 1.5: Identify AI Comments

Determine if each comment is from an AI reviewer. **Only AI comments will be auto-resolved; human comments will only receive replies.**

#### Detection Methods

1. **Bot Account Suffix**: Check if author login ends with `[bot]`
   - Examples: `github-actions[bot]`, `dependabot[bot]`, `copilot[bot]`

2. **Known AI Services**: Match against known AI code review services and LLMs
   - **Code Review Bots**: `coderabbitai`, `codiumai`, `sourcery-ai`, `deepsource`, `sonarcloud`, `codeclimate`, `snyk`
   - **LLM Assistants**: `copilot`, `claude`, `gemini`, `codex`, `openai`, `anthropic`, `chatgpt`, `gpt`
   - **CI/Automation**: `github-actions`, `dependabot`, `renovate`
   - (Extend this list as needed)

#### Detection Logic

```bash
# AI/Bot Detection Logic (case-insensitive)
is_ai_comment=false
author_lower=$(echo "$author_login" | tr '[:upper:]' '[:lower:]')

# Combined pattern: [bot] suffix OR known AI services
ai_patterns='(\[bot\]$|coderabbitai|codiumai|sourcery-ai|deepsource|sonarcloud|codeclimate|snyk|copilot|claude|gemini|codex|openai|anthropic|chatgpt|gpt|github-actions|dependabot|renovate)'

if [[ "$author_lower" =~ $ai_patterns ]]; then
  is_ai_comment=true
fi
```

#### Classification Result

| Author Type | Auto-Resolve | Action |
|-------------|--------------|--------|
| AI/Bot | Yes | Reply and resolve after fix |
| Human | No | Reply only, let human resolve |
| Uncertain | Ask user | Prompt user: "Should I resolve this comment from @{author}?" |

> **Note:** If the author doesn't match known AI patterns but has bot-like characteristics (e.g., automated messages, service accounts), ask the user whether to auto-resolve.

### Phase 2: Evaluate Each Comment

For each unresolved comment, **critically assess whether the suggestion is correct** before determining action:

| Decision | Criteria |
|----------|----------|
| **Needs Fix** | Valid point: actual bug, code issue, style violation, missing feature |
| **No Fix Needed** | Already addressed, misunderstanding, design choice, out of scope |
| **Disagree** | Reviewer's suggestion is incorrect, would introduce bugs, violates architecture, or is technically flawed |
| **Uncertain** | Ambiguous request, multiple interpretations, needs clarification |

> **⚠️ Important:** Do not blindly accept all comments. Reviewers can make mistakes. Always verify the technical validity of each suggestion before implementing.

### Phase 3: Execute Action

#### If Fix Needed

1. Read the relevant file(s)
2. Implement the fix
3. Create an atomic commit with descriptive message
4. Push to the PR branch
5. Reply with fix details
6. **If AI comment**: Resolve the thread | **If human**: Leave unresolved

#### If No Fix Needed

1. Compose explanation of why no change is required
2. Reply with the explanation
3. **If AI comment**: Resolve the thread | **If human**: Leave unresolved

#### If Disagree

1. **Verify your assessment** - Double-check your reasoning against the codebase
2. **Present to user first** - Always discuss with the user before responding to the reviewer
3. Explain why the suggestion may be problematic:
   - Would it introduce a bug?
   - Does it violate existing architecture patterns?
   - Is it based on incorrect assumptions about the code?
4. Compose a polite, technical response with evidence
5. **Do NOT auto-resolve** - Let the reviewer respond or the user decide

#### If Uncertain

1. Present the comment to the user
2. Explain the ambiguity
3. Ask for guidance
4. Proceed based on user input

### Phase 4: Reply (and Conditionally Resolve)

After each action, reply to the comment thread. **Only auto-resolve if the comment is from an AI/bot; human comments require manual resolution.**

| Comment Source | After Reply |
|----------------|-------------|
| AI/Bot | Auto-resolve the thread |
| Human | Do NOT resolve - let the reviewer close it |

> **⚠️ CRITICAL:** You MUST use the GraphQL `addPullRequestReviewThreadReply` mutation to reply directly to each review thread. Do NOT use `gh pr comment` as it posts to the PR bottom instead of the specific thread.

**Reply format for fixes:**

```markdown
- [<short-hash> <commit-message>](<commit-url>)

**Files modified:**
- `<file-path>`

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

Example:

```markdown
- [a1b2c3f fix(auth): add null check for user session](https://github.com/owner/repo/commit/a1b2c3f)

**Files modified:**
- `src/auth/session.ts`

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

**Reply format for no-fix:**

```markdown
No changes needed.

**Reason:** <explanation of why no fix is required>

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### Phase 5: Summary Report

After processing all comments, output a summary report:

```markdown
## PR Comment Resolution Summary

**PR:** #<number> - <title>
**Processed:** <total> comments

### Commits
- [<hash> <message>](<url>)
- [<hash> <message>](<url>)

### Statistics
| Status | AI/Bot | Human | Total |
|--------|--------|-------|-------|
| Fixed & Resolved | <n> | - | <n> |
| Fixed (reply only) | - | <n> | <n> |
| No fix & Resolved | <n> | - | <n> |
| No fix (reply only) | - | <n> | <n> |
| Disagreed (pending) | <n> | <n> | <n> |
| Skipped | <n> | <n> | <n> |

### Details
| Comment | Author | Type | File | Action | Resolved |
|---------|--------|------|------|--------|----------|
| <summary> | @bot | 🤖 AI | `<path>` | Fixed [<hash>](<url>) | ✅ |
| <summary> | @human | 👤 Human | `<path>` | Fixed [<hash>](<url>) | ⏳ Pending |
| <summary> | @bot | 🤖 AI | `<path>` | No fix | ✅ |
| <summary> | @human | 👤 Human | `<path>` | Disagreed | ⏳ Pending |
```

## GitHub CLI Commands

### Fetch PR Comments

```bash
# Get all review threads (requires GraphQL - gh pr view does not support reviewThreads)
gh api graphql -f query='
{
  repository(owner: "<OWNER>", name: "<REPO>") {
    pullRequest(number: <NUMBER>) {
      reviewThreads(first: 100) {
        nodes {
          id
          isResolved
          path
          line
          comments(first: 10) {
            nodes { body author { login } }
          }
        }
      }
    }
  }
}'

# Get unresolved threads only (add jq filter)
# ... --jq '.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved == false)'
```

### Reply to Comment

```bash
gh api graphql -f query='
  mutation($body: String!, $threadId: ID!) {
    addPullRequestReviewThreadReply(input: {
      pullRequestReviewThreadId: $threadId,
      body: $body
    }) {
      comment { id }
    }
  }
' -f threadId="<THREAD_ID>" -f body="<REPLY_BODY>"
```

### Resolve Thread

```bash
gh api graphql -f query='
  mutation {
    resolveReviewThread(input: {
      threadId: "<THREAD_ID>"
    }) {
      thread { isResolved }
    }
  }
'
```

## Commit Message Format

Follow conventional commit style. **Describe the change, not the comment:**

```
<type>(<scope>): <what was changed>

<why this change was needed - optional>
```

> **Important:** Commit messages should describe the modification topic, NOT "address comment" or "per reviewer request". The commit should make sense even without PR context.

Example - Good:

```
fix(auth): add null check for user session

The session object may be undefined when the user
is not logged in. Added defensive check to prevent
TypeError.
```

Example - Bad:

```
fix: address PR review comments

Addresses PR review comment by @reviewer
```

## Commit Grouping Strategy

> **Key principle:** Group by **modification topic**, not by comment count.

### When to use ONE commit for multiple comments

Use one commit when comments point to the **same logical change**:

```
Comment A: "Add null check for session"
Comment B: "Handle undefined session gracefully"
Comment C: "Session might be null here"

All three → same topic (session null safety) → ONE commit
→ Reply to all three comments with the same commit link
```

### When to use SEPARATE commits

Use separate commits when comments are **different concerns**:

```
Comment A: "Add error handling"
Comment B: "Improve performance here"
Comment C: "Add input validation"

Three different topics → THREE separate commits
→ Each comment gets its own commit link
```

### Decision guide

| Scenario | Commits | Why |
|----------|---------|-----|
| Same topic, different locations | 1 | Same logical change |
| Same function, different concerns | N | Different modifications |
| Same line, same fix | 1 | Literally one change |
| Related but independent | N | Can be reverted separately |

## Decision Tree

```
Comment Received
      │
      ▼
┌─────────────────┐
│ Is author       │──Yes──▶ is_ai_comment = true
│ AI/Bot?         │
└────────┬────────┘
         │No
         ▼
┌─────────────────┐
│ Uncertain?      │──Yes──▶ Ask user: "Resolve comment from @{author}?"
│ (bot-like but   │         └──▶ User decides
│ not in list)    │
└────────┬────────┘
         │No
         ▼
   is_ai_comment = false
         │
         ▼
┌─────────────────┐
│ Is the comment  │──No──▶ Ask user for clarification
│ clear?          │
└────────┬────────┘
         │Yes
         ▼
┌─────────────────┐
│ Is the comment  │──No──▶ Discuss with user first
│ technically     │        └──▶ Politely disagree with evidence
│ correct?        │             (Do NOT auto-resolve)
└────────┬────────┘
         │Yes
         ▼
┌─────────────────┐
│ Is a code       │──No──▶ Reply with explanation
│ change needed?  │        └──▶ If AI: resolve | If human: skip resolve
└────────┬────────┘
         │Yes
         ▼
   Fix → Commit → Push → Reply
         │
         ▼
┌─────────────────┐
│ is_ai_comment?  │──Yes──▶ Resolve thread
└────────┬────────┘
         │No
         ▼
   Leave unresolved (human reviewer will close)
```

## Important Guidelines

### DO

- **Critically evaluate comments:** Verify the technical validity of each suggestion against the codebase before acting. Reviewers can be wrong.
- **Check comment author type:** Determine if the comment is from AI/bot before deciding whether to auto-resolve.
- **Only auto-resolve AI comments:** After replying to an AI/bot comment, resolve the thread. Human comments should only receive replies.
- **Commit by topic:** Create atomic commits for each logical change. Group related fixes into one commit, never bundle unrelated changes. Reply to all related comments with the same commit link.
- **Write descriptive commit messages:** Describe the *what* and *why* of the change using conventional commit format. Avoid messages like "address PR comments".
- **Collaborate with the user:** Ask for clarification on ambiguous comments. Always discuss with the user before pushing back on a reviewer.
- **Provide detailed replies:** Include commit links for fixes. When disagreeing, use polite, technical reasoning with evidence.
- **Maintain code quality:** Verify fixes compile and pass linting before committing.

### DON'T

- **Blindly accept all comments** - always verify correctness first
- **Auto-resolve human reviewer comments** - only AI/bot comments should be auto-resolved; let humans close their own threads
- **Bundle different concerns** into one commit - separate topics need separate commits
- Write commit messages like "address PR comments" or "per reviewer request"
- Implement changes that would introduce bugs or violate architecture
- Auto-resolve disagreements without user confirmation
- Resolve without replying
- Make assumptions about ambiguous requests
- Force push or rewrite history
- Skip verification steps
- **Use `gh pr comment` for replies** - This posts to PR bottom, not to the review thread. Always use GraphQL `addPullRequestReviewThreadReply` for direct replies; use `gh pr comment` only as a fallback.

## Error Handling

| Error | Action |
|-------|--------|
| Comment already resolved | Skip and continue |
| File not found | Ask user for correct path |
| Commit fails | Report error, do not resolve |
| Push fails | Report error, suggest manual intervention |
| GraphQL API error | See the "Fallback Behavior" section |

## Additional Resources

### Reference Files

For detailed workflows and templates:

- **`references/workflow.md`** - Step-by-step workflow with examples
- **`references/reply-templates.md`** - Copy-paste reply templates for common scenarios

## Fallback Behavior

If the GraphQL API fails to reply to a thread (e.g., network error, permission issue, thread already resolved):

1. **Retry once** after a brief delay
2. **If retry fails**, fall back to `gh pr comment` with clear context:

```bash
gh pr comment <PR_NUMBER> --body "$(cat <<'EOF'
**Re: Review comment on `<FILE_PATH>:<LINE>`**

> <ORIGINAL_COMMENT_EXCERPT>

<YOUR_REPLY_CONTENT>

---
*Note: Unable to reply directly to the review thread. This is a fallback comment.*
EOF
)"
```

3. **Report to user** that the reply was posted as a general comment instead of a thread reply
4. **Continue processing** remaining comments

> **Important:** The fallback should only be used when GraphQL truly fails. Always attempt GraphQL first.

## Notes

- Requires `gh` CLI authenticated with appropriate permissions
- Works with GitHub PRs (GitLab/Bitbucket not supported)
- Branch must be checked out locally
- Respect repository's commit message conventions if defined
