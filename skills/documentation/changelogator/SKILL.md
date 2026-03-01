---
name: changelogator
description: Generate a changelog from git commits. Use when the user asks to generate a changelog, prepare release notes, summarize commits, prepare a release, suggest a version bump, or review changes since last tag. Also use when the user mentions "changelog", "release notes", "what changed", "version bump", or "semver".
---

# Changelogator

Generate a structured changelog from git commits following [Keep a Changelog](https://keepachangelog.com/) format.

**By default**, output the changelog in chat for the user to review — do **not** write to `CHANGELOG.md` unless the user explicitly asks.

## Workflow

1. **Find the last tag**: `git tag --sort=-v:refname | head -1`
2. **Retrieve commits**: `git log <tag>..HEAD --format="---COMMIT---%n%H%n%s%n%b" --reverse` (if no tag, use `git log --format="---COMMIT---%n%H%n%s%n%b" --reverse`)
3. **Parse** each commit subject using the format below
4. **Classify** into changelog categories using the mapping table
5. **Detect breaking changes**: `!` after type (e.g., `feat!`) or `BREAKING CHANGE` in body
6. **Suggest semver bump** based on the rules below
7. **Generate** the markdown changelog
8. **Output in chat** for validation

## Commit Format

Expected format (from commitor):

```
<emoji> <type>(<scope>): <description> (<TASK-XXX>)
```

Parse using these parts:
- **Emoji**: leading Unicode emoji
- **Type**: word after emoji (`feat`, `fix`, `patch`, etc.)
- **Scope**: optional, inside parentheses after type
- **Breaking**: `!` between type/scope and `:`
- **Description**: text after `: `
- **Task number**: optional, inside parentheses at end of subject

## Category Mapping

Map commit types to changelog categories (consistent with commitor):

| Changelog Category | Emoji | Commit Types                     |
| ------------------ | ----- | -------------------------------- |
| Added              | ✨    | feat                             |
| Changed            | 🔨    | patch, style, perf, data         |
| Fixed              | 🐛    | fix                              |
| Removed            | 🔥    | remove                           |
| Technical          | ⚙️    | docs, refactor, test, ai, config |

## Semver Rules

Suggest a version bump based on the most significant change:

| Condition                           | Bump  |
| ----------------------------------- | ----- |
| Any breaking change (`!` or body)   | Major |
| Any `feat` commit                   | Minor |
| Everything else (fix, patch, etc.)  | Patch |

Apply to the last tag version. If no tag exists, suggest `v0.1.0`.

## Output Template

```markdown
## vX.Y.Z - YYYY-MM-DD

### ✨ Added
- **scope**: description
- description without scope

### 🔨 Changed
- **scope**: description

### 🐛 Fixed
- description

### 🔥 Removed
- description

### ⚙️ Technical
- description
```

## Output Rules

- **Rewrite descriptions** — do not copy commit subjects verbatim. Rephrase into clear, human-readable sentences. Use backticks for code references (skill names, commands, files, config keys). Capitalize the first word.
- **Omit empty categories** — only show categories that have commits
- **Scope as bold prefix** when present: `- **scope**: description`
- **No scope**: just `- description`
- **Skip merge commits** — ignore subjects starting with `Merge`
- **Include task numbers** when present: `- **scope**: description (TASK-123)`
- **English only**

## Edge Cases

- **No tag found**: use all commits and suggest `v0.1.0`
- **No commits since tag**: output "No changes since `<tag>`."
- **Unrecognized format**: classify under "Other" with the full subject as description
- **Multiple breaking changes**: still one major bump, list all in a `### ⚠️ Breaking Changes` section at the top

## Example

**Input commits** (from `git log v1.2.0..HEAD`):

```
✨ feat(auth): add OAuth2 login with Google (NTT-128)
🐛 fix(exports): prevent duplicate rows in CSV export
🔨 patch(ui): improve loading spinner animation
♻️ refactor: simplify database connection pooling
🔥 remove(api): drop deprecated v1 endpoints
```

**Output**:

```markdown
## v1.3.0 - 2026-02-12

### ✨ Added
- **auth**: `OAuth2` login with Google provider (NTT-128)

### 🔨 Changed
- **ui**: Improve the loading spinner animation

### 🐛 Fixed
- **exports**: Fix duplicate rows appearing in CSV exports

### 🔥 Removed
- **api**: Drop deprecated `v1` API endpoints

### ⚙️ Technical
- Simplify database connection pooling
```

Suggested version: **v1.3.0** (minor bump — contains new feature)
