---
name: run-full-release
description: "Run the current repo's mise release pipeline with auto-scaffolding. Handles GitHub releases (mandatory), PyPI publishing (if Python), and crates.io (if Rust). TRIGGERS - mise release, full release, version bump, release automation, mise run release."
allowed-tools: Read, Bash, Glob, Grep, Write, Edit, AskUserQuestion
argument-hint: "[--dry] [--status]"
model: haiku
disable-model-invocation: true
---

# /mise:run-full-release

Run the current repo's mise release pipeline end-to-end with auto-scaffolding. **Automatically detects and handles**:

- ✅ **Mandatory**: GitHub releases + version tags via semantic-release
- 🐍 **Optional**: PyPI publishing (if `pyproject.toml` + `scripts/publish-to-pypi.sh` or `[tool.maturin]`)
- 🦀 **Optional**: Crates.io publishing (if `Cargo.toml` + `[workspace.package]`)

If no release tasks exist, audits the repo and scaffolds idiomatic release tasks first.

## Step 1: Detect Release Tasks

```bash
mise tasks ls 2>/dev/null | grep -i release
```

## Step 2: Branch Based on Detection

### If release tasks FOUND → Execute

1. Check working directory cleanliness: `git status --porcelain`
2. **Check for unpushed commits:** `git log --oneline @{u}..HEAD 2>/dev/null`
   - If unpushed commits exist → `git push origin main` before proceeding
   - semantic-release needs all commits pushed to analyze and version correctly
3. **Reset lockfile drift** (caused by preflight commands like `uv run pytest`):
   - Check for modified lockfiles: `git diff --name-only | grep -E '(uv\.lock|package-lock\.json|Cargo\.lock|bun\.lockb|yarn\.lock|pnpm-lock\.yaml)$'`
   - If ONLY lockfiles are dirty and no other changes exist → `git checkout -- <lockfile>` to reset them
   - If lockfiles are dirty alongside intentional changes → reset lockfiles first, then handle remaining changes in step 4
   - **Rationale**: Lockfiles modified by `uv run`, `npm install`, etc. during preflight are artifacts, not intentional changes. They should never block or pollute a release.
4. **If working directory is dirty → Autonomously resolve ALL changes before releasing:**
   a. Run `git status --porcelain` and `git diff` to understand every pending change
   b. For each group of related changes:
   - Read the changed files to understand what was modified and why
   - Craft a conventional commit message (`fix:`, `feat:`, `chore:`, `docs:`) that accurately describes the change
   - Stage specific files (never `git add -A`) and commit
     c. For untracked files that should NOT be committed (e.g., work-in-progress from other branches):
   - Stash them: `git stash push -u -m "pre-release: description"`
     d. Verify working directory is clean: `git status --porcelain` should be empty
     e. Restore any stash after release: `git stash pop`

   **Commit guidelines:**
   - Group logically related files into a single commit
   - Use the repo's existing commit message style (check `git log --oneline -5`)
   - Never skip pre-commit hooks (`--no-verify`)
   - If unsure whether a change should be committed or stashed, review the file contents and decide based on whether it's a completed change or work-in-progress

5. **Post-release lockfile cleanup**: After release completes, check again for lockfile drift and reset:
   - `git diff --name-only | grep -E '(uv\.lock|package-lock\.json|Cargo\.lock|bun\.lockb|yarn\.lock|pnpm-lock\.yaml)$' | xargs -r git checkout --`
   - This catches lockfiles modified by release tasks themselves (e.g., version bumps triggering lockfile updates)

6. **Detect optional publishing capabilities:**

   Before executing release, check for PyPI and crates.io:

   ```bash
   # PyPI detection: check for publish-to-pypi.sh OR pyproject.toml
   HAS_PYPI=false
   if [[ -x "scripts/publish-to-pypi.sh" ]] || grep -q '^\[tool\.maturin\]\|^\[project\]' pyproject.toml 2>/dev/null; then
       HAS_PYPI=true
   fi

   # Crates.io detection: check for Cargo.toml with [workspace.package]
   HAS_CRATES=false
   if [[ -f "Cargo.toml" ]] && grep -q '^\[workspace\.package\]' Cargo.toml 2>/dev/null; then
       HAS_CRATES=true
   fi

   export HAS_PYPI HAS_CRATES
   ```

7. Route by flags:
   - `--dry` → `mise run release:dry`
   - `--status` → `mise run release:status`
   - No flags → `mise run release:full` (which includes release:pypi and/or release:crates if detected)

### If release tasks NOT FOUND → Audit & Scaffold

Conduct a thorough audit of the repository to scaffold idiomatic release tasks.

#### Audit Checklist

Run these checks to understand the repo's release needs:

```bash
# 1. Detect language/ecosystem
ls pyproject.toml Cargo.toml package.json setup.py setup.cfg 2>/dev/null

# 2. Detect existing mise config
ls .mise.toml mise.toml 2>/dev/null
cat .mise.toml 2>/dev/null | head -50

# 3. Detect existing release infrastructure
ls .releaserc.yml .releaserc.json .releaserc release.config.* 2>/dev/null
ls .github/workflows/*release* 2>/dev/null
ls Makefile 2>/dev/null && grep -i release Makefile 2>/dev/null

# 4. Detect credential patterns
grep -r "GH_TOKEN\|GITHUB_TOKEN\|UV_PUBLISH_TOKEN\|CARGO_REGISTRY_TOKEN\|NPM_TOKEN" .mise.toml mise.toml 2>/dev/null

# 5. Detect build requirements
grep -i "maturin\|zig\|cross\|docker\|wheel\|sdist" .mise.toml Cargo.toml pyproject.toml 2>/dev/null
```

#### Read Reference Templates

Read these files from the cc-skills marketplace for the canonical 4-phase release pattern:

```
Read: $HOME/.claude/plugins/marketplaces/cc-skills/docs/RELEASE.md
```

Also examine cc-skills' own release tasks as a working template:

```bash
ls $HOME/.claude/plugins/marketplaces/cc-skills/.mise/tasks/release/
```

#### Scaffold `.mise/tasks/release/`

Create the release task directory and files customized to THIS repo:

| Task        | Always                        | Repo-Specific Additions                                     |
| ----------- | ----------------------------- | ----------------------------------------------------------- |
| `_default`  | Help/navigation               | —                                                           |
| `preflight` | Clean dir, auth, branch check | Plugin validation, build tool checks                        |
| `version`   | semantic-release              | Repo-specific `.releaserc.yml` plugins                      |
| `sync`      | Git push                      | PyPI publish (if exists), crates.io publish (if Rust), sync |
| `pypi`      | (Optional)                    | `scripts/publish-to-pypi.sh` via `uv publish` or `twine`    |
| `crates`    | (Optional)                    | `cargo publish --workspace` (Rust 1.90+, native ordering)   |
| `verify`    | Tag + release check           | Verify artifacts (wheels, packages, published versions)     |
| `full`      | Orchestrator (`depends`)      | Include all repo-specific phases                            |
| `dry`       | `semantic-release --dry-run`  | —                                                           |
| `status`    | Current version info          | —                                                           |

**Publishing Capability Detection:**

Before running release:

1. **PyPI**: Check if `scripts/publish-to-pypi.sh` exists OR `pyproject.toml` contains `[tool.maturin]` or `[project]` with name/version
   - If found → include `release:pypi` in `release:full` depends chain
   - Store as `HAS_PYPI=true` for conditional task execution

2. **Crates.io**: Check if `Cargo.toml` exists AND contains `[workspace.package]` with version
   - If found → include `release:crates` in `release:full` depends chain
   - Store as `HAS_CRATES=true` for conditional task execution

3. **GitHub Releases**: Mandatory (via `@semantic-release/github`)

#### Ensure SSoT via mise

- All credentials must be in `.mise.toml` `[env]` section (not hardcoded in scripts)
- All tool versions must be in `[tools]` section
- All thresholds/configs as env vars with fallback defaults
- Use `read_file()` template function for secrets (e.g., `GH_TOKEN`)

#### Task Orchestration (release:full)

The `release:full` task **must** use conditional task dependencies to handle optional PyPI/crates.io:

```bash
#!/usr/bin/env bash
#MISE description="Phase 4: Full release orchestration with conditional publishing"
#MISE depends=["release:preflight"]

set -euo pipefail

# Detect publishing capabilities
HAS_PYPI=false
if [[ -x "scripts/publish-to-pypi.sh" ]] || grep -q '^\[tool\.maturin\]\|^\[project\]' pyproject.toml 2>/dev/null; then
    HAS_PYPI=true
fi

HAS_CRATES=false
if [[ -f "Cargo.toml" ]] && grep -q '^\[workspace\.package\]' Cargo.toml 2>/dev/null; then
    HAS_CRATES=true
fi

# Phase 1: Version bump
echo "→ Phase 1: Versioning..."
mise run release:version

# Phase 2: Sync to main + conditional publishing
echo "→ Phase 2: Syncing..."
git push --follow-tags origin main

# Phase 2b: PyPI publishing (optional)
if [[ "$HAS_PYPI" == "true" ]]; then
    echo "→ Phase 2b: Publishing to PyPI..."
    if mise tasks list | grep -q 'release:pypi'; then
        mise run release:pypi || echo "⚠ PyPI publish failed (non-fatal)"
    else
        echo "⚠ release:pypi task not found (skipping)"
    fi
fi

# Phase 2c: Crates.io publishing (optional)
if [[ "$HAS_CRATES" == "true" ]]; then
    echo "→ Phase 2c: Publishing to crates.io..."
    if mise tasks list | grep -q 'release:crates'; then
        mise run release:crates || echo "⚠ Crates.io publish failed (non-fatal)"
    else
        echo "⚠ release:crates task not found (skipping)"
    fi
fi

# Phase 3: Verify
echo "→ Phase 3: Verifying..."
mise run release:verify

echo ""
echo "✓ Release complete!"
echo ""
```

**Publishing Flags**:

- Non-fatal: If PyPI or crates.io publishing fails, release continues (version tag still created)
- Verify phase reports which artifacts published successfully

#### After Scaffolding

Run `mise run release:full` with the newly created tasks.

## Publishing Task Implementation

### `release:pypi` (Optional - Only if Python Package)

**Triggers**: `pyproject.toml` exists AND (`scripts/publish-to-pypi.sh` exists OR `[tool.maturin]` present)

**Implementation**:

```bash
#!/usr/bin/env bash
#MISE description="Phase 2b: Publish to PyPI via uv publish (pure Python) or twine (maturin wheels)"
set -euo pipefail

if [[ -x "scripts/publish-to-pypi.sh" ]]; then
    # Use custom script (handles maturin, 1Password tokens, service accounts, etc.)
    ./scripts/publish-to-pypi.sh
elif grep -q '\[tool\.maturin\]' pyproject.toml; then
    # Maturin project: wheels built by release:build-all
    echo "Publishing maturin wheels to PyPI..."
    # Credentials sourced from .mise.toml [env] section
    # Implementation depends on maturin/uv/twine setup
    echo "ERROR: release:pypi requires scripts/publish-to-pypi.sh"
    exit 1
else
    # Pure Python: use uv publish with UV_PUBLISH_TOKEN
    echo "Publishing pure Python package to PyPI..."
    UV_PUBLISH_TOKEN="${UV_PUBLISH_TOKEN:-}" uv publish || {
        echo "⚠ uv publish failed - set UV_PUBLISH_TOKEN in .mise.toml [env]"
        return 1
    }
fi
```

**Credentials (via `.mise.toml [env]`)**:

```toml
[env]
# PyPI token (supports both uv and twine)
UV_PUBLISH_TOKEN = "{{ read_file(path=env.HOME ~ '/.claude/.secrets/pypi-token') | trim }}"

# For 1Password service account (alternative):
# OP_SERVICE_ACCOUNT_TOKEN = "{{ read_file(path=env.HOME ~ '/.claude/.secrets/op-service-account-token') | trim }}"
```

**Post-Publish Verification**:

In `release:verify`, add:

```bash
# Check PyPI availability
PACKAGE_NAME=$(grep '^name = ' pyproject.toml | sed 's/name = "\(.*\)"/\1/' | head -1)
CURRENT_VERSION=$(grep '^version = ' pyproject.toml | sed 's/version = "\(.*\)"/\1/' | head -1)

echo "Checking PyPI for ${PACKAGE_NAME} v${CURRENT_VERSION}..."
if curl -s "https://pypi.org/pypi/${PACKAGE_NAME}/${CURRENT_VERSION}/json" | grep -q "version"; then
    echo "✓ Published to PyPI"
else
    echo "⚠ Still propagating to PyPI (check in 30 seconds)"
fi
```

### `release:crates` (Optional - Only if Rust Workspace)

**Triggers**: `Cargo.toml` exists AND `[workspace.package]` present AND `rust-version >= 1.90`

**Use native `cargo publish --workspace`** (stabilized in Rust 1.90, Sept 2025). This single command:

- Auto-discovers all publishable crates (skips `publish = false`)
- Topologically sorts by dependency order
- Pre-validates the entire workspace builds correctly before publishing any crate
- Handles crates.io index propagation between dependent publishes

**Never** hardcode crate lists, iterate `crates/*/` in filesystem order, or write bespoke topological sort scripts. All of these are superseded by native Cargo support.

**Implementation**:

```bash
#!/usr/bin/env bash
#MISE description="Phase 2c: Publish Rust crates to crates.io (native workspace publish)"
set -euo pipefail

# Requires CARGO_REGISTRY_TOKEN in .mise.toml [env]
if [[ -z "${CARGO_REGISTRY_TOKEN:-}" ]]; then
    echo "ERROR: CARGO_REGISTRY_TOKEN not set in .mise.toml [env]"
    exit 1
fi

# Native workspace publish (Rust 1.90+):
# - Auto-discovers publishable crates (skips publish=false)
# - Topologically sorts by dependency order
# - Pre-validates entire workspace builds before publishing any crate
cargo publish --workspace

echo "✓ Crates.io publishing complete"
```

**Preflight Gate** (add to `release:preflight`):

```bash
# Verify all publishable crates can package before starting the release
cargo publish --workspace --dry-run
```

**Credentials (via `.mise.toml [env]`)**:

```toml
[env]
# Crates.io token
CARGO_REGISTRY_TOKEN = "{{ read_file(path=env.HOME ~ '/.claude/.secrets/crates-io-token') | trim }}"
```

**Post-Publish Verification**:

In `release:verify`, add:

```bash
# Verify all published crates are available on crates.io
CRATES=$(cargo metadata --no-deps --format-version 1 | \
  jq -r '.packages[] | select(.source == null) | select(.publish == null) | select(.name | endswith("-py") | not) | .name')
for crate in $CRATES; do
    echo "Checking crates.io for ${crate} v${CRATE_VERSION}..."
    if curl -s "https://crates.io/api/v1/crates/${crate}/${CRATE_VERSION}" | grep -q "version"; then
        echo "✓ ${crate} published"
    else
        echo "⚠ ${crate} still propagating (check in 30 seconds)"
    fi
done
```

## Error Recovery

| Error                                  | Resolution                                                               |
| -------------------------------------- | ------------------------------------------------------------------------ |
| `mise` not found                       | Install: `curl https://mise.run \| sh`                                   |
| No release tasks                       | Scaffold using audit above                                               |
| Working dir not clean                  | Review, commit, or stash all changes autonomously                        |
| Lockfile drift (uv.lock etc.)          | `git checkout -- uv.lock` (artifact, not intentional)                    |
| Unpushed commits                       | `git push origin main` before release                                    |
| Not on main branch                     | `git checkout main`                                                      |
| No releasable commits                  | Create a `feat:` or `fix:` commit first                                  |
| Missing GH_TOKEN                       | Add to `.mise.toml` `[env]` section                                      |
| semantic-release not configured        | Create `.releaserc.yml` (see cc-skills reference)                        |
| **PyPI-Specific Errors**               |                                                                          |
| `UV_PUBLISH_TOKEN` not set             | Add to `.mise.toml` [env]; store token in `~/.claude/.secrets/`          |
| `scripts/publish-to-pypi.sh` not found | Create using template (see Publishing Task Implementation above)         |
| `twine upload` 403 Forbidden           | Check PyPI token permissions (must be account-wide, not project)         |
| Package already exists on PyPI         | Non-fatal; release continues (tag still created on GitHub)               |
| **Crates.io-Specific Errors**          |                                                                          |
| `CARGO_REGISTRY_TOKEN` not set         | Add to `.mise.toml` [env]; get token from <https://crates.io/me>         |
| `cargo publish` timeout                | Retry with `mise run release:crates` (non-fatal, tag already set)        |
| Crate already published on crates.io   | Non-fatal; check version in `Cargo.toml` for next release                |
| Workspace publish order error          | Use `cargo publish --workspace` (Rust 1.90+) — handles ordering natively |
| Missing crate on crates.io             | Check `publish = false` — crate may need publishing or its dep does      |
