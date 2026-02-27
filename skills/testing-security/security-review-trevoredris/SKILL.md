---
name: security-review
description: Perform security-focused code review to identify HIGH-CONFIDENCE vulnerabilities with real exploitation potential. Based on Anthropic's claude-code-security-review. Minimizes false positives with >80% confidence threshold. Use when reviewing PRs for security issues.
context: fork
agent: security-review
allowed-tools: Bash(git:*), Bash(gh:*), Read, Glob, Grep, LS, Task
---

# Security Review

Conduct a security-focused code review of the pending changes.

## Context

GIT STATUS:

```
!`git status`
```

PR METADATA:

```
!`gh pr view --json title,body,author,files,additions,deletions,headRefName,state 2>/dev/null || echo "No PR context - reviewing local branch"`
```

FILES MODIFIED:

```
!`gh pr diff --name-only 2>/dev/null || git diff --name-only origin/HEAD...`
```

COMMITS:

```
!`gh pr view --json commits --jq '.commits[].messageHeadline' 2>/dev/null || git log --oneline origin/HEAD...`
```

DIFF CONTENT:

```
!`gh pr diff 2>/dev/null || git diff --merge-base origin/HEAD`
```

## Objective

Use the security-review agent to identify HIGH-CONFIDENCE security vulnerabilities in the diff above. Focus only on issues with >80% confidence of real exploitability. Your final reply must contain the markdown report.

## References

- `references/OWASP_TOP_10.md` — OWASP Top 10 quick reference
- `references/FALSE_POSITIVE_GUIDE.md` — Detailed false positive filtering guide
