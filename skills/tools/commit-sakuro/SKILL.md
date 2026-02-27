---
name: commit
description: Git Commit Command
---

# Commit

Create a git commit following project conventions.

## Workflow

1. Run project tests before committing (skip for documentation-only changes)
2. Stage specific files individually — never use `git add .` or `git add -A`
3. Review staged changes with `git diff --cached`
4. Commit using heredoc with single-quoted delimiter (`<<'EOF'`)

## Commit Message Rules

- Write in English, imperative mood ("Fix" not "Fixed")
- Start with `:emoji:` code (not raw Unicode), followed by a space
- No period at end of subject line
- Body: focus on WHAT and WHY, 2-3 bullet points max
- Omit changes that were written but not included in the final commit
- Never include AI attribution

## Emoji Codes

`:zap:` Performance · `:bug:` Bug fix · `:sparkles:` Feature · `:memo:` Documentation
`:recycle:` Refactor · `:art:` Structure/format · `:wrench:` Configuration
`:white_check_mark:` Tests · `:fire:` Remove · `:package:` Dependencies
