---
name: github-ops
description: Workflow for repository reconnaissance and operations using GitHub CLI (gh). Optimizes token usage by using structured API queries instead of blind file fetching.
version: 1.1.0
model: sonnet
invoked_by: both
user_invocable: true
tools: [Bash, Read]
best_practices:
  - List directory contents before fetching specific files
  - Use --jq to filter JSON output and reduce token noise
  - Combine multiple checks into single API calls where possible
error_handling: graceful
streaming: supported
verified: true
lastVerifiedAt: 2026-02-22T00:00:00.000Z
---

# GitHub Ops Skill

Provides structured guidance for repository reconnaissance using `gh api` and `gh search`.

## Overview

Repository reconnaissance often fails when agents guess file paths or attempt to fetch large files blindly. This skill enforces a structured `Map -> Identify -> Fetch` sequence using the GitHub CLI to minimize token waste and improve reliability.

## ⚡ Essential Reconnaissance Commands

Use these commands to understand a repository structure before fetching content.

### 1. List Repository Root

```bash
gh api repos/{owner}/{repo}/contents --jq '.[].name'
```

### 2. List Specific Directory

```bash
gh api repos/{owner}/{repo}/contents/{path} --jq '.[].name'
```

### 3. Fetch File Content (Base64 Decoded)

```bash
gh api repos/{owner}/{repo}/contents/{path} --jq '.content' | base64 -d
```

### 4. Search for Pattern in Repository

```bash
gh search code "{pattern}" --repo {owner}/{repo}
```

### 5. Get Repository Metadata

```bash
gh repo view {owner}/{repo} --json description,stargazerCount,updatedAt
```

## 🔄 Token-Efficient Workflow

1.  **Map Tree**: List the root and core directories (`commands`, `src`, `docs`).
2.  **Identify Entrypoints**: Look for `README.md`, `gemini-extension.json`, `package.json`, or `SKILL.md`.
3.  **Targeted Fetch**: Download only the entrypoints first.
4.  **Deep Dive**: Use `gh search code` to find logic patterns rather than reading every file.

## 🛡️ Platform Safety (Windows)

- When using `base64 -d`, ensure the output is redirected to a file using the `Write` tool if it's large.
- Avoid Linux-style `/dev/stdin` patterns in complex pipes.
- Use native paths for any local storage.

## Iron Laws

1. **ALWAYS** follow the Map → Identify → Fetch sequence before reading any file — blindly fetching files by guessed path wastes tokens, triggers 404s, and produces hallucinated repo structure.
2. **NEVER** fetch a file without first listing its parent directory or confirming it exists via `gh api` — large files fetched unnecessarily can exhaust the context window.
3. **ALWAYS** use `--jq` to filter `gh api` JSON output to only the fields needed — unfiltered API responses contain hundreds of irrelevant fields that inflate token usage.
4. **NEVER** use `gh search code` without a scoping qualifier (repo, org, or path) — unscoped code search returns results from all of GitHub, producing irrelevant noise.
5. **ALWAYS** prefer `gh api` structured queries over reading repository files directly when repository metadata is needed — API queries are faster, structured, and don't require authentication context for public repos.

## Anti-Patterns

| Anti-Pattern                                   | Why It Fails                                                    | Correct Approach                                                                         |
| ---------------------------------------------- | --------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Guessing file paths and fetching them directly | High 404 rate; wasted tokens on non-existent paths              | Map root tree first: `gh api repos/{owner}/{repo}/git/trees/HEAD --jq '.tree[].path'`    |
| Fetching entire files for a single field       | Large files exhaust context; slow and imprecise                 | Use `--jq` to extract only the required field from API response                          |
| Unscoped `gh search code` queries              | Returns GitHub-wide results; noise overwhelms signal            | Always add `--repo owner/name` or `--owner org` scope qualifier                          |
| Reading binary or generated files              | Binary content is unreadable; generated files change frequently | Identify file type first; skip binaries; read source files only                          |
| Sequential API calls for each file             | Unnecessary round-trips inflate latency                         | Batch: use `gh api` trees or search to identify multiple targets, then fetch in parallel |

## Assigned Agents

- **artifact-integrator**: Lead agent for repository onboarding.
- **developer**: PR management and exploration.

## Memory Protocol (MANDATORY)

**Before starting:**
Read `.claude/context/memory/learnings.md`

**After completing:**

- New pattern -> `.claude/context/memory/learnings.md`
- Issue found -> `.claude/context/memory/issues.md`
- Decision made -> `.claude/context/memory/decisions.md`

> ASSUME INTERRUPTION: If it's not in memory, it didn't happen.
