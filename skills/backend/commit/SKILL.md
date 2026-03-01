---
name: commit
description: Create commits following project guidelines.
---

# Commit

Create commits following the commit guidelines.

## Instructions

When invoked, create a single git commit:

1. Review changes and recent commits
2. Draft message following conventional commit format
3. Stage files and create commit

## Commit Message Rules

### 1. Use Conventional Commit Format

**Rule:** Use conventional commit prefixes like `feat:`, `fix:`, `chore:`,
`docs:`, `refactor:`, etc.

**Scope:** When adding, modifying, or deleting a specific plugin, use the plugin
name as the scope.

**Examples:**

- ✅ `feat(my-plugin): add new feature`
- ✅ `fix(auth-plugin): fix token expiration`
- ✅ `chore(my-plugin): update dependencies`
- ✅ `docs: update README`
- ✅ `refactor: improve error handling`

### 2. Start with Lowercase

**Rule:** The message after the prefix should start with a lowercase letter.

**Examples:**

- ✅ `feat(plugin): add new feature`
- ❌ `feat(plugin): Add new feature`

### 3. Keep Messages Concise

**Rule:** Commit messages should be clear and scannable.

**Why:** Short messages are easier to scan in git log.

**Examples:**

- ✅ `feat(user-auth): add login endpoint`
- ✅ `fix(payment): handle null amount`
- ❌
  `feat(user-auth): add a new user authentication endpoint with JWT support and validation`
  (too long)

### 4. No Co-Author Attributions

**Rule:** Do NOT include `Co-Authored-By:` lines in commit messages.

**Why:** Attribution is handled at the PR level, not commit level.

**Examples:**

- ✅ `chore: update database schema`
- ❌
  `chore: update database schema\n\nCo-Authored-By: Claude <noreply@anthropic.com>`

### 5. No AI Agent Mentions

**Rule:** Do NOT mention AI tools or agents in commit messages.

**Why:** Commits should describe what changed, not how it was created.

**Examples:**

- ✅ `refactor: refactor payment processing logic`
- ❌ `refactor: refactor payment processing logic with Claude`

### 6. Match Repository Style

**Rule:** Follow the style of recent commits in the repository.

**How:** Run `git log --oneline -10` to see recent commit messages and match
their tone and format.

## Message Guidelines

### Focus on WHAT, not WHY or HOW

**Good:**

- `feat(search): add product search API`
- `fix(shipping): fix cost calculation`
- `chore: remove deprecated endpoints`

**Bad:**

- `feat(search): add product search API to improve UX` (includes why)
- `fix(shipping): fix cost calculation using new formula` (includes how)

### Use Imperative Mood

**Good:**

- `feat(profile): add user profile page`
- `fix(auth): fix login redirect bug`
- `chore: remove unused imports`

**Bad:**

- `feat(profile): added user profile page` (past tense)
- `fix(auth): fixes login redirect bug` (present tense)
- `chore: removing unused imports` (gerund)
