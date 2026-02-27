---
name: commit
description: Reviews working copy changes and drafts a concise 1-line commit message. Use when the user wants a commit message or asks to describe/commit changes.
---

# Commit Skill

Generate a 1-line commit message for current changes using a fast model.

## Steps

1. Delegate to a task subagent with a fast model:

```json
{
  "description": "Review working copy changes and draft a 1-line commit message",
  "model": "claude-sonnet-4-5",
  "prompt": "1. Run `jj diff --git --no-pager` to see working copy changes.\n2. If there are no changes, say 'No changes in working copy.' and stop.\n3. Draft a 1-line commit message: short phrase, brevity over grammar, explicitly describes changes, NO prefixes (feat:, fix:, etc.), NO verbose descriptions. Examples: 'task tool for parallel subagents', 'fix token expiry edge case', 'refactor auth middleware'.\n4. Return ONLY the commit message, nothing else."
}
```

2. Present the commit message to the user. Do NOT run any jj/git commands.

## Rules

- **NEVER** run `jj commit`, `jj describe`, `jj new`, or any jj mutation commands
- **NEVER** run `git add`, `git commit`, `git push`
- **NEVER** create branches or bookmarks
- Only return the message — the user will commit/describe themselves
