---
name: release
description: Standardized release workflow with version updates and CHANGELOG management
---

# /release - Release Preparation Skill

Automates the pre-release preparation workflow, ensuring consistent version updates across all files and proper CHANGELOG management.

## Language Requirement

**IMPORTANT**: All outputs (commits, PRs, branch names) MUST be written in **English**.

## Scope Boundaries

### What this skill DOES:
- Update version numbers in all required files
- Update CHANGELOG.md format
- Run pre-flight checks (fmt, clippy, test)
- Create release branch and PR to main

### What this skill does NOT do (manual steps):
- **Tag creation** - User must manually create and push the tag after PR merge
- **PR merge** - User must manually review and merge the PR
- **Post-release verification** - User must verify the release was successful

This separation ensures human oversight for the irreversible release action (tag push triggers CI/CD).

## Pre-flight Checks (MANDATORY)

Before proceeding with version updates, ALL of the following checks MUST pass:

### 1. Format Code

```bash
cargo fmt --all
```

### 2. Clippy Check

```bash
cargo clippy --all-targets --all-features -- -D warnings
```

**CRITICAL**: Zero warnings required. Fix all issues before proceeding.

### 3. Test Suite

```bash
cargo test --all
```

All tests must pass.

## Steps

### Step 0: Validate Current Branch (MANDATORY)

```bash
git branch --show-current
```

**CRITICAL**: Must be on `develop` or a feature branch. Cannot release from `main`.

| Current Branch | Action |
|----------------|--------|
| `main` | ❌ **STOP** - Cannot release from main |
| `develop` | ✅ Proceed |
| `feature/*` | ✅ Proceed |
| `bugfix/*` | ✅ Proceed |

If on `main`:
```
⚠️ ERROR: Cannot start release from 'main' branch.
Please checkout 'develop' or a feature branch first.
```

### Step 1: Get Target Version from User

Ask the user for the target version number (e.g., "1.1.0").

**Version Format Validation**:
- Must be valid semver: `X.Y.Z` where X, Y, Z are non-negative integers
- Examples: `1.0.0`, `1.1.0`, `2.0.0-beta.1`
- Invalid: `v1.0.0` (no 'v' prefix), `1.0` (missing patch)

### Step 2: Run Pre-flight Checks

Execute ALL pre-flight checks:

```bash
cargo fmt --all
cargo clippy --all-targets --all-features -- -D warnings
cargo test --all
```

If any check fails:
1. Fix the issues
2. Re-run all checks
3. Only proceed when ALL pass

### Step 3: Update Version Files

Update version in exactly these 3 files:

| File | Pattern | Example |
|------|---------|---------|
| `Cargo.toml` | `version = "X.Y.Z"` | `version = "1.1.0"` |
| `python-wrapper/pyproject.toml` | `version = "X.Y.Z"` | `version = "1.1.0"` |
| `python-wrapper/uv_sbom_bin/install.py` | `UV_SBOM_VERSION = "X.Y.Z"` | `UV_SBOM_VERSION = "1.1.0"` |

### Step 4: Update CHANGELOG.md

**Before**:
```markdown
## [Unreleased]

### Added
- New feature
```

**After**:
```markdown
## [Unreleased]

## [X.Y.Z] - YYYY-MM-DD

### Added
- New feature
```

- Replace `## [Unreleased]` section header with version and date
- Add new empty `## [Unreleased]` section above the new version
- Date format: `YYYY-MM-DD` (e.g., `2025-02-06`)

### Step 5: Verify Version Consistency

```bash
# Check all versions match
grep 'version = "' Cargo.toml | head -1
grep 'version = "' python-wrapper/pyproject.toml
grep 'UV_SBOM_VERSION = "' python-wrapper/uv_sbom_bin/install.py
```

All three must show the same version. If not, stop and fix.

### Step 6: Create Release Branch

```bash
git checkout -b release/vX.Y.Z
```

Branch naming: `release/v{version}` (e.g., `release/v1.1.0`)

### Step 7: Commit Changes

Stage and commit all version changes using `/commit` skill conventions:

```bash
git add Cargo.toml python-wrapper/pyproject.toml python-wrapper/uv_sbom_bin/install.py CHANGELOG.md
```

Commit message format:
```
chore(release): prepare v{version}

- Update version to {version} in all files
- Update CHANGELOG with release date

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Step 8: Create Pull Request

Create PR to `main` using `/pr` skill:

```bash
gh pr create --base main --title "chore(release): prepare v{version}" --body "$(cat <<'EOF'
## Summary
- Prepare release v{version}
- Update version numbers in all required files
- Update CHANGELOG with release date

## Version Files Updated
- `Cargo.toml`: version = "{version}"
- `python-wrapper/pyproject.toml`: version = "{version}"
- `python-wrapper/uv_sbom_bin/install.py`: UV_SBOM_VERSION = "{version}"

## CHANGELOG
- Converted [Unreleased] to [{version}] - {date}
- Added empty [Unreleased] section

## Test Plan
- [x] `cargo fmt --all -- --check` passes
- [x] `cargo clippy --all-targets --all-features -- -D warnings` passes
- [x] `cargo test --all` passes
- [x] All version strings are consistent

## Post-Merge Manual Steps
After merging this PR:
1. Create tag: `git tag v{version}`
2. Push tag: `git push origin v{version}`
3. Verify CI release workflow completes successfully

---
Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

**IMPORTANT**: PR base branch is `main`, not `develop`.

### Step 9: Output Manual Next Steps

After PR creation, display the following instructions:

```
✅ Release PR created successfully!

📋 Manual steps after PR is merged:

1. Wait for CI to pass and get approval
2. Merge the PR to main
3. Switch to main and pull:
   git checkout main
   git pull origin main

4. Create and push the tag:
   git tag v{version}
   git push origin v{version}

5. Verify the release:
   - Check GitHub Actions release workflow
   - Verify GitHub Release was created
   - Check crates.io publication
   - Check PyPI publication

6. Merge main back to develop:
   git checkout develop
   git merge main
   git push origin develop
```

## Error Handling

### Invalid Version Format

If version doesn't match semver:
```
⚠️ ERROR: Invalid version format: '{input}'
Version must be in semver format: X.Y.Z (e.g., 1.1.0)
```

### Pre-flight Check Failures

If any pre-flight check fails:
1. Display the error
2. Fix the issues
3. Re-run ALL checks
4. Only proceed when all pass

### Version Mismatch After Update

If versions don't match after update:
```
⚠️ ERROR: Version mismatch detected!

Cargo.toml: {version1}
pyproject.toml: {version2}
install.py: {version3}

Please fix manually and retry.
```

### CHANGELOG Already Has Version

If CHANGELOG already contains the target version:
```
⚠️ WARNING: CHANGELOG.md already contains [{version}]
This version may have already been released.
Do you want to continue anyway? (y/N)
```

## Example Usage

User: "/release 1.1.0" or "リリース 1.1.0"

Claude executes /release skill:
1. Validates current branch (develop) ✓
2. Validates version format (1.1.0) ✓
3. Runs pre-flight checks (fmt, clippy, test) ✓
4. Updates Cargo.toml version
5. Updates pyproject.toml version
6. Updates install.py UV_SBOM_VERSION
7. Updates CHANGELOG.md
8. Verifies all versions match ✓
9. Creates branch `release/v1.1.0`
10. Commits: "chore(release): prepare v1.1.0"
11. Creates PR to main
12. Outputs manual next steps

Final output:
```
✅ Release preparation complete!
PR: https://github.com/Taketo-Yoda/uv-sbom/pull/XXX

📋 After PR merge, run:
   git tag v1.1.0
   git push origin v1.1.0
```
