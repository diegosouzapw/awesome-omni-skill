---
name: release
description: Automate release process with Conventional Commits-based version detection. Creates CHANGELOG entry, version bump commit, git tag, and GitHub Release. Auto-detects version from commit types (feat→MINOR, fix→PATCH, BREAKING CHANGE→MAJOR). Usage: /release [version] [--dry-run]
license: MIT
metadata:
  author: kuju63
  version: "1.0.0"
  compatibility: Requires git, gh CLI, yamllint, and internet access
---

# Release Automation Skill

Automate the complete release process for the podman-provider project, from validation to GitHub Release creation.

## Guidelines

### Release Process Overview

This skill automates the 7-phase release process:

1. **Phase 0: Validation and Preparation** - Verify arguments, prerequisites, and commit range
2. **Phase 1: Pre-release Verification** - Run tests and validate provider.yaml
3. **Phase 2: CHANGELOG Generation** - Parse git log and generate CHANGELOG entry
4. **Phase 3: Version Bump Commit** - Update provider.yaml version and create commit
5. **Phase 4: Git Tag Creation** - Create annotated git tag
6. **Phase 5: Push Changes** - Push commits and tags to remote
7. **Phase 6: GitHub Release Creation** - Create GitHub Release with notes
8. **Phase 7: Post-release Verification** - Verify and display results

### Semantic Versioning Rules

When version is not specified (`$ARGUMENTS[0]` is empty), the skill automatically determines the version bump type based on Conventional Commits:

- **MAJOR bump** (e.g., v0.3.0 → v1.0.0): Contains `BREAKING CHANGE:` or `feat!`/`fix!` commits
- **MINOR bump** (e.g., v0.3.0 → v0.4.0): Contains `feat:` commits
- **PATCH bump** (e.g., v0.3.0 → v0.3.1): Contains only `fix:` commits or other types (docs, chore, etc.)

### Dry-run Mode

Use `--dry-run` flag to preview changes without executing commits, tags, or pushes:
- Runs Phase 0-1 (validation and tests)
- Displays CHANGELOG and release notes preview
- Does not modify git state

## Task

Execute the release process for the specified version or auto-detect version from Conventional Commits.

**Arguments**:
- `$ARGUMENTS[0]`: Version number (e.g., `v0.4.0`) - **Optional** (auto-detected if omitted)
- `$ARGUMENTS[1]`: Optional flag `--dry-run` for preview mode

## Steps

### Phase 0: Validation and Preparation

**0.1 Parse and validate arguments**

Check if `$ARGUMENTS[0]` is provided:

**If version is specified** (`$ARGUMENTS[0]` is not empty and not `--dry-run`):
- Validate version format: Must match `^v\d+\.\d+\.\d+$` (e.g., v0.4.0)
- If invalid, display error and exit

**If version is NOT specified** (proceed to auto-detection):

1. **Get current version** from provider.yaml line 2:
   ```bash
   current_version=$(sed -n '2p' provider.yaml | sed 's/version: //')
   ```

2. **Get previous tag**:
   ```bash
   prev_tag=$(git describe --tags --abbrev=0 2>/dev/null || echo "")
   if [ -z "$prev_tag" ]; then
     echo "No previous tag found. Using initial version v0.1.0"
     prev_tag="v0.0.0"  # For first release
   fi
   ```

3. **Get commit range**:
   ```bash
   commits=$(git log ${prev_tag}..HEAD --format="%s")
   commit_count=$(echo "$commits" | wc -l | tr -d ' ')

   if [ "$commit_count" -eq 0 ]; then
     echo "Error: No commits since last release ($prev_tag)"
     exit 1
   fi
   ```

4. **Determine version bump type**:
   ```bash
   if echo "$commits" | grep -qE "BREAKING CHANGE:|^(feat|fix)!:"; then
     bump_type="major"
   elif echo "$commits" | grep -q "^feat:"; then
     bump_type="minor"
   else
     bump_type="patch"
   fi
   ```

5. **Calculate new version**:
   ```bash
   # Parse current version
   IFS='.' read -r major minor patch <<< "${current_version#v}"

   # Apply bump
   case "$bump_type" in
     major)
       major=$((major + 1))
       minor=0
       patch=0
       ;;
     minor)
       minor=$((minor + 1))
       patch=0
       ;;
     patch)
       patch=$((patch + 1))
       ;;
   esac

   new_version="v${major}.${minor}.${patch}"
   ```

6. **User confirmation**:
   ```bash
   echo "=========================================="
   echo "Version Auto-Detection Results"
   echo "=========================================="
   echo "Current version: $current_version"
   echo "Commit range: $prev_tag..HEAD ($commit_count commits)"
   echo "Detected bump type: $bump_type"
   echo "New version: $new_version"
   echo ""
   echo "Sample commits:"
   echo "$commits" | head -5
   if [ "$commit_count" -gt 5 ]; then
     echo "... and $((commit_count - 5)) more commits"
   fi
   echo "=========================================="
   echo ""
   read -p "Proceed with version $new_version? (y/n) " -n 1 -r
   echo
   if [[ ! $REPLY =~ ^[Yy]$ ]]; then
     echo "Release cancelled."
     exit 0
   fi
   ```

After this step, `$new_version` variable contains the version to release.

**0.2 Check for dry-run mode**

```bash
dry_run=false
if [ "$ARGUMENTS[1]" == "--dry-run" ] || [ "$ARGUMENTS[0]" == "--dry-run" ]; then
  dry_run=true
  echo "🔍 DRY-RUN MODE: No commits, tags, or pushes will be executed"
  echo ""
fi
```

**0.3 Verify prerequisites**

Run the following checks:

1. **Working tree is clean**:
   ```bash
   if [ -n "$(git status --porcelain)" ]; then
     echo "Error: Working tree is not clean. Commit or stash changes first."
     git status --short
     exit 1
   fi
   ```

2. **On main branch**:
   ```bash
   current_branch=$(git branch --show-current)
   if [ "$current_branch" != "main" ]; then
     echo "Error: Not on main branch. Current branch: $current_branch"
     echo "Switch to main branch with: git checkout main"
     exit 1
   fi
   ```

3. **gh CLI is authenticated**:
   ```bash
   if ! gh auth status >/dev/null 2>&1; then
     echo "Error: gh CLI is not authenticated"
     echo "Run: gh auth login"
     exit 1
   fi
   ```

4. **yamllint is available**:
   ```bash
   if ! command -v yamllint >/dev/null 2>&1; then
     echo "Warning: yamllint not found. Install with: pip install yamllint"
   fi
   ```

**0.4 Get commit range**

```bash
prev_tag=$(git describe --tags --abbrev=0 2>/dev/null || echo "")
if [ -z "$prev_tag" ]; then
  # First release
  commit_range="HEAD"
else
  commit_range="${prev_tag}..HEAD"
fi

commits=$(git log $commit_range --format="%s")
commit_count=$(echo "$commits" | wc -l | tr -d ' ')

echo "Release preparation complete:"
echo "  Version: $new_version"
echo "  Commit range: $commit_range"
echo "  Commits: $commit_count"
echo ""
```

### Phase 1: Pre-release Verification

**1.1 Run test suite**

Execute unit tests:

```bash
echo "Running unit tests..."
cd tests

if ! ./test_init_script.sh; then
  echo "Error: test_init_script.sh failed"
  exit 1
fi

if ! ./test_mismatch_detection.sh; then
  echo "Error: test_mismatch_detection.sh failed"
  exit 1
fi

cd ..
echo "✅ All tests passed"
echo ""
```

**1.2 Validate provider.yaml**

```bash
if command -v yamllint >/dev/null 2>&1; then
  echo "Validating provider.yaml syntax..."
  if ! yamllint provider.yaml; then
    echo "Error: provider.yaml validation failed"
    exit 1
  fi
  echo "✅ provider.yaml syntax valid"
  echo ""
fi
```

**1.3 Verify documentation links**

Check that README.md and README.ja.md language selector links are valid:

```bash
echo "Verifying documentation links..."
if ! grep -q "README.ja.md" README.md; then
  echo "Warning: README.md does not link to README.ja.md"
fi
if ! grep -q "README.md" README.ja.md; then
  echo "Warning: README.ja.md does not link to README.md"
fi
echo "✅ Documentation links verified"
echo ""
```

**1.4 Verify provider icon URL**

```bash
echo "Verifying provider icon URL..."
icon_url=$(grep "^icon:" provider.yaml | sed 's/icon: //' | tr -d '"')
if [ -n "$icon_url" ]; then
  http_status=$(curl -s -o /dev/null -w "%{http_code}" "$icon_url")
  if [ "$http_status" -ne 200 ]; then
    echo "Warning: Icon URL returned HTTP $http_status: $icon_url"
  else
    echo "✅ Icon URL accessible"
  fi
else
  echo "Note: No icon URL defined"
fi
echo ""
```

### Phase 2: CHANGELOG Generation

**2.1 Parse git log for Conventional Commits**

Extract commits by category:

```bash
echo "Generating CHANGELOG entry..."

added_commits=$(echo "$commits" | grep "^feat:" | sed 's/^feat: /- /' || true)
fixed_commits=$(echo "$commits" | grep "^fix:" | sed 's/^fix: /- /' || true)
changed_commits=$(echo "$commits" | grep -E "^(docs|chore|refactor|test):" | sed 's/^[a-z]*: /- /' || true)
breaking_commits=$(echo "$commits" | grep "BREAKING CHANGE:" || true)
```

**2.2 Extract issue numbers**

```bash
# Extract issue references like #123, Refs: #123, Closes #123
extract_issue_links() {
  local text="$1"
  echo "$text" | sed -E 's/#([0-9]+)/[#\1](https:\/\/github.com\/kuju63\/podman-provider\/issues\/\1)/g'
}

added_commits=$(extract_issue_links "$added_commits")
fixed_commits=$(extract_issue_links "$fixed_commits")
changed_commits=$(extract_issue_links "$changed_commits")
```

**2.3 Build CHANGELOG entry**

```bash
changelog_entry="## [${new_version}] - $(date +%Y-%m-%d)

"

if [ -n "$breaking_commits" ]; then
  changelog_entry+="### ⚠️ Breaking Changes

$breaking_commits

"
fi

if [ -n "$added_commits" ]; then
  changelog_entry+="### Added

$added_commits

"
fi

if [ -n "$changed_commits" ]; then
  changelog_entry+="### Changed

$changed_commits

"
fi

if [ -n "$fixed_commits" ]; then
  changelog_entry+="### Fixed

$fixed_commits

"
fi

# Remove trailing newlines
changelog_entry=$(echo "$changelog_entry" | sed -e :a -e '/^\n*$/{ $d; N; ba' -e '}')
```

**2.4 Preview or insert CHANGELOG**

Reference the template at [references/changelog-template.md](references/changelog-template.md) for format guidance.

```bash
if [ "$dry_run" = true ]; then
  echo "=========================================="
  echo "CHANGELOG Preview (DRY-RUN)"
  echo "=========================================="
  echo "$changelog_entry"
  echo "=========================================="
  echo ""
else
  # Insert after line 7 (after "## [Unreleased]" section)
  # Use a temporary file to avoid sed issues
  head -7 CHANGELOG.md > CHANGELOG.tmp
  echo "$changelog_entry" >> CHANGELOG.tmp
  echo "" >> CHANGELOG.tmp
  tail -n +8 CHANGELOG.md >> CHANGELOG.tmp
  mv CHANGELOG.tmp CHANGELOG.md

  echo "✅ CHANGELOG.md updated"
  echo ""
fi
```

### Phase 3: Version Bump Commit

**3.1 Update provider.yaml version**

```bash
if [ "$dry_run" = false ]; then
  echo "Updating provider.yaml version..."
  sed -i.bak "2s/.*/version: ${new_version}/" provider.yaml
  rm provider.yaml.bak
  echo "✅ provider.yaml updated to $new_version"
  echo ""
fi
```

**3.2 Create version bump commit**

```bash
if [ "$dry_run" = false ]; then
  echo "Creating version bump commit..."
  git add CHANGELOG.md provider.yaml

  commit_message="chore: bump version to ${new_version}

Release notes:
$(echo "$changelog_entry" | head -20)

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"

  git commit -m "$commit_message"
  echo "✅ Commit created"
  echo ""
fi
```

### Phase 4: Git Tag Creation

**4.1 Create annotated tag**

```bash
if [ "$dry_run" = false ]; then
  echo "Creating git tag..."

  # Check if tag already exists
  if git rev-parse "$new_version" >/dev/null 2>&1; then
    echo "Error: Tag $new_version already exists"
    echo "Existing tags:"
    git tag --list | tail -5
    exit 1
  fi

  tag_message="Release ${new_version}

$(echo "$changelog_entry" | head -10)"

  git tag -a "$new_version" -m "$tag_message"
  echo "✅ Tag $new_version created"
  echo ""
fi
```

### Phase 5: Push Changes

**5.1 Push commit and tag**

```bash
if [ "$dry_run" = false ]; then
  echo "Pushing changes to remote..."

  # Confirm before push
  read -p "Push commit and tag to origin/main? (y/n) " -n 1 -r
  echo
  if [[ ! $REPLY =~ ^[Yy]$ ]]; then
    echo "Push cancelled. You can manually push with:"
    echo "  git push origin main"
    echo "  git push origin $new_version"
    exit 0
  fi

  git push origin main
  git push origin "$new_version"
  echo "✅ Changes pushed"
  echo ""
fi
```

### Phase 6: GitHub Release Creation

**6.0 Validate release assets**

Check that provider.yaml exists and is valid:

```bash
if [ "$dry_run" = false ]; then
  echo "Validating release assets..."

  if [ ! -f "provider.yaml" ]; then
    echo "Error: provider.yaml not found in current directory"
    echo "Cannot create release without provider asset"
    exit 1
  fi

  # Verify provider.yaml is valid YAML (if yamllint available)
  if command -v yamllint >/dev/null 2>&1; then
    if ! yamllint provider.yaml >/dev/null 2>&1; then
      echo "Error: provider.yaml has syntax errors"
      echo "Run: yamllint provider.yaml"
      exit 1
    fi
  fi

  echo "✅ Release asset validated: provider.yaml"
  echo ""
fi
```

**6.1 Generate release notes**

Reference the template at [references/release-notes-template.md](references/release-notes-template.md).

```bash
if [ "$dry_run" = false ]; then
  echo "Creating GitHub Release..."

  release_notes="## 📦 Release ${new_version}

${changelog_entry}

## 🚀 Installation

\`\`\`bash
devpod provider add github.com/kuju63/podman-provider
devpod provider use podman
\`\`\`

## 🔗 Links

- **Full Changelog**: https://github.com/kuju63/podman-provider/compare/${prev_tag}...${new_version}
- **Documentation**: https://github.com/kuju63/podman-provider/blob/${new_version}/README.md

---
🤖 Generated with [Claude Code](https://claude.com/claude-code)"

  # Create release with provider.yaml as asset
  # Note: Files are specified after the tag name as positional arguments
  echo "Uploading asset: provider.yaml"
  gh release create "$new_version" \
    provider.yaml \
    --title "Release ${new_version}" \
    --notes "$release_notes"

  echo "✅ GitHub Release created with asset: provider.yaml"
  echo ""
fi
```

**6.2 Dry-run preview**

```bash
if [ "$dry_run" = true ]; then
  echo "=========================================="
  echo "Release Notes Preview (DRY-RUN)"
  echo "=========================================="
  echo "$release_notes"
  echo "=========================================="
  echo ""
  echo "Asset to attach: provider.yaml"
  if [ -f "provider.yaml" ]; then
    echo "  ✅ File exists: $(ls -lh provider.yaml | awk '{print $5}')"
  else
    echo "  ❌ File missing: provider.yaml"
    echo "  Error: Release creation would fail"
  fi
  echo "=========================================="
  echo ""
fi
```

### Phase 7: Post-release Verification

**7.1 Verify remote tag**

```bash
if [ "$dry_run" = false ]; then
  echo "Verifying release..."

  # Check remote tag
  if git ls-remote --tags origin | grep -q "$new_version"; then
    echo "✅ Remote tag verified"
  else
    echo "⚠️ Warning: Remote tag not found (may need time to propagate)"
  fi

  # Display release URL
  release_url="https://github.com/kuju63/podman-provider/releases/tag/${new_version}"
  echo ""
  echo "=========================================="
  echo "🎉 Release Complete!"
  echo "=========================================="
  echo "Version: $new_version"
  echo "Release URL: $release_url"
  echo "=========================================="
else
  echo "=========================================="
  echo "🔍 Dry-run Complete!"
  echo "=========================================="
  echo "To execute the release, run:"
  echo "  /release $new_version"
  echo "=========================================="
fi
```

## Error Handling

### Common Error Cases

1. **No commits since last tag**:
   - Message: "Error: No commits since last release (TAG)"
   - Resolution: Create commits or skip release

2. **Dirty working tree**:
   - Message: "Error: Working tree is not clean"
   - Resolution: Run `git status` and commit/stash changes

3. **Not on main branch**:
   - Message: "Error: Not on main branch. Current branch: BRANCH"
   - Resolution: Run `git checkout main`

4. **Tag already exists**:
   - Message: "Error: Tag VERSION already exists"
   - Resolution: Choose a different version or delete old tag

5. **Test failures**:
   - Message: "Error: TESTFILE failed"
   - Resolution: Fix failing tests before release

6. **gh CLI not authenticated**:
   - Message: "Error: gh CLI is not authenticated"
   - Resolution: Run `gh auth login`

7. **provider.yaml not found**:
   - Message: "Error: provider.yaml not found in current directory"
   - Resolution: Verify you're in repository root directory
   - Command: `ls provider.yaml` to verify file exists

8. **provider.yaml syntax errors**:
   - Message: "Error: provider.yaml has syntax errors"
   - Resolution: Fix YAML syntax issues
   - Command: `yamllint provider.yaml` for detailed errors

## Rollback Instructions

If release fails after Phase 5 (push), manual rollback is required:

```bash
# Delete local tag
git tag -d vX.Y.Z

# Reset version bump commit
git reset --hard HEAD~1

# Delete remote tag (if pushed)
git push origin --delete vX.Y.Z

# Restore CHANGELOG.md and provider.yaml
git checkout HEAD -- CHANGELOG.md provider.yaml
```

## Examples

### Example 1: Auto-detect version (default)

```bash
/release
```

Output:
```
Version Auto-Detection Results
Current version: v0.3.0
Commit range: v0.3.0..HEAD (5 commits)
Detected bump type: minor
New version: v0.4.0

Sample commits:
feat: add automatic resource update
fix: handle podman machine name with asterisk
docs: improve README installation steps
...

Proceed with version v0.4.0? (y/n)
```

### Example 2: Manual version specification

```bash
/release v0.4.0
```

### Example 3: Dry-run mode

```bash
/release --dry-run
```

or

```bash
/release v0.4.0 --dry-run
```

## References

- [Changelog Template](references/changelog-template.md) - Keep a Changelog format example
- [Release Notes Template](references/release-notes-template.md) - GitHub Release notes structure
- [Semantic Versioning Calculator](scripts/semver.sh) - Version bump calculation script (optional)
