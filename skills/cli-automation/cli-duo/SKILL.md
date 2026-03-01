---
name: cli-duo
description: Coordinate two AI CLIs on the same repo without conflicts (subordinate worktrees + engineer/judge roles).
metadata:
  version: 0.1.0
  author: mcarls
  category: collaboration
compatibility:
  - codex-cli
  - claude-code
  - cursor
  - gemini-cli
---

# CLI Duo

Orchestrates two AI CLIs working on the same repository. Supports safe subordinate worktrees and engineer/judge review loops.

## When to Use

- Two CLIs need to edit the same repo without clobbering each other's state
- You want a create → judge loop with explicit role swaps
- You need a temporary subordinate worktree for experiments

## Core Concepts

- **CLI Registry**: Store known CLIs with names, commands, and default repos.
- **Sessions**: Pair two registered CLIs in either mode:
  - `subordinate`: Primary stays on the main repo; secondary gets a git worktree sandbox.
  - `engineer-judge`: Assigns engineer/judge roles and lets you swap them per round.

## Commands

```bash
# Register CLIs
duo cli register -n claude -c "claude" -d "Claude Code CLI"
duo cli register -n cursor -c "cursor" -d "Cursor CLI"
duo cli list

# Subordinate mode (creates secondary worktree)
duo pair create -s landing-refactor -p claude -b cursor -m subordinate -r ~/projects/app --confirm
duo pair info -s landing-refactor
duo pair end -s landing-refactor --confirm  # removes worktree + session

# Engineer/Judge mode
duo pair create -s review-loop -p claude -b cursor -m engineer-judge -R 2
duo pair swap -s review-loop  # swap roles after a round
duo pair info -s review-loop
```

## Outputs

- Session summaries include the working directories each CLI should use.
- For subordinate sessions, the secondary path is a git worktree under `~/.local/share/cli-duo/worktrees/<session>/secondary`.

## Cleanup

- `duo pair end -s <session> --confirm` tears down the worktree (if created) and deletes the session.
