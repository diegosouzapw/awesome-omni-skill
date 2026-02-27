---
name: code-reviewer
description: Review code changes for correctness, security, performance, and maintainability. Use after writing/modifying code or before merging, and produce prioritized findings with concrete fixes.
---

# Code Reviewer

## Workflow

1. Identify review scope:
   - Review the uncommitted diff, a target commit, or a PR.
   - Focus on changed files; sample nearby call sites when needed.
2. Review by priority:
   - Correctness and edge cases
   - Security (secrets, injection, authz, unsafe deserialization, path traversal)
   - Reliability (error handling, retries/timeouts)
   - Maintainability (naming, complexity, duplication)
   - Performance (hot paths, N+1, unnecessary renders/queries)
3. Verify quality gates:
   - Confirm tests exist and cover behavior changes.
   - Recommend minimal new tests when coverage is missing.

## Output format

- Group findings as **Critical**, **Warnings**, and **Suggestions**.
- For each issue, provide:
  - What is wrong and why it matters
  - Where it is (file and line when possible)
  - How to fix it (concrete guidance or patch sketch)

## Reference

- Read `references/code-reviewer.md` for deeper checklists and examples.
