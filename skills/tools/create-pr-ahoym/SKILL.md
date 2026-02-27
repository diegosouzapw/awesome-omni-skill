---
name: create-pr
description: "Create a pull request or update an existing one following project conventions."
---

## Context
- Current branch: !`git branch --show-current 2>/dev/null`
- Commits on branch: !`git log origin/main..HEAD --oneline 2>/dev/null | head -20`

# Create Pull Request

Create a pull request or update an existing one following project conventions.

## Usage

- `/git:create-pr` - Create PR to main
- `/git:create-pr <base-branch>` - Create PR to specified base branch

## Reference Files (conditional — read when creating PR)

- pr-body-template.md - PR body structure template

## Reference Files (conditional — read only when needed)

- @~/.claude/skill-references/platform-detection.md - Platform detection for GitHub/GitLab

## Pre-PR Checklist

Before creating the PR, verify these items are complete:

- [ ] All tests pass locally
- [ ] Code formatted with linter
- [ ] No sensitive data in code or logs
- [ ] Metrics added for new operations (if applicable)
- [ ] Error handling implemented
- [ ] Documentation updated (if needed)

## Instructions

0. **Detect platform** — follow `@~/.claude/skill-references/platform-detection.md` to determine GitHub vs GitLab. Set `CLI`, `REVIEW_UNIT`, and API command patterns accordingly. All commands below use GitHub (`gh`) syntax; substitute GitLab equivalents if on GitLab.

1. **Gather context** (run in parallel):
   - `git status` - Check for uncommitted changes
   - `git branch --show-current` - Get current branch name
   - `git log origin/main..HEAD --oneline` - See commits to include (adjust base as needed)
   - `git diff origin/main..HEAD --stat` - See files changed

2. **Check for uncommitted changes**:
   - If there are uncommitted changes, ask user if they want to commit first
   - Do not proceed with PR creation if there are uncommitted changes

3. **Run formatter on changed files**:
   - Detect project formatter: check for `.prettierrc*`, `biome.json`, or formatter config in `package.json`
   - Run the formatter in check mode on changed files: e.g., `npx prettier --check $(git diff --name-only origin/main -- '*.ts' '*.tsx' '*.js' '*.json' '*.css')`
   - If formatting issues found, fix them (`npx prettier --write <files>`), stage, and commit before proceeding
   - If no formatter config found, skip this step

4. **Check for existing PR**:
   ```bash
   gh pr list --head <current-branch>
   ```
   - If a PR already exists, ask user: "PR #N already exists for this branch. Update its description instead of creating new?"
   - If yes, use `gh pr edit <number> --body "..."` instead of `gh pr create`

5. **Infer Jira ticket** (if applicable):
   - Extract Jira ticket ID from the branch name (common patterns: `feature/PROJ-123`, `bugfix/PROJ-456`, `PROJ-789-description`)
   - Look for Jira ticket references in commit messages
   - If no ticket can be inferred, leave the URL incomplete for the user to fill in

6. **Determine base branch**:
   - If `$ARGUMENTS` provided, use that as base
   - If branch name suggests a parent (e.g., `feature/foo-part2` might be based on `feature/foo`), ask user
   - Default to `main`

7. **Check if push needed**:
   - If local is ahead of remote, push first: `git push -u origin <branch>`

8. **Create or update PR** using the template from pr-body-template.md

9. **Run the gh command**:

For new PR:
```bash
gh pr create --base <base-branch> --title "<title>" --body "$(cat <<'EOF'
<body content>
EOF
)"
```

For existing PR:
```bash
gh pr edit <number> --body "$(cat <<'EOF'
<body content>
EOF
)"
```

10. **Return the PR URL** to the user.
