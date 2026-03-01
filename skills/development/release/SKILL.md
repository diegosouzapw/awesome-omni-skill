---
name: release
description: How to release a new pgdbm library version to PyPI
---

# Releasing pgdbm

## Overview

This skill guides you through releasing a new version of the pgdbm library.

## Prerequisites

- All tests passing
- Changes committed to main branch
- CHANGELOG.md updated with changes

## Release Checklist

### 1. Run Tests

```bash
uv run pytest
```

All tests must pass before releasing.

### 2. Update Version Numbers

Three files need version updates:

```bash
# Check current version
grep "^version" pyproject.toml
```

Update these files to the new version:

| File | Location |
|------|----------|
| `pyproject.toml` | `version = "X.Y.Z"` |
| `src/pgdbm/__version__.py` | `__version__ = "X.Y.Z"` |
| `.claude-plugin/plugin.json` | `"version": "X.Y.Z"` |

**Important**: The plugin.json version controls Claude Code skill caching. Always bump it when releasing.

### 3. Update CHANGELOG.md

Add entry at the top:

```markdown
## [X.Y.Z] - YYYY-MM-DD

### Added
- New features

### Changed
- Changes to existing features

### Fixed
- Bug fixes
```

### 4. Commit the Release

```bash
git add pyproject.toml src/pgdbm/__version__.py .claude-plugin/plugin.json CHANGELOG.md uv.lock
git commit -m "chore: bump version to X.Y.Z

- Summary of changes

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
```

### 5. Create Git Tag

```bash
git tag -a vX.Y.Z -m "Release X.Y.Z"
```

### 6. Push to GitHub

```bash
git push origin main
git push origin vX.Y.Z
```

### 7. Publish to PyPI

**Important**: Clean the dist/ directory first to avoid uploading old versions.

```bash
rm -rf dist/
uv build
uv publish
```

Or if using twine:

```bash
rm -rf dist/
uv build
twine upload dist/*
```

### 8. Verify Release

```bash
# Check PyPI
pip index versions pgdbm

# Check plugin update works
claude plugin update pgdbm@juanre-ai-tools
```

## Version Numbering

Follow semver:
- **MAJOR** (X.0.0): Breaking API changes
- **MINOR** (0.X.0): New features, backward compatible
- **PATCH** (0.0.X): Bug fixes, documentation

## Related Skills

- For skill updates: See `publish-skills` skill
