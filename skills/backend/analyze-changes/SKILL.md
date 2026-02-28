---
name: analyze-changes
description: Interactive chunk-by-chunk review of PR or branch changes. Identifies logical groupings, explains each chunk, and critiques for issues. Use when user says "analyze-changes", "review changes", "review this PR", "what changed", or "walk me through these changes".
compatibility: Requires git and optionally gh CLI for PR context.
---

# Analyze Changes

Perform an interactive, chunk-by-chunk review of changes in a PR or branch. Explain what each logical change does, then critique for potential issues.

## When to Use

- Reviewing a PR before merge
- Understanding what changed on a feature branch
- Self-review before pushing
- Onboarding to someone else's changes

## Process

### 1. Detect Context

Determine the diff source:

**If on a PR branch or user mentions PR:**
```bash
gh pr view --json number,title,body,baseRefName 2>/dev/null
gh pr diff
```

**If on a feature branch (no PR):**
```bash
# Detect main branch
git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@'
# Or default to main/master

git diff main...HEAD
# or
git diff master...HEAD
```

**If user specifies:**
- Follow their instruction (specific commits, files, etc.)

### 2. Identify Logical Chunks

Analyze the diff and group changes into logical chunks. A chunk is:
- A coherent unit of change (single purpose)
- May span multiple files
- Should be reviewable independently

**Good chunking:**
- "Add user validation middleware"
- "Refactor database connection pooling"
- "Update API response format"
- "Add tests for auth flow"

**Bad chunking:**
- By file (misses logical grouping)
- Too granular (every function)
- Too broad ("all the changes")

### 3. Present Overview

```markdown
## Change Overview

**Context**: {PR #N: title | Branch: name vs main}

**Overall Goal**: {1-2 sentences: what does this PR/branch aim to achieve}

## Logical Chunks

1. **{Chunk 1 theme}**: {one-line description}
2. **{Chunk 2 theme}**: {one-line description}
3. **{Chunk 3 theme}**: {one-line description}

---

Ready to review Chunk 1: "{theme}"? (y/skip/all)
```

### 4. Review Each Chunk

Wait for user confirmation before each chunk.

**User responses:**
- `y` / `yes` / `continue` / `proceed` → Review this chunk
- `skip` → Skip to next chunk
- `all` → Review all remaining chunks without pausing

For each chunk, follow [CHUNK-REVIEW-GUIDE.md](CHUNK-REVIEW-GUIDE.md):

```markdown
### Chunk {N}: {Theme}

**Files touched**:
- `{path/to/file1}` — {brief: added/modified/deleted, what}
- `{path/to/file2}` — {brief}

**What this changes**:
{Explanation of what the code does now, how it differs from before}

**Why** (if discernible):
{Rationale for the change — from PR description, commit messages, or code context}

**Concerns**:
{Any issues, risks, or suggestions — or "None identified" if solid}

---

Ready for Chunk {N+1}: "{theme}"? (y/skip/all)
```

### 5. Final Summary

After all chunks reviewed:

```markdown
## Review Summary

**Chunks reviewed**: {N} of {total}

**Cross-cutting concerns**:
{Issues that span multiple chunks or affect the PR as a whole}

**Overall assessment**:
{Brief: is this PR ready? What needs attention?}
```

## Guidelines

- **Explain before critiquing**: User should understand the change before hearing concerns
- **Be specific in concerns**: "This might break X because Y" not "This seems risky"
- **Acknowledge good work**: If a chunk is well-done, say so briefly
- **Don't invent issues**: If code is solid, "None identified" is fine
- **Respect user's time**: Keep explanations concise, expand if asked
