---
name: code-review
description: "Code review local changes"
allowed-tools: Bash(jj diff:*), Bash(jj log:*)
---

# Code Review

Review local changes using specialized subagents across 5 domains. Only noteworthy findings — no nitpicking.

**Agent assumptions (applies to all agents and subagents):**
- All tools are functional. Do not test tools or make exploratory calls.
- Only call a tool if required. Every tool call should have a clear purpose.
- If you are running a jj diff ALWAYS include the --git flag.

## Workflow

### 1. Gather context

Launch a sonnet agent to collect:
```bash
jj diff --git --from main --summary   # changed files
jj diff --git --from main             # full diff
jj log --limit 5                      # recent commits for context
```

### 2. Launch 5 specialized review agents in parallel

Pass each agent the full diff, file summary, and the CRITICAL FILTER below verbatim. Each returns a list of issues with: file, line, description, severity (critical/warning), and domain tag.

**Scope rule:** Only review lines starting with `+` in the diff. Ignore context lines (no `+`/`-` prefix). You are reviewing the change, not the file.

**Agent 1 — Security (opus)**
Scan for vulnerabilities in changed `+` lines only:
- Injection flaws (SQL, command, XSS)
- Authentication/authorization weaknesses
- Cryptographic misuse (weak hashing, predictable tokens)
- Sensitive data exposure (credentials, PII in responses)
- OWASP Top 10 patterns

**Agent 2 — Code Quality (sonnet)**
Review structural quality of changed code only:
- Clear logic errors, unreachable code, dead branches
- Type safety issues (any, missing null checks on nullable values)
- Error handling gaps (unhandled promise rejections, swallowed errors)
- CLAUDE.md compliance — quote the exact rule broken

**Agent 3 — Performance (sonnet)**
Flag performance concerns in changed code only:
- Algorithmic complexity issues (O(n²)+ where O(n) is possible)
- N+1 query patterns, unnecessary DB round-trips
- Missing pagination on unbounded queries
- Blocking operations in async contexts

**Agent 4 — Test Coverage (sonnet)**
Evaluate test adequacy for changed code:
- New public functions/endpoints without corresponding tests
- Changed behavior without updated tests
- Edge cases in critical paths (auth, payments, data mutation) lacking coverage
- Only flag when absence creates real risk, not for trivial utilities

**Agent 5 — Documentation Accuracy (sonnet)**
Cross-check user-facing docs against implementation:
- README/docs claims that contradict the actual code
- API documentation (JSDoc/TSDoc) with wrong parameters, return types, or behavior
- Outdated examples that won't work with current code
- Scope: README, CHANGELOG, API doc comments only. Ignore inline TODOs and dev notes

**CRITICAL FILTER for ALL agents:**

Only report issues that are **noteworthy** — a senior engineer would stop and comment on them. Specifically:

Flag:
- Will cause bugs, data loss, or security breach
- Misleads future developers (wrong docs, deceptive naming)
- Performance cliff that scales badly
- Missing tests for critical/risky code paths

Do NOT flag:
- Style preferences, formatting, naming conventions
- Linter-catchable issues
- Pre-existing issues outside the diff
- "Nice to have" improvements
- Issues that depend on unknown runtime state
- General suggestions without concrete problems

If uncertain whether an issue is real, do not flag it.

### 3. Validate flagged issues

For each issue from step 2, launch a parallel validation subagent:
- Security/logic issues → opus agent reads relevant code and confirms
- Quality/docs/test/perf issues → sonnet agent validates

Each validator gets the issue description and must confirm: **valid** or **false positive** with reasoning.

Reject as false positive if:
- Linter/formatter would catch it
- Issue is in unchanged context lines, not `+` lines
- Cosmetic or stylistic concern
- Would be auto-fixed by tooling
- Uncertain — when in doubt, reject

Filter out false positives.

### 4. Deduplicate and build action plan

Deduplicate by (file, line). If multiple agents flag the same location, merge into one issue with all relevant domain tags.

Assign unified severity:
- **P0** — data loss, security exploits, production crashes
- **P1** — logic errors, perf degradation at scale, public API without tests
- **P2** — misleading docs, missing error handling, moderate perf issues

For each issue include:
- Severity (P0/P1/P2)
- Domain tags
- File and line
- Description
- Suggested fix (committable suggestion for small fixes, description for larger ones)

Present sorted by severity. Ask user which items they want to resolve.

### 5. Execute selected fixes

Only fix items the user selects. For unselected items, leave them as-is without comment.

If NO issues found across all domains:
```
No issues found. Checked: security, code quality, performance, test coverage, documentation accuracy.
```

## Quick Reference

| Domain | Agent | Focus |
|--------|-------|-------|
| Security | opus | injection, auth, crypto, data exposure |
| Code Quality | sonnet | logic errors, types, error handling, CLAUDE.md |
| Performance | sonnet | complexity, N+1, unbounded queries |
| Test Coverage | sonnet | missing tests for risky/public code |
| Documentation | sonnet | code/docs mismatches |

## Common Mistakes

- **Flagging style issues** — no formatting, naming, or convention feedback unless CLAUDE.md requires it
- **Reviewing pre-existing code** — only review what's in the diff
- **Missing validation step** — every issue must be validated before reporting
- **Duplicate issues** — one entry per unique problem, even if multiple agents flag it
- **Fixing without asking** — present plan first, let user choose what to resolve
