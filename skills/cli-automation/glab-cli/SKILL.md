---
name: glab-cli
description: GitLab CLI (glab) utilities - pipeline debugging, MR management, release automation. Use when working with GitLab via CLI for anything beyond basic git commands.
---

# GitLab CLI Power Utilities

Intelligent wrappers and patterns for GitLab CLI operations.

## When to Use

- Debugging failed GitLab CI/CD pipelines
- Managing merge requests across projects
- Automating releases
- Bulk operations on issues/MRs
- Project health checks

## Prerequisites

```bash
# macOS
brew install glab

# Or download from https://gitlab.com/gitlab-org/cli

# Authenticate
glab auth login
```

## Core Capabilities

### 1. Pipeline Debugging

Get actionable errors from failed pipelines.

```bash
# Diagnose most recent failure
python3 scripts/glab_ops.py diagnose

# Specific project
python3 scripts/glab_ops.py diagnose --project group/project

# With suggested fixes
python3 scripts/glab_ops.py diagnose --suggest-fixes
```

**Output:** Job failures, error excerpts, timing info.

### 2. MR Operations

Merge request management at scale.

```bash
# List MRs needing review
python3 scripts/glab_ops.py mr-review-queue

# MRs ready to merge (approved + passing)
python3 scripts/glab_ops.py mr-merge-ready --dry-run

# MR health check
python3 scripts/glab_ops.py mr-status 123
```

### 3. Release Automation

Create releases with changelogs.

```bash
# Preview release
python3 scripts/glab_ops.py release --dry-run

# Create minor release
python3 scripts/glab_ops.py release --bump minor

# With custom notes
python3 scripts/glab_ops.py release --notes "Breaking: API v2"
```

## Quick Patterns

```bash
# Watch pipeline
glab ci view --live

# Retry failed jobs
glab ci retry

# View job logs
glab ci trace <job-id>

# Create MR from current branch
glab mr create --fill

# Checkout MR locally
glab mr checkout 123

# View MR changes
glab mr diff 123

# Approve MR
glab mr approve 123

# Merge MR
glab mr merge 123 --squash --remove-source-branch
```

## glab vs gh Quick Reference

| Task | GitHub (gh) | GitLab (glab) |
|------|-------------|---------------|
| Create PR/MR | `gh pr create` | `glab mr create` |
| List CI runs | `gh run list` | `glab ci list` |
| View CI logs | `gh run view --log` | `glab ci trace` |
| Watch CI | `gh run watch` | `glab ci view --live` |
| Retry CI | `gh run rerun --failed` | `glab ci retry` |
| Checkout PR/MR | `gh pr checkout 123` | `glab mr checkout 123` |

## Integration

Works with:
- **wicked-search**: Find code, create issues via glab
- **wicked-kanban**: Track MR status alongside tasks

## References

- `refs/glab-ops.md` - Full script documentation
- `refs/patterns.md` - Common workflow patterns
- `refs/ci-debugging.md` - Pipeline troubleshooting
