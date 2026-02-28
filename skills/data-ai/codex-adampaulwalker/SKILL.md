---
name: codex
description: Review code changes, implementation plans, or specific files using OpenAI Codex CLI. Triggers on "review", "check", "validate", "codex". Perfect for maker-checker workflows where you want a second AI agent to review Claude's work before committing. Requires the Codex CLI to be installed (brew install codex-cli or from https://codex.storage.googleapis.com).
allowed-tools:
  - Bash
  - Read
---

# Codex Code Review Skill

**Version:** 1.0.0
**Author:** Community-contributed
**Requires:** [Codex CLI](https://codex.storage.googleapis.com) installed and configured

## Overview

Use the Codex CLI to perform independent code reviews on changes made during your Claude Code session. This enables a "proposer-checker-maker-checker" workflow where Claude Code proposes/implements changes and Codex provides an independent review.

## Prerequisites

Install the Codex CLI:
```bash
brew install codex-cli
# OR download from https://codex.storage.googleapis.com
```

Configure your OpenAI API key:
```bash
codex login
```

## Use the Codex CLI to review recent changes, implementation plans, or specific files.

## Review Recent Git Changes

```bash
# Review uncommitted changes
codex review "Review the uncommitted changes in this repo"

# Review specific commit
codex review "Review the changes in commit <hash>"

# Review changes between branches
codex review "Review the diff between main and current branch"
```

## Review Specific Files

```bash
# Review a specific file
codex review "Review the code in <file_path>"

# Review multiple files
codex review "Review the implementation in <file1> and <file2>"
```

## Review Implementation Plan

```bash
# Review a plan before implementation
codex review "Review this implementation plan: <plan_content>"
```

## Usage Pattern

1. Identify what needs review (recent changes, specific files, or a plan)
2. Run `codex review` with appropriate prompt
3. Present the review results to the user

The `codex review` command runs non-interactively and returns feedback that can be incorporated into the Claude Code session.

## Workflow Example

```
User: I just implemented user authentication
Claude: [implements auth.ts, auth-middleware.ts, etc.]
User: /codex
Claude: [runs: codex review --uncommitted]
Claude: Codex found 2 issues:
  1. Missing error handling in auth.ts:45
  2. Password hash strength could be improved
User: Fix those issues
Claude: [fixes issues]
User: /codex
Claude: [runs review again]
Claude: Codex approves! No issues found.
```

## Notes

- Works best in git repositories
- Review results are shown inline in the Claude Code session
- Combines well with Claude's implementation capabilities for a robust development workflow
