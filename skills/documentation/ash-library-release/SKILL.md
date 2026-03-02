---
name: ash-library-release
description: Manages version releases for ash_cookie_consent Elixir library including CHANGELOG updates, version bumping, quality checks, Hex.pm publishing, and git tagging. Use when user asks to "release", "publish to hex", "bump version", "create release", or "prepare for publishing".
---

# AshCookieConsent Library Release Management

This skill guides you through releasing a new version of the ash_cookie_consent library to Hex.pm.

## When to Use This Skill

Activate this skill when the user requests:
- "Release version X.Y.Z"
- "Publish to Hex.pm"
- "Prepare for release"
- "Bump version to X.Y.Z"
- "Create a new release"

## Prerequisites Check

Before starting, verify:

1. **Clean working directory**:
   ```bash
   git status
   ```
   All changes should be committed.

2. **On main branch and up to date**:
   ```bash
   git branch --show-current  # Should show: main
   git pull origin main
   ```

3. **User has Hex credentials**:
   ```bash
   mix hex.user whoami
   ```
   If not authenticated, run: `mix hex.user auth`

## Release Workflow

### Step 1: Determine Version Number

Ask user for version number if not provided. Follow Semantic Versioning:

- **MAJOR** (X.0.0): Breaking changes, incompatible API changes
- **MINOR** (0.X.0): New features, backward compatible
- **PATCH** (0.0.X): Bug fixes, backward compatible

**Pre-1.0 note**: While version < 1.0.0, MINOR can include breaking changes.

### Step 2: Update CHANGELOG.md

1. Read current CHANGELOG.md
2. Move all items from `[Unreleased]` section to new version section:
   ```markdown
   ## [X.Y.Z] - YYYY-MM-DD

   ### Added
   - Feature descriptions from [Unreleased]

   ### Changed
   - Changes from [Unreleased]

   ### Fixed
   - Bug fixes from [Unreleased]
   ```

3. Add today's date (YYYY-MM-DD format)
4. Update comparison links at bottom of file
5. Ensure `[Unreleased]` section is empty but present

**Quality check**:
- [ ] All significant changes documented
- [ ] Breaking changes clearly marked
- [ ] User-facing language (not internal jargon)
- [ ] Issue/PR numbers referenced where applicable

### Step 3: Bump Version in mix.exs

1. Read `mix.exs`
2. Update `@version "X.Y.Z"` at top of file
3. Verify no other version references need updating

### Step 4: Run Quality Checks

Execute all quality checks and report results:

```bash
# 1. Tests
mix test

# 2. Formatting
mix format --check-formatted

# 3. Credo (strict mode)
mix credo --strict

# 4. Dialyzer
mix dialyzer

# 5. Documentation build
mix docs
```

**Required**: All must pass with 0 errors/warnings before proceeding.

If any fail:
- Fix issues
- Re-run checks
- Update changelog if fixes are significant

### Step 5: Build and Review Package

```bash
# Build package
mix hex.build
```

Review package contents:
1. Check file list in output
2. Verify size is reasonable (should be ~30-40KB)
3. Confirm no test files included
4. Confirm no .claude directory included
5. List contents if needed: `tar -tf ash_cookie_consent-X.Y.Z.tar`

### Step 6: Commit Version Bump

```bash
git add CHANGELOG.md mix.exs
git commit -m "Bump version to X.Y.Z"
```

**Commit message format**: Always use `Bump version to X.Y.Z` for consistency.

### Step 7: Create and Push Git Tag

```bash
# Create annotated tag
git tag -a vX.Y.Z -m "Release vX.Y.Z"

# Push commits and tag
git push origin main
git push origin vX.Y.Z
```

**Tag format**: Always prefix with `v` (e.g., `v0.2.0`)

### Step 8: Publish to Hex.pm

```bash
mix hex.publish
```

The command will:
1. Show package details
2. Show files to be included
3. Show dependencies
4. Prompt: "Proceed? [Yn]"

**Action**: Inform user to type `y` to confirm.

**After publishing**:
- Note the published version
- Save package checksum if shown

### Step 9: Create GitHub Release

If `gh` CLI is available:

```bash
gh release create vX.Y.Z \
  --title "vX.Y.Z - Brief Description" \
  --notes "See [CHANGELOG.md](https://github.com/shotleybuilder/ash_cookie_consent/blob/main/CHANGELOG.md#vXYZ) for full details."
```

If `gh` not available:
- Provide manual instructions to create release on GitHub
- Include link: `https://github.com/shotleybuilder/ash_cookie_consent/releases/new`

### Step 10: Post-Release Verification

Verify release succeeded:

1. **Hex.pm package**:
   ```bash
   open https://hex.pm/packages/ash_cookie_consent
   ```
   - Verify version shows correctly
   - Check documentation built

2. **GitHub release**:
   ```bash
   open https://github.com/shotleybuilder/ash_cookie_consent/releases
   ```
   - Verify release created
   - Check tag points to correct commit

### Step 11: Post-Release Checklist

Present this checklist to user:

- [ ] Verify package published on Hex.pm
- [ ] Verify documentation built on hexdocs.pm
- [ ] Verify GitHub release created
- [ ] Announce on Elixir Forum (optional)
- [ ] Announce on Ash Discord (optional)
- [ ] Update roadmap/milestones (if applicable)

## Troubleshooting

### "mix hex.publish" fails with authentication error

**Solution**:
```bash
mix hex.user auth
```
Re-authenticate and try again.

### Documentation fails to build

**Solution**:
1. Run `mix docs` locally
2. Fix any errors in module documentation
3. Commit fixes
4. Restart release process from Step 6

### Tests fail during quality checks

**Solution**:
1. Fix failing tests
2. Add changelog entry under `### Fixed` if user-facing
3. Restart release process from Step 2

### Package size is too large

**Solution**:
1. Check `files:` list in `mix.exs` package config
2. Ensure test files, docs, and .claude excluded
3. Rebuild: `mix hex.build`

## Quick Reference: Common Release Types

### Patch Release (Bug Fix)

Example: 0.1.0 → 0.1.1

```markdown
## [0.1.1] - 2025-11-XX

### Fixed
- Fix consent expiration calculation (#42)
- Correct cookie encoding for edge case (#43)
```

### Minor Release (New Features)

Example: 0.1.0 → 0.2.0

```markdown
## [0.2.0] - 2025-XX-XX

### Added
- Built-in database sync helpers
- Multi-language support (EN, ES, FR, DE)

### Changed
- Improved performance of cookie encoding (50% faster)
```

### Major Release (Breaking Changes)

Example: 0.9.0 → 1.0.0

```markdown
## [1.0.0] - 2025-XX-XX

### Added
- Stable API guarantee
- Comprehensive security audit

### Changed
- **BREAKING**: Renamed `consent_given?/2` to `has_consent?/2`
- **BREAKING**: Changed default cookie expiration from 365 to 180 days

### Migration Guide
...
```

## Emergency Hotfix Process

If critical bug needs immediate fix:

1. **Create hotfix branch**:
   ```bash
   git checkout -b hotfix/X.Y.Z vX.Y.Z
   ```

2. **Fix the bug**:
   - Make minimal changes
   - Add regression test
   - Update CHANGELOG under `### Fixed`

3. **Bump PATCH version**:
   - X.Y.Z → X.Y.Z+1

4. **Test thoroughly**:
   ```bash
   mix test
   mix credo --strict
   mix dialyzer
   ```

5. **Merge and publish**:
   ```bash
   git checkout main
   git merge hotfix/X.Y.Z+1
   # Then follow normal release process from Step 6
   ```

6. **Retire vulnerable version** (if security issue):
   ```bash
   mix hex.retire ash_cookie_consent X.Y.Z \
     --reason security \
     --message "Security fix in vX.Y.Z+1, please upgrade"
   ```

## Reference Documents

- Full workflow: `.claude/MAINTAINER_GUIDE.md`
- Semantic versioning: https://semver.org
- Hex.pm publishing: https://hex.pm/docs/publish
- Keep a Changelog: https://keepachangelog.com

## Success Criteria

Release is successful when:

- ✅ All quality checks pass (tests, credo, dialyzer, docs)
- ✅ Version bumped in mix.exs
- ✅ CHANGELOG updated with release date
- ✅ Git tag created and pushed
- ✅ Package published on Hex.pm
- ✅ GitHub release created
- ✅ Documentation built on hexdocs.pm
- ✅ No errors reported by users within first 24 hours

## Notes

- Always be conservative with version bumps
- When in doubt about breaking changes, bump MINOR (pre-1.0)
- Document breaking changes thoroughly
- Test in real app before publishing if possible
- Can't unpublish from Hex after 1 hour or if downloaded
