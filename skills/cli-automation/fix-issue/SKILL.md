---
name: fix-issue
description: 'Workflow for fixing GitHub issues from start to finish'
disable-model-invocation: true
allowed-tools: 'Read, Write, Edit, Glob, Grep, Bash'
---

# Fix Issue Workflow

Invoked with: `/fix-issue <issue-number>`

## Step 1: Fetch Issue Details

```bash
gh issue view $ARGUMENTS
```

Extract: title, description, labels, reproduction steps, related PRs.

## Step 2: Analyze Codebase

Based on issue type:

**For bugs:**

- Search error messages with Grep
- Find related files with Glob
- Read implementation and tests
- Identify root cause before writing any code

**For features:**

- Find similar existing features and patterns
- Identify files to modify
- Check test coverage of related code

**For documentation:**

- Find relevant docs files
- Check for outdated examples

## Step 3: Create Implementation Plan

- [ ] Files to modify (with reasons)
- [ ] Files to create (if any)
- [ ] Tests to add/update
- [ ] Documentation to update

## Step 4: Create Feature Branch

```bash
git checkout -b fix/<issue-number>-<short-description>
```

## Step 5: Implement Fix

Follow project patterns:

- Svelte 5 runes only (no `$:`)
- `$effect` cleanup if managing resources
- TypeScript types for all parameters and return values
- Follow existing code style

## Step 6: Write/Update Tests

- Unit tests for new/changed functions
- Component tests for UI changes
- Update existing tests if behavior changed

```bash
npm run test
```

Fix any failing tests before proceeding.

## Step 7: Full Validation

```bash
npm run check && npm run lint && npm run test
```

ALL THREE must pass. If `build-ci` validation needed:

```bash
npm run build-ci
```

## Step 8: Commit Changes

```bash
git add <specific-files>
git commit -m "$(cat <<'EOF'
<type>(<scope>): <description>

Fixes #<issue-number>

<detailed explanation if needed>

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

**Types**: fix, feat, docs, refactor, test, chore
**Scopes**: component, map, routing, test, deps, ci

## Step 9: Create Pull Request

```bash
git push -u origin fix/<issue-number>-<short-description>

gh pr create --title "<type>: <description> (fixes #<issue>)" --body "$(cat <<'EOF'
## Summary
- Fixes #<issue-number>
- <brief description of changes>

## Changes
- <file1>: <what changed>
- <file2>: <what changed>

## Test Plan
- [ ] Unit tests added/updated and passing
- [ ] `npm run check` passes
- [ ] `npm run lint` passes
- [ ] Manual testing: <steps>

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

## Quality Checklist

Before marking complete:

- [ ] Issue fully resolved (not partial fix)
- [ ] Tests added/updated and passing
- [ ] TypeScript check passes (`npm run check`)
- [ ] Lint passes (`npm run lint`)
- [ ] Commit message references issue number
- [ ] PR created and linked to issue

## Project-Specific Notes

### Common Issue Areas

1. **Map rendering** → check `src/lib/ol/` directory
2. **Logbook entries** → check `static/data/logbook_geo.json`
3. **Routing** → check `src/routes/` and prerender config
4. **Svelte 5 patterns** → check for `$:` syntax violations

### Testing Requirements

- Vitest for all logic
- Testing Library for components
- Coverage: 70% lines, 70% functions, 70% statements, 50% branches

### Static Site Constraint

This is deployed as a static site on Netlify. No server-side fixes possible. All data must be available at build time.
