---
name: push
description: Add all changes, commit, and push to GitHub
user-invocable: true
disable-model-invocation: true
argument-hint: [commit message]
allowed-tools: Bash(git:*), Bash(gh:*)
---

# Add, Commit, and Push

Stage all changes, create a commit, and push to GitHub.

## Steps

1. Run `git status` to review what will be committed. Never use the `-uall` flag.
2. Run `git diff` to see staged and unstaged changes.
3. Run `git log --oneline -5` to see recent commit message style.
4. **Show the user the list of files that will be added/committed and ask for confirmation before proceeding.** If the user declines, stop.
5. If `$ARGUMENTS` is provided, use it as the commit message. Otherwise, analyze the changes and generate a concise commit message that describes **why** the changes were made.
6. Stage all changes:
   ```bash
   git add -A
   ```
7. Create the commit. Always end the message with the co-author line:
   ```
   Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
   ```
8. Push to the current branch:
   ```bash
   git push origin $(git rev-parse --abbrev-ref HEAD)
   ```
9. If the push fails because the remote branch doesn't exist yet, push with `-u`:
   ```bash
   git push -u origin $(git rev-parse --abbrev-ref HEAD)
   ```
10. Report the result — show the commit hash and confirm the push succeeded.

## Rules

- Do NOT use `--force` or `--no-verify` flags.
- Do NOT commit files that look like secrets (`.env`, credentials, tokens). Warn the user if any are staged.
- If there are no changes to commit, tell the user and stop.
- Always use a HEREDOC to pass the commit message to avoid quoting issues.
