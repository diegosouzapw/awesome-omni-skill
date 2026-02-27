---
name: Conventional Commit Generator
description: This skill should be used when the user asks to "create a conventional commit", "generate conventional commits", "commit with conventional format", "group my changes for commits", "make a conventional commit message", or mentions "semantic commits", "commitizen", "commit conventions". Analyzes staged and unstaged changes, groups related modifications, and generates properly formatted conventional commit messages with interactive commit grouping options.
version: 0.1.0
---

# Conventional Commit Generator

Generate properly formatted conventional commits by analyzing changes, grouping related modifications, and creating semantic commit messages following the Conventional Commits specification.

## Purpose

Automate the process of reviewing code changes, identifying logical groupings, and generating conventional commit messages. Ensure commits follow the Conventional Commits specification while providing control over commit granularity (separate small commits, combined medium commits, or a single large commit).

## When to Use

Use this skill when:
- Creating commits that follow Conventional Commits specification
- Need to group related changes logically before committing
- Want to understand what changes are being committed
- Need guidance on proper commit message formatting
- Working on projects requiring semantic versioning or automated changelogs

## Conventional Commit Format

Follow the standard format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Type** (required):
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, missing semicolons, etc.)
- `refactor`: Code refactoring without changing functionality
- `perf`: Performance improvements
- `test`: Adding or updating tests
- `build`: Build system or dependency changes
- `ci`: CI/CD configuration changes
- `chore`: Maintenance tasks, tooling changes
- `revert`: Reverting previous commits

**Scope** (optional): Component or module affected (e.g., `auth`, `api`, `ui`, `parser`)

**Subject** (required): Brief imperative description (≤50 chars)

**Body** (optional): Detailed explanation of what and why

**Footer** (optional): Breaking changes, issue references

## Workflow

### Step 1: Analyze Changes

Run git commands in parallel to gather change information:

```bash
# View status (never use -uall flag)
git status

# View unstaged changes
git diff

# View staged changes
git diff --cached

# Recent commit messages for style reference
git log --oneline -10
```

Review the output to understand:
- What files have changed
- What modifications were made
- Current staging status
- Existing commit message patterns in the repository

### Step 2: Group Related Changes

Analyze the changes and identify logical groupings based on:

**Grouping criteria:**
- **Purpose**: Changes serving the same goal (e.g., all authentication-related)
- **Type**: Same conventional commit type (e.g., all bug fixes)
- **Scope**: Same component or module (e.g., all API changes)
- **Dependencies**: Changes that depend on each other
- **File proximity**: Related files in the same directory or feature

**Example groupings:**

Changes to group together:
- `src/auth/login.ts` + `src/auth/session.ts` → Authentication feature
- `tests/auth.test.ts` + `tests/session.test.ts` → Authentication tests
- `README.md` + `docs/api.md` → Documentation updates

Changes to separate:
- New feature in `src/feature.ts` ≠ Bug fix in `src/auth.ts`
- Test file updates ≠ Source code changes (unless tightly coupled)
- Documentation ≠ Code changes (unless documenting new code)

Present the analysis showing:
1. Total changes detected
2. Proposed logical groups with files in each group
3. Conventional commit type for each group
4. Brief description of what each group accomplishes

### Step 3: Ask for Commit Strategy

Use AskUserQuestion to let the user choose commit granularity:

```
Question: "How would you like to structure these commits?"

Options:
1. "Separate small commits" - One commit per logical group (detailed history)
2. "Combined medium commits" - Group related changes (balanced approach) (Recommended)
3. "Single large commit" - All changes in one commit (simple history)
```

**Guidance for recommendations:**
- **Separate small commits**: Best for ≥4 distinct logical groups, complex features
- **Combined medium commits**: Best for 2-3 logical groups, typical development work
- **Single large commit**: Best for 1 logical group, simple changes, prototypes

### Step 4: Generate Commit Messages

Based on the user's choice, generate conventional commit messages:

**For each commit group:**

1. **Determine type**: Select the most appropriate conventional commit type
2. **Identify scope**: Determine the component/module (optional but recommended)
3. **Write subject**: Clear, imperative description (≤50 chars)
4. **Add body** (if needed): Explain what changed and why (not how)
5. **Add footer** (if needed): Breaking changes, issue references

**Subject line rules:**
- Use imperative mood ("add" not "added" or "adds")
- Don't capitalize first letter after colon
- No period at the end
- Keep under 50 characters
- Be specific but concise

**Body guidelines:**
- Wrap at 72 characters
- Explain what and why, not how
- Use bullet points for multiple changes
- Reference issue numbers when relevant

**Footer format:**
- Breaking changes: `BREAKING CHANGE: description`
- Issue references: `Closes #123, Fixes #456`
- Co-authors: `Co-authored-by: Name <email>`

**Example commit messages:**

```
feat(auth): add JWT token refresh mechanism

Implement automatic token refresh to improve user experience
by preventing unexpected logouts during active sessions.

- Add refresh token endpoint
- Implement token rotation strategy
- Add retry logic for failed refresh attempts

Closes #234
```

```
fix(api): handle null responses in user service

Previously, null user responses caused uncaught exceptions.
Add null checks and return appropriate error messages.

Fixes #456
```

```
docs(readme): update installation instructions

Add troubleshooting section for common installation issues
and clarify dependency requirements for Node.js 18+.
```

### Step 5: Stage and Commit Changes

After presenting commit messages:

1. **Show commit plan**: Display all proposed commits with files and messages
2. **Stage files**: Use `git add` for specific files in each group
3. **Create commits**: Use heredoc format for multi-line messages
4. **Verify**: Run `git log` to confirm commits were created correctly

**Staging strategy:**
- For separate/combined commits: Stage only files for each specific commit
- For single commit: Stage all changed files at once
- Always stage specific files by path, never use `git add .` or `git add -A`

**Commit execution:**

```bash
# Use heredoc for proper formatting
git commit -m "$(cat <<'EOF'
feat(auth): add JWT token refresh mechanism

Implement automatic token refresh to improve user experience
by preventing unexpected logouts during active sessions.

- Add refresh token endpoint
- Implement token rotation strategy
- Add retry logic for failed refresh attempts

Closes #234

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
EOF
)"
```

**Important notes:**
- Never use `--no-verify` flag
- Always include Co-Authored-By footer
- If pre-commit hook fails, fix issues and create NEW commit (never amend)
- Run `git status` after each commit to verify

### Step 6: Present Summary

After creating commits, provide a clear summary:

```
✓ Created 3 conventional commits:

1. feat(auth): add JWT token refresh mechanism
   Files: src/auth/token.ts, src/auth/refresh.ts

2. test(auth): add token refresh test coverage
   Files: tests/auth/token.test.ts

3. docs(auth): document token refresh flow
   Files: docs/authentication.md, README.md

All changes committed successfully.
```

## Edge Cases

**No changes to commit:**
- Inform that there are no changes
- Do not create empty commits

**Merge conflicts:**
- Identify conflicted files from git status
- Advise resolving conflicts before committing
- Do not attempt to commit with unresolved conflicts

**Pre-commit hook failures:**
- Show the hook error message
- Fix the reported issues (linting, formatting, etc.)
- Re-stage fixed files
- Create a NEW commit (never amend unless explicitly requested)

**Mixed staged/unstaged changes:**
- Clearly identify what is staged vs unstaged
- Ask if unstaged changes should be included
- Only commit confirmed changes

**Very large changesets (>20 files):**
- Suggest more granular grouping
- Consider recommending combined commits over single large commit
- Ensure commit messages adequately describe scope

## Best Practices

**DO:**
- Review all changes before grouping
- Group related changes logically
- Use consistent commit types throughout the project
- Write clear, imperative subject lines
- Include body when context is needed
- Reference issue numbers
- Always add Co-Authored-By footer
- Verify commits were created correctly

**DON'T:**
- Create commits without reviewing changes
- Mix unrelated changes in one commit
- Use vague subjects like "fix bug" or "update files"
- Exceed 50 characters in subject lines
- Use past tense ("added", "fixed")
- Capitalize subject line after type/scope
- End subject lines with periods
- Skip verification step

## Additional Resources

### Reference Files

For detailed conventional commit specifications and examples:
- **`references/conventional-commits-spec.md`** - Full specification with examples
- **`references/commit-patterns.md`** - Common patterns and anti-patterns

### Example Files

Working examples of commit workflows:
- **`examples/multi-commit-workflow.sh`** - Example of creating multiple commits
- **`examples/commit-messages.txt`** - Example commit message formats

### Scripts

Utility scripts for validation:
- **`scripts/validate-commit-msg.sh`** - Validate commit message format
- **`scripts/group-changes.py`** - Analyze and suggest change groupings

## Quick Reference

**Common Types:**
```
feat     - New feature
fix      - Bug fix
docs     - Documentation
style    - Formatting
refactor - Code restructuring
test     - Tests
chore    - Maintenance
```

**Template:**
```
<type>(<scope>): <subject ≤50 chars>

<body wrapped at 72 chars>

<footer: BREAKING CHANGE, Closes #123>

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

**Grouping Strategy:**
```
Small commits  → Detailed history, many commits
Medium commits → Balanced (recommended)
Large commit   → Simple history, one commit
```