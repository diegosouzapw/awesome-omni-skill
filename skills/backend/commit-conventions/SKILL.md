---
name: commit-conventions
description: This skill should be used when writing commit messages, when asked about commit format, when reviewing a commit message, or when creating a git commit. Provides Conventional Commits format and project-specific conventions.
version: 0.1.0
user-invocable: false
---

# Commit Message Conventions

This project follows [Conventional Commits](https://www.conventionalcommits.org/).

## Format

```
{type}({optional-scope}): {subject}

{optional body}

{optional footer}
```

## Types

| Type | When to Use |
|---|---|
| `feat` | New feature or user-visible capability |
| `fix` | Bug fix |
| `refactor` | Code change that neither adds a feature nor fixes a bug |
| `test` | Adding or modifying tests only |
| `docs` | Documentation changes only |
| `chore` | Build system, tooling, dependency updates |
| `perf` | Performance improvement |
| `ci` | CI/CD pipeline changes |
| `style` | Formatting only — no logic change |

## Subject Line Rules

- Imperative mood: "add feature" not "added" or "adding"
- Lowercase first letter
- No period at the end
- Max 72 characters
- Be specific: "fix null pointer in UserService.getById" not "fix bug"

## Scope (optional)

Identifies the part of the codebase affected. Use the feature or module name:
- `feat(auth): add OAuth2 login`
- `fix(payment): handle timeout on charge API`
- `chore(deps): update riverpod to 2.5.0`

## Body

Explain the **why**, not the what. The diff shows what changed.

```
fix(auth): handle expired session tokens on app resume

Previously, the app silently failed on resume when the session token
had expired, leaving users on a blank screen. Now we detect expiry
on AppLifecycleState.resumed and redirect to the login screen.

Closes #234
```

## Breaking Changes

Add `BREAKING CHANGE:` in the footer:

```
feat(api)!: remove deprecated v1 endpoints

BREAKING CHANGE: /api/v1/* endpoints removed. Migrate to /api/v2/*.
See migration guide in docs/api-migration.md
```

The `!` after the type also signals a breaking change.

## Footer Keywords

- `Closes #123` — closes a GitHub issue on merge
- `Fixes #123` — same as Closes
- `Related to #123` — links without closing
- `Co-authored-by: Name <email>` — multiple authors

## Examples

```
feat(checkout): add Apple Pay support

fix(ui): prevent keyboard overlap on login form

refactor(data): extract UserMapper from UserRepository

test(auth): add integration tests for token refresh flow

chore(deps): bump flutter to 3.19.0
```

## What to Avoid

- "WIP", "fix", "update", "changes" — too vague
- Commit messages describing only the diff ("add null check")
- Past tense ("added feature X")
- Very long subject lines (use the body instead)
