---
name: commit
description: Create atomic git commits following the three-pass methodology.
---

# commit

## Trigger

`/commit`

## Process

Read `./atomic-git-commits.md` in this skill's directory for the full reference. Then follow these steps.

### 1. Content pass

- Run `pre-commit run --all-files` to fix formatting first
- Check `git status` and `git diff --staged` to understand what's staged
- Identify how many distinct logical/atomic changes exist
- Summarize them: one sentence per proposed commit

### 2. Get user confirmation

Ask the user which change(s) to proceed with. Even if there is only ONE logical change, get explicit confirmation before proceeding.

### 3. Standards check

For each commit:

- Standard prefix used (feat:, fix:, docs:, refactor:, style:, test:, build:, ci:, perf:)
- Subject capitalized after colon
- Imperative mood ("Add" not "Added")
- Under 50 characters for subject
- Body explains WHY not just what
- 72 character wrap for body lines

### 4. Final review

- Re-read the commit message. Does it tell a complete story?
- Check the diff one more time. Are all changes intentional?
- Can this be reverted independently?

### 5. Commit

Use `git commit -e -m "your message"` so the message is pre-filled but the user can edit it in their editor before finalizing.

NEVER add a co-author credit.

### 6. Output

Display the full commit message:

```
git log -1 --format="%h %s%n%n%b"
```
