---
name: update-docs-and-commit
description: Updates documentation files (changelog, architecture, project_status) based on git changes, then stages and commits all changes. Use after completing features or fixes.
---

# Update Docs and Commit

Automatically analyzes git changes and updates project documentation to reflect new features, fixes, and architectural changes.

## What This Skill Does

1. **Analyze git changes** - Examine staged/unstaged changes via `git status` and `git diff`
2. **Update changelog** - Add entries for new features and bug fixes to `docs/changelog.md`
3. **Update architecture** - Only modify `docs/architecture.md` if structural changes occurred
4. **Update project status** - Move completed items in `docs/project_status.md`
5. **Create commit** - Stage and commit all documentation changes

## When to Use

Use this skill when:
- You've completed a feature or bug fix
- You want to keep documentation synchronized with code changes
- You need to create a clean commit with updated docs

Invoke by saying:
- "Update docs and commit"
- "/update-docs-and-commit"
- "Document what we just built and commit"

## Execution Steps

### Step 1: Analyze Git Changes

Run these commands to understand what changed:

```bash
git status --short
git diff HEAD --stat
git log -1 --oneline
```

### Step 2: Categorize Changes

Categorize based on:
- **Feature**: New functionality (new files, new routes, new components)
- **Fix**: Bug fixes (modifications to existing files)
- **Refactor**: Code organization without behavior change
- **Architecture**: Structural changes (new directories, schema changes, new integrations)
- **Chore**: Dependencies, configs, tooling

### Step 3: Update docs/changelog.md

Read the current changelog and add entries under `## [Unreleased]`:

```markdown
### Added
- Feature name - brief description

### Fixed
- Bug name - what was fixed

### Changed
- What changed and why
```

Be concise. One line per item.

### Step 4: Update docs/architecture.md (Conditional)

**Only update if:**
- New directories were created
- Database schema changed
- New components/routes added to architecture
- API routes added or changed

**Skip update if:**
- Only bug fixes
- Only styling changes
- Only logic changes within existing structure

If updating, modify the relevant section (component tree, API routes table, etc.)

### Step 5: Update docs/project_status.md

Read current status and:
- Move completed items from "Pending" to "Done"
- Update progress percentages
- Update milestone status if appropriate

### Step 6: Create Commit

Stage and commit all changes:

```bash
git add -A
git commit -m "$(cat <<'EOF'
docs: update documentation for recent changes

- Updated changelog with new features/fixes
- Updated project status with completed items
[- Updated architecture docs (if applicable)]

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
```

Show the user what was committed with `git show --stat HEAD`.

## Conservative By Design

This skill is intentionally conservative:
- Only updates docs that genuinely need updating
- Doesn't over-document small changes
- Leaves architecture.md alone for non-structural changes
- Creates focused, readable changelog entries

## Example Output

After running, you should see:

```
✓ Analyzed 5 changed files
✓ Updated docs/changelog.md (added 2 features, 1 fix)
✓ Updated docs/project_status.md (marked 3 tasks complete)
✓ docs/architecture.md unchanged (no structural changes)
✓ Created commit: docs: update documentation for recent changes
```
