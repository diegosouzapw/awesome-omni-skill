---
name: agent-browser-upstream
description: Safely sync navigator's agent-browser fork with upstream vercel-labs/agent-browser, analyze changes, and generate integration documentation
triggers:
  - upstream
  - agent-browser update
  - sync fork
  - integration plan
  - update agent-browser
  - merge upstream
archetype: dev-workflow
user_invocable: false
---

# Agent-Browser Upstream Sync Skill

Manages the process of keeping navigator's agent-browser fork in sync with the upstream vercel-labs/agent-browser repository.

## Related Skills

| Skill | Purpose |
|-------|---------|
| **[upstream-evaluation](../upstream-evaluation/SKILL.md)** | Decision frameworks for what to adopt/skip/adapt |
| **[docs/architecture/DESIGN.md](../../../docs/architecture/DESIGN.md)** | Navigator's design philosophy and conventions |

**Workflow**: Use `upstream-evaluation` skill for the "what and why" decisions, then return here for the "how" execution.

## Prerequisites

**None required** - the skill auto-manages everything:

1. **Auto-clone**: If `.agent-browser/repo/` doesn't exist, the script clones the fork automatically
2. **Auto-configure**: Upstream remote is added if missing
3. **Override**: Set `AGENT_BROWSER_LOCAL` env var to use an existing local clone instead

The `.agent-browser/` directory is gitignored and contains:
- `repo/` — Git clone of the fork
- `analysis/<sha>/` — Generated artifacts (jq-able JSON, diffs)

## Workflow Overview

```
Phase 1: Check Status (fetch, compare)
    ↓
Phase 2: Analyze Changes (categorize commits)
    ↓
Phase 3: Evaluate Changes ← uses upstream-evaluation skill
    ↓
Phase 4: Write Integration Docs + Create Issue
    ↓
Phase 5: User Review & Confirmation
    ↓
Phase 6: Execute Merge (ONLY after user confirms)
```

> **CRITICAL**: Never merge without completing phases 3-5 first.
> Phase 3 MUST use the `upstream-evaluation` skill to apply Navigator's design frameworks.

---

## Phase 1: Sync Fork

### Steps

1. **Run the analysis script** (handles everything automatically)
   ```bash
   bun run .claude/skills/agent-browser-upstream/scripts/analyze-upstream.ts --format summary
   ```

   The script will:
   - Clone `.agent-browser/` if missing
   - Add upstream remote if needed
   - Fetch latest from both remotes
   - Show divergence summary

2. **Or check manually**
   ```bash
   cd .agent-browser/repo

   # Current fork version
   git describe --tags origin/main 2>/dev/null || git rev-parse --short origin/main

   # Latest upstream version
   git describe --tags upstream/main 2>/dev/null || git rev-parse --short upstream/main

   # Divergence
   git log --oneline origin/main..upstream/main
   ```

### Decision Point

If no commits in divergence → **STOP** with "Fork is up to date with upstream"

If commits exist → **CONTINUE** to Phase 2 (never skip to merge)

---

## Phase 2: Analyze Changes

### Steps

1. **Run analysis script**
   ```bash
   bun run .claude/skills/agent-browser-upstream/scripts/analyze-upstream.ts \
     --repo "$REPO_PATH" \
     --base origin/main \
     --target upstream/main
   ```

2. **Review commit categories**
   The script outputs JSON with commits categorized as:
   - `breaking` - API changes, removed exports
   - `additive` - New features, new exports
   - `fix` - Bug fixes
   - `docs` - Documentation only
   - `chore` - Build, deps, tooling

3. **Identify key files changed**
   Focus on these files for navigator impact:
   - `src/protocol.ts` - MCP protocol definitions
   - `src/browser.ts` - Browser control API
   - `src/index.ts` - Public exports
   - `src/cli/` - CLI commands (may inform navigator CLI)

### Output

Present a summary table:

| Category | Count | Key Changes |
|----------|-------|-------------|
| Breaking | N | List significant ones |
| Additive | N | List new features |
| Fix | N | List relevant fixes |

---

## Phase 3: Evaluate Changes

**REQUIRED** — Even for "clean" updates with 0 breaking changes.

### Run the Evaluation Command

Use `/agent-browser:integrate-changes`, which:
- Loads the **upstream-evaluation** skill
- Provides decision frameworks from DESIGN.md
- Guides through structured evaluation process
- Outputs adopt/skip/defer tables for integration docs

### Why This Phase Exists

The 2025-01-20 v0.6.0 sync skipped evaluation because "0 breaking changes" was treated as a green light. This caused:
- Marketplace plugin merged without review (Vercel-branded, not navigator-appropriate)
- 8 new features not assessed for navigator integration
- No documentation created before merge

### Evaluation Output

The `upstream-evaluation` skill produces:
- **Adopt** table: Features to add with Navigator naming and schema changes
- **Skip** table: Features to exclude with rationale
- **Defer** table: Features to revisit later
- **Extend existing** table: Changes to existing actions

2. **Review evaluation output**
   - Ensure all new commands identified
   - Ensure plugins flagged
   - Ensure schema changes documented

3. **Do NOT proceed to merge** until analyst complete

---

## Phase 4: Impact Assessment (Navigator-Specific)

### Steps

1. **Map upstream changes to navigator usage**

   Check which navigator files import from agent-browser:
   ```bash
   grep -r "@outfitter/agent-browser" packages/*/src --include="*.ts" -l
   ```

2. **Cross-reference with changed APIs**

   For each breaking change, check if navigator uses it:
   ```bash
   # Example: if upstream changed BrowserOptions
   grep -r "BrowserOptions" packages/*/src --include="*.ts"
   ```

3. **Flag breaking changes**

   Create a list:
   - [ ] Change X affects `packages/server/src/browser.ts:42`
   - [ ] Change Y affects `packages/core/src/types.ts:18`

4. **Identify required navigator changes**

   For each breaking change, document:
   - What changed upstream
   - How navigator currently uses it
   - What navigator code needs to change

### Decision Point

If breaking changes exist → **STOP** and confirm with user before proceeding

---

## Phase 5: Write Integration Docs

### Steps

1. **Determine version**
   ```bash
   VERSION=$(git describe --tags upstream/main 2>/dev/null || echo "v$(date +%Y.%m.%d)")
   ```

2. **Create version directory**
   ```bash
   mkdir -p docs/_upstream/$VERSION
   ```

3. **Generate changes.md**
   Raw changelog with all commits and diffs.

4. **Generate integration.md**
   Use template from `references/integration-template.md`:
   - Version metadata
   - Breaking changes with navigator impact
   - Additive features with adoption plan
   - Required code changes
   - Test plan

5. **Generate status.md**
   Tracking checklist:
   ```markdown
   ## Implementation Status

   - [ ] Merge upstream into fork
   - [ ] Update navigator bun.lock
   - [ ] Apply breaking change fixes
   - [ ] Run navigator tests
   - [ ] Update navigator CHANGELOG
   ```

6. **Update index**
   Add entry to `docs/_upstream/README.md`

7. **Create GitHub issue** (optional, recommended for breaking changes)
   The integration doc *is* the issue — its frontmatter has title/labels:
   ```bash
   bun run .claude/skills/agent-browser-upstream/scripts/create-issue.ts \
     docs/_upstream/$VERSION/integration.md
   ```
   Or use the command: `/agent-browser:issue docs/_upstream/$VERSION/integration.md`

---

## Phase 6: Execute Merge

> **REQUIRES USER CONFIRMATION** - Do not proceed without explicit approval from Phase 5 docs review

### Steps

1. **Merge upstream into fork**
   ```bash
   cd /path/to/navigator/.agent-browser/repo  # Always use absolute path
   git checkout main
   git merge upstream/main --no-edit
   ```

2. **Resolve conflicts if any**
   - Prefer upstream changes unless navigator-specific customization
   - Document any conflict resolutions in integration.md

3. **Push to fork**
   ```bash
   git push origin main
   ```

4. **Create fork release tag**
   ```bash
   # Determine tag version: <upstream-version>-nav.<patch>
   # e.g., v0.6.0-nav.1, v0.6.0-nav.2
   VERSION="v0.6.0-nav.1"  # Adjust based on upstream version

   git tag -a "$VERSION" -m "Navigator fork release: synced with upstream

   Upstream: vercel-labs/agent-browser <upstream-tag>
   Navigator tracking: <issue-url>"

   git push origin "$VERSION"
   ```

5. **Update navigator's package.json**
   ```bash
   cd /path/to/navigator  # Back to navigator root

   # Update packages/server/package.json to reference the tag:
   # "@outfitter/agent-browser": "github:outfitter-dev/agent-browser#v0.6.0-nav.1"
   ```

6. **Force refresh bun lockfile** (required for GitHub deps)
   ```bash
   cd /path/to/navigator
   rm bun.lock
   bun install
   ```

   > **Why?** Bun caches GitHub commit SHAs. Without removing the lockfile,
   > `bun install` may not fetch the new tag even after pushing.

7. **Verify lockfile updated**
   ```bash
   grep "agent-browser" bun.lock
   # Should show the new tag: #v0.6.0-nav.1
   ```

8. **Run navigator tests**
   ```bash
   bun run typecheck
   bun test
   ```

9. **Update integration docs**
   - Mark fork synced in `docs/_upstream/<version>/integration.md`
   - Add tag version to the doc

### Decision Point

If tests fail → **STOP** and document failures, do not commit

---

## Fork Tagging Convention

The fork uses `<upstream-version>-nav.<patch>` tags:

| Tag | Meaning |
|-----|---------|
| `v0.6.0-nav.1` | First navigator release based on upstream v0.6.0 |
| `v0.6.0-nav.2` | Second navigator release (e.g., hotfix to fork) |
| `v0.7.0-nav.1` | First navigator release based on upstream v0.7.0 |

Benefits:
- Clear upstream version lineage
- Navigator can branch and test new versions before merging to main
- Pinnable in package.json: `github:outfitter-dev/agent-browser#v0.6.0-nav.1`

---

## Integration Patterns

### Pattern: API Signature Change

When upstream changes a function signature:

1. Find all navigator usages
2. Update to new signature
3. Add backward-compat wrapper if needed (temporary)
4. Document in integration.md

### Pattern: New Feature Adoption

When upstream adds a useful feature:

1. Evaluate if navigator should expose it
2. Add to navigator's schema if needed
3. Wire through action-executor
4. Add tests
5. Document in navigator CHANGELOG

### Pattern: Breaking Type Change

When upstream changes a type definition:

1. Update navigator's re-exports
2. Check all type usages compile
3. Update any Zod schemas that reference it

---

## Quick Reference

### Commands

```bash
# Run analysis (auto-clones if needed)
bun run .claude/skills/agent-browser-upstream/scripts/analyze-upstream.ts --format summary

# Generate diff artifacts (for incremental context loading)
bun run .claude/skills/agent-browser-upstream/scripts/generate-diff.ts

# Check current versions
cd .agent-browser/repo && git describe --tags origin/main upstream/main

# View pending changes
cd .agent-browser/repo && git log --oneline origin/main..upstream/main

# Diff specific file
cd .agent-browser/repo && git diff origin/main..upstream/main -- src/protocol.ts

# Read analysis artifacts incrementally
cat .agent-browser/analysis/<sha>/summary.json        # Start here
jq '.counts' .agent-browser/analysis/<sha>/summary.json
cat .agent-browser/analysis/<sha>/by-category/breaking.json
```

### Key Files

| Location | Purpose |
|----------|---------|
| `.agent-browser/repo/` | Local clone of the fork (gitignored) |
| `.agent-browser/analysis/<sha>/` | Generated artifacts for specific upstream SHA |
| `docs/_upstream/README.md` | Index of all integration docs |
| `docs/_upstream/<version>/integration.md` | Version-specific integration plan |
| `references/integration-template.md` | Template for integration docs (also the issue) |
| `scripts/generate-diff.ts` | Generates jq-able artifacts for agents |
| `scripts/analyze-upstream.ts` | Analyzes commits between refs |
| `scripts/create-issue.ts` | Creates GitHub issue from integration doc |

### Environment Variables

| Variable | Purpose | Default |
|----------|---------|---------|
| `AGENT_BROWSER_LOCAL` | Override repo path | `.agent-browser/repo/` in navigator root |

---

## Troubleshooting

### Bun Lockfile Not Updating

**Symptom:** After pushing new tag to fork, `bun install` still shows old commit SHA.

**Cause:** Bun caches GitHub dependencies by commit SHA. Even with a new tag, it may reuse the cached resolution.

**Fix:**
```bash
cd /path/to/navigator
rm bun.lock
bun install
grep "agent-browser" bun.lock  # Verify new tag/commit
```

### Git Context Issues

**Symptom:** `gh` commands target wrong repo (e.g., upstream instead of navigator).

**Cause:** Working directory is `.agent-browser/repo/` (the fork clone) instead of navigator root.

**Fix:** Always use absolute paths:
```bash
# WRONG - relative path, may be in wrong directory
cd .agent-browser/repo && git push

# RIGHT - absolute path
cd /Users/you/project/navigator/.agent-browser/repo && git push

# For navigator commands, go back to root
cd /Users/you/project/navigator && gh issue create
```

### Tag Already Exists

**Symptom:** `git tag` fails with "tag already exists".

**Fix:** Increment the patch number:
```bash
# If v0.6.0-nav.1 exists, use v0.6.0-nav.2
git tag -a "v0.6.0-nav.2" -m "..."
```

### Fork Behind After Merge

**Symptom:** `git log origin/main..upstream/main` still shows commits after merge.

**Cause:** Origin wasn't pushed.

**Fix:**
```bash
cd .agent-browser/repo
git push origin main
git log --oneline origin/main..upstream/main  # Should be empty
```
