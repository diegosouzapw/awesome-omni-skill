---
name: release
description: Create a release following Git Flow strategy. Use when preparing a release, bumping version, creating release branch, or when user says "release", "bump version", "prepare release", "ship version".
allowed-tools: Bash, Read, Edit
---

# Release Process

This skill guides the release process following Git Flow with automated CI/CD.

## Release Flow Overview

```
develop ──▶ release/v{X.Y.Z} ──PR──▶ main
   ▲                                   │
   │                                   ▼
   │                            (auto-release.yml)
   │                                   │
   │                                   ▼
   │                              tag: v{X.Y.Z}
   │                                   │
   │                                   ▼
   │                            (publish.yml)
   │                                   │
   │                                   ▼
   │                              JSR publish
   │                                   │
   └───────────── backmerge ◀──────────┘
```

## Workflow

### 1. Ensure develop is ready

```bash
git checkout develop
git pull origin develop
git status
```

Verify all features for this release are merged.

### 2. Run full CI locally

```bash
deno task ci
```

All tests must pass before proceeding.

### 3. Determine version number

Follow semantic versioning:

- **MAJOR** (X.0.0): Breaking changes
- **MINOR** (0.X.0): New features, backward compatible
- **PATCH** (0.0.X): Bug fixes only

Check current version:

```bash
jq -r '.version' deno.json
```

### 4. Create release branch

```bash
git checkout -b release/v{NEW_VERSION}
```

### 5. Bump version

Use the project's bump script:

```bash
bash scripts/bump_version.sh
```

Or manually update `deno.json`:

```bash
# Edit deno.json version field
```

### 6. Update CHANGELOG.md

Add release notes under new version heading.

### 7. Commit version bump

```bash
git add deno.json CHANGELOG.md
git commit -m "chore: bump version to {NEW_VERSION}"
```

### 8. Push and create PR to main

```bash
git push -u origin release/v{NEW_VERSION}
```

Create PR: `release/v{NEW_VERSION}` → `main`

**Important**: PR must be from `release/*` branch directly to `main` for auto-release to trigger.

### 9. Merge triggers automation

When PR to main is merged:

1. `auto-release.yml` creates tag `v{NEW_VERSION}`
2. `publish.yml` publishes to JSR

### 10. Sync develop with main

After release is published, backmerge main to develop:

```bash
git checkout develop
git pull origin develop
git merge origin/main
git push origin develop
```

This ensures develop has the version bump and CHANGELOG updates.

## Checklist

- [ ] All features merged to develop
- [ ] `deno task ci` passes
- [ ] Version number determined (semver)
- [ ] Release branch created from develop
- [ ] Version bumped in `deno.json`
- [ ] CHANGELOG.md updated
- [ ] PR to main created (from release/* branch)
- [ ] PR merged → auto-release creates tag
- [ ] Verify JSR publication
- [ ] Backmerge main to develop

## Example: Release v1.8.0

```bash
# Prepare
git checkout develop
git pull origin develop
deno task ci

# Create release
git checkout -b release/v1.8.0
bash scripts/bump_version.sh
# Edit CHANGELOG.md
git add deno.json CHANGELOG.md
git commit -m "chore: bump version to 1.8.0"
git push -u origin release/v1.8.0

# Create PR via GitHub
# release/v1.8.0 → main (triggers auto-release on merge)

# After merge and JSR publish, sync develop
git checkout develop
git pull origin develop
git merge origin/main
git push origin develop
```

## Important Notes

- Never push directly to `main`
- Version in `deno.json` must match tag
- Auto-release only triggers on `release/*` branch merge to main
- JSR publish requires tag to be on main branch
