---
name: release
description: Commit, push, tag, and release timesheetz
disable-model-invocation: true
---

# Release Skill

Create a new release for timesheetz. Usage: `/release`

## Steps

### Phase 1: Analyze Changes

1. **Check status**: Run `git status` and `git diff --stat` to see uncommitted changes
2. **Abort if clean**: If no changes, inform user and stop
3. **Analyze changes**: Review the diff to categorize changes:
   - **patch**: Bug fixes, typo corrections, small tweaks, no new functionality
   - **minor**: New features, enhancements, new files, UI improvements
   - **major**: Breaking changes, API changes, major rewrites

### Phase 2: Get User Approval

4. **Present summary**: Show the user:
   - List of changed files
   - Summary of what changed
   - Suggested version bump type with reasoning
   - Proposed commit message
5. **Wait for approval**: Ask user to confirm or adjust the version bump type

### Phase 3: Execute Release (only after approval)

6. **Create commit**:
   - Use the approved commit message
   - Always end with: `Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>`
   - Use HEREDOC format for the commit message
7. **Handle remote changes**: Run `git pull --rebase origin main` before pushing
8. **Push**: Run `git push origin main`
9. **Determine version**:
   - Get latest tag: `git tag --sort=-v:refname | head -1`
   - Apply approved bump type:
     - `patch`: v1.29.0 → v1.29.1
     - `minor`: v1.29.0 → v1.30.0
     - `major`: v1.29.0 → v2.0.0
10. **Create tag**: `git tag <new-version>`
11. **Push tag**: `git push origin <new-version>`
12. **Create release**: Check if GitHub Action created it, otherwise create with `gh release create`
13. **Report**: Show the release URL to the user

## Version Bump Guidelines

| Change Type | Bump | Examples |
|-------------|------|----------|
| Bug fix | patch | Fix typo, correct calculation, fix crash |
| New feature | minor | Add new tab, new keybinding, new config option |
| Enhancement | minor | Improve UI layout, better error messages |
| New files | minor | Add skill, add documentation |
| Breaking change | major | Remove feature, change API, incompatible config |

## Commit Message Format

```
<Summary of changes in imperative mood>

- Bullet point details if multiple changes

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
```

## Safety

- Never force push
- Never skip hooks
- Always pull --rebase before pushing
- Always get user approval before executing
- Wait for user confirmation if there are merge conflicts
