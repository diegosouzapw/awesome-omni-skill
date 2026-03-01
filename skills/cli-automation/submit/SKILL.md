---
name: submit
description: Commit all changes and push to the current branch
disable-model-invocation: true
---

Commit all outstanding changes and push to the current branch.

1. Run `git status` (never use `-uall`) and `git diff` to see all changes, and `git log --oneline -5` to see recent commit style.
2. List all files that will be staged (modified, added, deleted, untracked) and the proposed commit message. Ask the user for confirmation using AskUserQuestion before proceeding. Do not stage files that contain secrets.
3. Once confirmed, stage the approved files explicitly by name (prefer naming files explicitly over `git add -A`).
4. Write the commit message as confirmed. End with `Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>`.
5. Push to the current branch with `git push`.
