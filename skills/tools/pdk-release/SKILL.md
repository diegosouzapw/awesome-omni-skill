---
name: pdk-release
description: Release echo-pdk packages to npm. Bumps versions, creates tags, triggers GitHub Actions, and verifies frontend consumption.
allowed-tools: Bash, Read, Edit
---

# Echo PDK Release Workflow

Automates the full release process for echo-pdk packages including npm publish via GitHub Actions.

## Usage

```
/pdk-release core minor        # Release core package with minor bump
/pdk-release cli patch         # Release CLI package with patch bump
/pdk-release core,cli minor    # Release multiple packages
/pdk-release all patch         # Release all packages
```

## Arguments

- **packages**: `core`, `cli`, `language`, or `all` (comma-separated for multiple)
- **bump**: `patch`, `minor`, or `major`

## Step-by-Step Instructions

### Step 1: Pre-Release Checks

```bash
cd /Users/yoadelkayam/github/PDK/echo-pdk

# Verify clean working directory
git status -sb

# Ensure on main and up-to-date
git checkout main
git pull origin main

# Run tests
pnpm test

# Build all packages
pnpm build
```

If tests fail, STOP and report the failure.

### Step 2: Determine Current Versions

Read current versions from package.json files:

```bash
# Core package
cat packages/core/package.json | grep '"version"'

# CLI package
cat packages/cli/package.json | grep '"version"'

# Language package
cat packages/language/package.json | grep '"version"'
```

### Step 3: Bump Versions

Calculate new versions based on bump type and update package.json files.

**Version bump rules:**
- `patch`: 0.5.1 → 0.5.2
- `minor`: 0.5.1 → 0.6.0
- `major`: 0.5.1 → 1.0.0

Edit the `"version"` field in each package.json being released.

### Step 4: Generate Changelog

Get commits since last tag:

```bash
# For core package
git log $(git tag -l 'core-v*' --sort=-v:refname | head -1)..HEAD --oneline -- packages/core/

# For CLI package
git log $(git tag -l 'cli-v*' --sort=-v:refname | head -1)..HEAD --oneline -- packages/cli/

# For language package
git log $(git tag -l 'language-v*' --sort=-v:refname | head -1)..HEAD --oneline -- packages/language/
```

### Step 5: Create Commit

Stage and commit all changes:

```bash
git add packages/*/package.json

git commit -m "chore: release @goreal-ai/echo-pdk v{CORE_VERSION}, @goreal-ai/echo-pdk-cli v{CLI_VERSION}

Co-Authored-By: Claude <noreply@anthropic.com>"
```

### Step 6: Create Tags

Create annotated tags for each package being released:

```bash
# For core
git tag -a core-v{VERSION} -m "Release @goreal-ai/echo-pdk v{VERSION}"

# For CLI
git tag -a cli-v{VERSION} -m "Release @goreal-ai/echo-pdk-cli v{VERSION}"

# For language
git tag -a language-v{VERSION} -m "Release @goreal-ai/echo-pdk-language v{VERSION}"
```

### Step 7: Push to Origin

```bash
git push origin main
git push --tags
```

### Step 8: Create GitHub Releases

This triggers the npm publish workflow.

```bash
# For core package
gh release create core-v{VERSION} \
  --title "@goreal-ai/echo-pdk v{VERSION}" \
  --notes "## Changes

{CHANGELOG_CONTENT}

## Breaking Changes
None"

# For CLI package
gh release create cli-v{VERSION} \
  --title "@goreal-ai/echo-pdk-cli v{VERSION}" \
  --notes "## Changes

- Updated to use @goreal-ai/echo-pdk v{CORE_VERSION}"

# For language package
gh release create language-v{VERSION} \
  --title "@goreal-ai/echo-pdk-language v{VERSION}" \
  --notes "## Changes

{CHANGELOG_CONTENT}"
```

### Step 9: Monitor GitHub Actions

Wait for workflows to complete:

```bash
# Check workflow status
gh run list --limit 5

# Watch specific run
gh run watch {RUN_ID}
```

All workflows should show `completed` with `success` status.

### Step 10: Verify npm Publication

```bash
# Check npm for new versions
npm view @goreal-ai/echo-pdk version
npm view @goreal-ai/echo-pdk-cli version
npm view @goreal-ai/echo-pdk-language version
```

### Step 11: Check Frontend Consumption

Verify gra-echostash-fe is using the correct version:

```bash
# Check current version in frontend
cat /Users/yoadelkayam/github/Echostash/gra-echostash-fe/package.json | grep "@goreal-ai/echo-pdk"
```

**If frontend needs update:**

```bash
cd /Users/yoadelkayam/github/Echostash/gra-echostash-fe

# Update to new version
pnpm update @goreal-ai/echo-pdk@{NEW_VERSION}

# Verify update
cat package.json | grep "@goreal-ai/echo-pdk"

# Run frontend build to verify compatibility
pnpm build
```

Report whether frontend update is needed and its status.

## Output Summary

After completing, report:

| Package | Old Version | New Version | npm Status | GitHub Release |
|---------|-------------|-------------|------------|----------------|
| core    | x.x.x       | y.y.y       | ✅/❌      | URL            |
| cli     | x.x.x       | y.y.y       | ✅/❌      | URL            |
| language| x.x.x       | y.y.y       | ✅/❌      | URL            |

**Frontend Status:**
- Current version: x.x.x
- Needs update: Yes/No
- Updated: Yes/No

## Rollback

If release fails:

```bash
# Delete local tags
git tag -d core-v{VERSION}
git tag -d cli-v{VERSION}

# Delete remote tags
git push --delete origin core-v{VERSION}
git push --delete origin cli-v{VERSION}

# Reset commit
git reset --hard HEAD~1
git push --force-with-lease

# Delete GitHub releases
gh release delete core-v{VERSION} --yes
gh release delete cli-v{VERSION} --yes
```
