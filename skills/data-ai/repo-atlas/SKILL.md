---
name: repo-atlas
description: Build a self-contained persistent context system (atlas) for any repository. Use when asked to create a repo map, generate codebase documentation for LLM agents, set up an atlas, or create onboarding docs for a codebase. Also use when asked to "map this repo", "document this codebase", or "create context docs".
---

# Repo Atlas

Build an in-repo persistent context system so engineers and LLM agents can understand any codebase quickly with minimal searching.

## Hard Constraints

- Do NOT change product/runtime behavior
- No paid/hosted tooling — everything lives in the repo
- Zero or minimal dependencies (Python 3 standard library only)
- All generated content must reflect real repo specifics, not generic filler

## Workflow

### Phase 1: Reconnaissance

Before writing anything, understand the repo:

1. Read the top-level directory structure
2. Identify the repo type:
   - **App** (web, mobile, desktop) — has screens/views, state managers, routes
   - **Backend/API** — has controllers, routes, middleware, models
   - **Library** — has public exports, module structure, build config
   - **Monorepo** — multiple packages/services
   - **CLI tool** — has command handlers, argument parsing
   - **Infrastructure** — has deployment configs, IaC files
3. Identify the primary language(s) and framework(s)
4. Find entrypoints, build configs, CI files
5. Read 5-10 key files to understand architecture patterns

### Phase 2: Run the Generator Script

Copy `scripts/generate_atlas.py` (bundled with this skill) to the repo at `scripts/atlas/generate_atlas.py`. Then customize and run it:

1. Copy the script to the target repo
2. Review and adjust the configuration section at the top:
   - `IGNORE_NAMES` — add repo-specific directory names to ignore (exact segment match)
   - `TREE_ANNOTATIONS` — add short descriptions for key directories
   - `ENTRYPOINT_NAMES` — add framework-specific entry filenames
   - `ENTRYPOINT_PATH_PATTERNS` — add path-based patterns (fnmatch style, e.g., `cmd/*/main.go`)
   - `ENTRYPOINT_CONTENT_MARKERS` — add code markers that identify entry points
   - `CONVENTIONAL_COMMITS` — adjust if repo uses different commit conventions
   - `CHANGELOG_DAYS` — change the changelog lookback window (default: 14)
3. Run: `python3 scripts/atlas/generate_atlas.py --write`

This auto-generates:
- `docs/atlas/repo-map.md` — directory tree + entrypoints + file stats
- `docs/atlas/08_CHANGELOG_LAST_14_DAYS.md` — categorized recent commits

### Phase 3: Enhance repo-map.md

After the script generates the skeleton, manually add these sections to `repo-map.md`:

**Router Table** — "Where to look for X" (10-15 rows):
```markdown
## Where to Look for X

| Task | Start Here |
|------|-----------|
| Fix [domain concept] | `path/to/file.ext` |
```

Map the top 10-15 tasks someone would do in this repo to specific files.

**Danger Zones** — fragile files/areas:
```markdown
## Danger Zones

| File/Area | Why It's Fragile |
|-----------|-----------------|
| `path/to/file` | Reason |
```

### Phase 4: Write Manual Atlas Docs

Create `docs/atlas/` with these files. See `references/atlas-templates.md` for structure guidance on each.

| File | Content Source |
|------|--------------|
| `00_README.md` | How to use the atlas + agent workflow conventions |
| `01_ARCHITECTURE.md` | Read entrypoints, DI setup, module boundaries |
| `02_DOMAIN_MODEL.md` | Read models/types, identify state machines |
| `03_CRITICAL_FLOWS.md` | Trace top 3-5 user flows through the code |
| `04_STATE_SOURCES_OF_TRUTH.md` | Identify all state stores (DB, cache, files, memory) |
| `05_EXTERNAL_DEPENDENCIES.md` | Read package manifests + integration code |
| `06_GOTCHAS.md` | Look for race conditions, init ordering, fragile patterns |
| `07_TEST_MATRIX.md` | Read test configs, describe how to run tests |

Each doc should be 50-150 lines with real paths, real code references, and real gotchas from the codebase. Not generic advice.

### Phase 5: Add Agent On-Ramp

Add an atlas section to the repo's `CLAUDE.md` (or create one). Include:

```markdown
## Atlas — Persistent Context System

The `docs/atlas/` folder contains structured documentation for fast codebase onboarding.

### Agent Workflow

**Agent A (Plan + Execute)**:
1. Load `docs/atlas/repo-map.md` for orientation
2. Load the domain-specific atlas doc for your task
3. Read source files only after the atlas narrows your search
4. Implement changes following the patterns in the atlas

**Agent B (Verify)**:
1. Review diffs against `docs/atlas/06_GOTCHAS.md`
2. Verify changes match the flow described in `03_CRITICAL_FLOWS.md`
3. Confirm tests pass per `07_TEST_MATRIX.md`
4. Check state consistency against `04_STATE_SOURCES_OF_TRUTH.md`

### Working Rules
- **Analysis first**: Read the relevant atlas docs before writing code
- **Verify behavior**: After changes, confirm critical flows still work
- **No test-cheating**: Tests must pass because the code is correct
- **Update atlas**: If changes alter architecture/flows/state, update the relevant doc
- **Regenerate**: Run `make atlas-generate` after structural changes
```

### Phase 6: Add Build Targets

Add to `Makefile` (create if needed):

```makefile
atlas-generate:
	python3 scripts/atlas/generate_atlas.py --write

atlas-check:
	python3 scripts/atlas/generate_atlas.py --check
```

If the repo uses `package.json`, also add to scripts:
```json
"atlas:generate": "python3 scripts/atlas/generate_atlas.py --write",
"atlas:check": "python3 scripts/atlas/generate_atlas.py --check"
```

### Phase 7: Verify

1. Run `atlas-generate` — must complete without errors
2. Run `atlas-check` — must exit 0 immediately after generation
3. Confirm every atlas doc has real file paths and repo-specific content
4. Confirm no runtime/product code was changed

## Output Summary

After completing all phases, report:
- List of created files
- How to run atlas generation/check
- 10-line "How an agent should use this atlas" quick reference

## Resources

- **Generator script**: See `scripts/generate_atlas.py` — copy to target repo and customize
- **Doc templates**: See `references/atlas-templates.md` for structure guidance on each manual doc
