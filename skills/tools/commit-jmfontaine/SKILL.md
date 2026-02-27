---
name: commit
description: Git commit guidelines. Use when creating, amending, squashing, or rewording git commits, staging files, or writing commit messages.
allowed-tools: Bash(git add:*), Bash(git commit:*), Bash(git diff:*), Bash(git log:*), Bash(git status:*)
---

# Git Commit Guidelines

Follow Conventional Commits with these overrides:

- Allowed types: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `ci`
- Message format: `<type>: <lowercase imperative description>`
- No scopes — do not use `<type>(scope):` form
- Add body, separated by blank line, only when subject line insufficient

## Pre-Commit Review

Before committing, review all staged and unstaged changes to determine if they should be split into multiple commits. Changes belong in separate commits when they have different types (e.g., `feat` + `fix`), affect unrelated areas, or serve distinct purposes.

If the user has not explicitly asked to split, suggest doing so and list the proposed commits. Proceed with a single commit only if all changes are logically cohesive.

Also check for changes made outside the current session (e.g., editor saves, other tools). If they are relevant to the commit, offer to include them. If they are unrelated, silently ignore them unless the user asks to include them.

## New Commit vs. Amend

When changes closely follow a previous commit (e.g., a quick fix or forgotten file), evaluate whether amending the previous commit is more appropriate than creating a new one. Amending is preferable when the change corrects or completes the previous commit and that commit has not been pushed.

Never amend without the user's explicit approval. Present the two options (new commit vs. amend) and let the user decide.

## Additional Guidelines

- Always sign commits with `git commit -S`
- Do NOT include AI co-authoring information
- Do NOT use `git -C <path>` when the current directory is already the repository root
