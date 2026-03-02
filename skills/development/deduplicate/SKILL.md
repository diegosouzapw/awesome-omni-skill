---
name: deduplicate
description: Find and refactor duplicate code. Use this skill when the user wants to find near-duplicate code, check for copy-paste redundancy, or DRY up a codebase — optionally scoped to changed files.
metadata:
  author: "swissarmyhammer"
  version: "1.0"
---

# Deduplicate

Find near-duplicate code using tree-sitter semantic similarity analysis, then refactor to eliminate redundancy.

## Process

### 1. Determine scope

Ask yourself: did the user specify files, or do they want to check what's changed?

- **Changed files** — call `git_changes` with no arguments to get the list of files modified on the current branch:

```json
{"op": "git changes"}
```

- **Specific files** — the user named files directly. Use those.
- **Whole codebase** — the user asked for a broad sweep with no file constraint.

### 2. Check index readiness

Before querying, confirm the tree-sitter index is ready:

```json
{"op": "get status"}
```

If the index is not ready, tell the user and wait.

### 3. Find duplicates

Run duplicate detection scoped to the files from step 1.

**For each changed or specified file**, call treesitter scoped to that file:

```json
{"op": "find duplicates", "file": "src/handlers/user.rs"}
```

This finds code in that file that is semantically similar to code elsewhere in the codebase.

**For a whole-codebase sweep**, call without a file:

```json
{"op": "find duplicates", "min_similarity": 0.85, "min_chunk_bytes": 100}
```

Adjust thresholds based on the user's intent:
- Strict (exact copies): `min_similarity: 0.95`
- Default (near duplicates): `min_similarity: 0.85`
- Loose (similar patterns): `min_similarity: 0.70`

### 4. Analyze and report

For each duplicate cluster found, assess:

- **What is duplicated** — summarize the shared logic in one sentence
- **Where it lives** — list every location (file:lines)
- **Severity** — how much code is repeated, and how many copies exist
- **Refactoring opportunity** — propose a concrete extraction: a shared function, trait implementation, helper module, or generic abstraction

Present results grouped by severity (most duplicated first). Skip trivial clusters (boilerplate, single-line patterns, or auto-generated code).

### 5. Refactor

If the user wants to proceed with refactoring:

1. Extract the shared logic into a single location (new function, module, or trait)
2. Replace every duplicate site with a call to the extracted code
3. Run tests after each extraction to confirm nothing broke
4. Re-run duplicate detection on the changed files to verify the duplication is resolved:

```json
{"op": "find duplicates", "file": "src/shared/new_helper.rs"}
```

## Guidelines

- Always scope to changed files when the user says "check my changes" or "what I've been working on" — use `git_changes` to get the file list
- When scoping to changed files, run `find duplicates` once per file — do not run a single unscoped scan and filter afterward
- Report only actionable duplication. Ignore: test fixtures, generated code, trait impl boilerplate, and single-line matches
- Prefer the smallest extraction that removes the duplication. Do not over-abstract
- When refactoring, preserve the public API — callers should not need to change unless the user explicitly wants an API change
- If duplicate code exists across different crates or packages, note the dependency implications before extracting
