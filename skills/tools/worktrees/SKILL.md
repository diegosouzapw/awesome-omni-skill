---
name: worktrees
description: Multi-agent parallel development using git worktrees. Use when user says "plan workstreams", "create worktree", "spawn worktree", "write status", "check status", "coordinate agents", "parallel development", or asks about managing multiple Claude instances working on the same codebase.
metadata:
  author: tobiadeangelis
  version: "2.0"
---

# Worktrees

Coordinate multiple Claude agents working in parallel on the same codebase using git worktrees.

## Key Concept

**Workers are separate Claude sessions**, not subagents. The Manager creates worktrees, then the user opens a new terminal in each worktree and runs `claude`. Each Claude instance operates independently.

## Concepts

- **PLAN.md** — Coordination document listing workstreams, file ownership, and decisions
- **STATUS-{name}.md** — Progress updates from each workstream (REQUIRED before merge)
- **Manager** — Plans on main branch, creates/merges worktrees, monitors progress
- **Worker** — Executes in worktree, respects file ownership, writes status

## Determine Your Role

```bash
git branch --show-current
```

- `main` or `master` → **Manager**
- `wt/*` branch → **Worker** for that workstream

---

## Manager Workflow

### 1. Pre-Flight Checklist

Before creating worktrees, ensure:

- [ ] Foundation code committed on main
- [ ] `pnpm install` (or npm/yarn) works
- [ ] `pnpm build` passes
- [ ] Shared files ready (types, utils, mock data)
- [ ] No file ownership overlap between planned workstreams

### 2. Plan Workstreams

Create PLAN.md in project root. See `references/plan-template.md` for structure.

**Critical:** Define explicit file ownership to prevent merge conflicts. Be specific:

```
Good: apps/web/components/layout/*, apps/web/app/(dashboard)/layout.tsx
Bad:  apps/web/**  # Too broad, will conflict
```

### 3. Create Worktrees

```bash
git worktree add ../projectname-{name} -b wt/{name}
cd ../projectname-{name} && pnpm install  # Install deps in each
```

Then tell the user:
```
Worktrees ready. Start workers:
  cd ~/projectname-{name} && claude
  cd ~/projectname-{other} && claude
```

### 4. Monitor Progress (Heartbeat)

Periodically check worker status:

```bash
# Quick status check
echo "=== COMMITS ===" && git log wt/* --oneline -3
echo "=== STATUS FILES ===" && cat STATUS-*.md 2>/dev/null || echo "None yet"
echo "=== UNCOMMITTED ===" && for d in ../projectname-*; do echo "$d:"; git -C "$d" status --short | head -5; done
```

### 5. Merge and Cleanup

Only merge when STATUS file shows "Ready for merge: yes" and build passes.

```bash
# Verify build in worktree first
cd ../projectname-{name}/apps/web && pnpm build

# Merge (back on main)
cd /path/to/main
git merge wt/{name} -m "Merge wt/{name}: Brief description"

# Cleanup
git worktree remove ../projectname-{name} --force
git branch -d wt/{name}
```

### 6. Update PLAN.md

Move completed workstream to "Completed Workstreams" section.

---

## Worker Workflow

### 1. Orientation (Start of Session)

When you see you're on a `wt/*` branch:

```
You are Worker for `wt/{name}`.

1. Read PLAN.md for your scope and file ownership
2. ONLY modify files listed under your "Owns:" section
3. Commit frequently with clear messages
4. Write STATUS-{name}.md when done (REQUIRED)
```

Then:
- Read PLAN.md — find your workstream, understand scope
- Read STATUS-{name}.md if it exists (resuming previous session)

### 2. While Working

- **Only modify files assigned to your workstream** — this is critical
- Commit early and often (every logical chunk)
- If you need something from another workstream, note it as a blocker

### 3. Before Finishing (REQUIRED)

Write STATUS-{name}.md in project root. See `references/status-template.md`.

**Do not consider work complete without a STATUS file.**

```bash
# Final steps
pnpm build  # Verify build passes
git add . && git commit -m "Add worker status file"
```

---

## File Ownership Rules

Merge conflicts kill productivity. Strict ownership prevents them.

| Pattern | Risk |
|---------|------|
| `components/sidebar/*` | Safe — specific directory |
| `lib/utils.ts` | Safe — specific file |
| `components/**` | Dangerous — too broad |
| `*.ts` | Dangerous — overlaps everything |

If two workstreams might touch the same file, either:
1. Assign it to one workstream only
2. Put it in "Shared Files (Manager Only)" section
3. Split the file first

---

## Quick Reference

| Command | Purpose |
|---------|---------|
| `git worktree add ../proj-api -b wt/api` | Create worktree |
| `git worktree list` | List all worktrees |
| `git log wt/* --oneline -3` | Check recent commits |
| `git worktree remove ../proj-api --force` | Remove worktree |
| `git branch -d wt/api` | Delete branch after merge |

---

## Troubleshooting

**Worker can't read/write files:** Check you're in the right directory. Worktrees are sibling directories, not subdirectories.

**Merge conflicts:** File ownership wasn't specific enough. Resolve manually, then tighten ownership rules for next time.

**Worker didn't write STATUS:** Work is not complete. Ask them to write it before merging.

**Build fails after merge:** Run build in each worktree before merging. Merge in dependency order (shared code first).
