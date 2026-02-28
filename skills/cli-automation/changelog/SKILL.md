---
name: changelog
description: Generates release notes and changelogs by analyzing git history, commits, PRs, and tags between versions. Use when the user says "generate changelog", "release notes", "what changed since last release", "write changelog", "prepare release", "what's new", or "summarize changes".
allowed-tools: Read, Grep, Glob, Bash
---

# Changelog Skill

When generating a changelog, follow this structured process. Good release notes tell users and developers what changed, why it matters, and what they need to do about it.

## 1. Discovery — Understand the Release Context

### Detect Versioning Strategy
```bash
# Check for existing changelog
cat CHANGELOG.md CHANGELOG CHANGES.md HISTORY.md RELEASE_NOTES.md 2>/dev/null | head -30

# Check package version
cat package.json 2>/dev/null | grep '"version"'
cat pyproject.toml 2>/dev/null | grep 'version'
cat Cargo.toml 2>/dev/null | grep '^version'
cat pom.xml 2>/dev/null | grep -m1 '<version>'
cat build.gradle 2>/dev/null | grep 'version'
cat mix.exs 2>/dev/null | grep '@version'
cat setup.py 2>/dev/null | grep 'version'
cat composer.json 2>/dev/null | grep '"version"'
cat *.gemspec 2>/dev/null | grep 'version'

# List existing tags
git tag --sort=-version:refname | head -20

# Latest tag
git describe --tags --abbrev=0 2>/dev/null

# Check for release branches
git branch -r | grep -E "release/|v[0-9]"
```

### Determine Range
```bash
# Commits since last tag
LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null)
echo "Last tag: $LAST_TAG"

# If no tags, use initial commit
if [ -z "$LAST_TAG" ]; then
  LAST_TAG=$(git rev-list --max-parents=0 HEAD)
  echo "No tags found, using first commit: $LAST_TAG"
fi

# Commit count since last tag
git rev-list --count ${LAST_TAG}..HEAD

# Date range
echo "From: $(git log -1 --format=%ci $LAST_TAG)"
echo "To: $(git log -1 --format=%ci HEAD)"
```

## 2. Gather Changes

### Collect Commits
```bash
LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null)

# All commits since last tag with full message
git log ${LAST_TAG}..HEAD --format="COMMIT_START%nHash: %H%nShort: %h%nAuthor: %aN <%aE>%nDate: %ai%nSubject: %s%nBody: %b%nCOMMIT_END"

# One-line summary
git log ${LAST_TAG}..HEAD --oneline

# With file changes
git log ${LAST_TAG}..HEAD --oneline --name-status

# Diff stats
git diff --stat ${LAST_TAG}..HEAD

# Contributors
git log ${LAST_TAG}..HEAD --format='%aN' | sort | uniq -c | sort -rn
```

### Collect Merged PRs (GitHub)
```bash
# List merged PRs since last tag date
LAST_TAG_DATE=$(git log -1 --format=%ci $(git describe --tags --abbrev=0 2>/dev/null) 2>/dev/null)

# Using gh CLI
gh pr list --state merged --search "merged:>=${LAST_TAG_DATE}" --json number,title,labels,author,body --limit 100

# Alternative: extract PR numbers from merge commits
git log ${LAST_TAG}..HEAD --oneline --grep="Merge pull request" | grep -oE "#[0-9]+"
git log ${LAST_TAG}..HEAD --oneline --grep="(#[0-9]+)" | grep -oE "#[0-9]+"
```

### Collect Merged MRs (GitLab)
```bash
# Using glab CLI
glab mr list --state merged --json number,title,labels,author
```

## 3. Categorize Changes

### Parse Conventional Commits

Classify commits based on their prefix:

| Prefix | Category | User-Facing Label |
|--------|----------|-------------------|
| `feat:` / `feature:` | Features | ✨ New Features |
| `fix:` / `bugfix:` | Bug Fixes | 🐛 Bug Fixes |
| `perf:` | Performance | ⚡ Performance Improvements |
| `security:` | Security | 🔒 Security |
| `breaking:` / `BREAKING CHANGE:` | Breaking | 💥 Breaking Changes |
| `refactor:` | Refactoring | ♻️ Refactoring |
| `docs:` | Documentation | 📝 Documentation |
| `test:` | Testing | ✅ Testing |
| `chore:` | Maintenance | 🔧 Maintenance |
| `ci:` | CI/CD | 🏗️ CI/CD |
| `style:` | Code Style | 💄 Code Style |
| `deps:` / `build:` | Dependencies | 📦 Dependencies |
| `revert:` | Reverts | ⏪ Reverts |
| `deprecate:` | Deprecations | ⚠️ Deprecations |

### If Commits Are NOT Conventional

When commits don't follow conventional format, analyze the diff to categorize:
```bash
# Analyze what areas changed
git diff --stat ${LAST_TAG}..HEAD | tail -1  # summary

# Group by directory/module
git diff --name-only ${LAST_TAG}..HEAD | sed 's|/.*||' | sort | uniq -c | sort -rn

# Look for keywords in commit messages
git log ${LAST_TAG}..HEAD --oneline | grep -iE "fix|bug|patch|resolve" # Bug fixes
git log ${LAST_TAG}..HEAD --oneline | grep -iE "add|new|feature|implement|support" # Features
git log ${LAST_TAG}..HEAD --oneline | grep -iE "remove|delete|deprecate|drop" # Removals
git log ${LAST_TAG}..HEAD --oneline | grep -iE "update|upgrade|bump|depend" # Dependencies
git log ${LAST_TAG}..HEAD --oneline | grep -iE "perf|speed|fast|optim|cache" # Performance
git log ${LAST_TAG}..HEAD --oneline | grep -iE "security|vuln|cve|auth" # Security
git log ${LAST_TAG}..HEAD --oneline | grep -iE "refactor|clean|restructure" # Refactoring
git log ${LAST_TAG}..HEAD --oneline | grep -iE "doc|readme|comment" # Docs
git log ${LAST_TAG}..HEAD --oneline | grep -iE "test|spec|coverage" # Testing
git log ${LAST_TAG}..HEAD --oneline | grep -iE "ci|pipeline|workflow|deploy" # CI/CD
```

### Detect Breaking Changes
```bash
# Check commit bodies for BREAKING CHANGE
git log ${LAST_TAG}..HEAD --format="%B" | grep -i "BREAKING CHANGE"

# Check for removed exports/endpoints
git diff ${LAST_TAG}..HEAD -- "*.ts" "*.js" | grep -E "^-export|^-module\.exports"
git diff ${LAST_TAG}..HEAD -- "*.py" | grep -E "^-def |^-class "

# Check for removed API routes
git diff ${LAST_TAG}..HEAD -- "*route*" "*controller*" "*endpoint*" | grep "^-"

# Check for database migrations that drop/rename
find . -name "*.sql" -newer $(git rev-parse ${LAST_TAG}) | xargs grep -liE "DROP|RENAME|ALTER.*DROP" 2>/dev/null

# Check for changed config/env requirements
git diff ${LAST_TAG}..HEAD -- ".env.example" "*.config.*" "docker-compose.*"
```

## 4. Changelog Formats

### Format A: Keep a Changelog (Recommended for Libraries/OSS)

Follows [keepachangelog.com](https://keepachangelog.com) format:
```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [2.4.0] - 2026-02-17

### Added
- OAuth2 login support with Google and GitHub providers (#234)
- Real-time order status notifications via WebSocket (#241)
- Bulk export of analytics data in CSV and JSON formats (#238)
- Rate limiting on all public API endpoints (#245)

### Changed
- Upgraded Next.js from 14.2 to 15.1 (#250)
- Migrated email service from Mailgun to SendGrid (#247)
- Improved search indexing performance by 3x with batch processing (#243)

### Fixed
- Fixed duplicate charge on payment retry when Stripe webhook is delayed (#236)
- Fixed pagination returning wrong total count on filtered queries (#239)
- Fixed timezone handling in scheduled report generation (#242)
- Fixed memory leak in WebSocket connection handler (#248)

### Security
- Patched XSS vulnerability in user profile bio field (#237)
- Updated jsonwebtoken to 9.0.2 to fix CVE-2024-XXXXX (#246)

### Deprecated
- The `/api/v1/reports` endpoint is deprecated in favor of `/api/v2/analytics`. Will be removed in v3.0.0.

### Removed
- Removed legacy CSV import endpoint `/api/v1/import/csv` (use `/api/v2/import` instead)

## [2.3.1] - 2026-01-28

### Fixed
- ...

[Unreleased]: https://github.com/org/repo/compare/v2.4.0...HEAD
[2.4.0]: https://github.com/org/repo/compare/v2.3.1...v2.4.0
[2.3.1]: https://github.com/org/repo/compare/v2.3.0...v2.3.1
```

### Format B: User-Facing Release Notes (For Products/Apps)

Written for end users, not developers:
```markdown
# What's New in v2.4.0 🚀

Released: February 17, 2026

## Highlights

### Sign in with Google and GitHub
You can now log in using your Google or GitHub account — no more passwords
to remember. Head to Settings → Account → Connected Accounts to link yours.

### Live Order Tracking
Watch your order status update in real time. No more refreshing the page —
you'll see status changes the moment they happen.

### Bulk Data Export
Export your analytics data in CSV or JSON format. Perfect for custom reports
or importing into your own tools. Find it under Analytics → Export.

## Improvements

- **Faster search**: Search results now load 3x faster, especially on large catalogs
- **Better emails**: We've moved to a more reliable email provider — fewer delays on order confirmations
- **API rate limits**: Added rate limiting to protect against abuse and ensure consistent performance

## Bug Fixes

- Fixed an issue where payments could be charged twice if your connection dropped during checkout
- Fixed incorrect page counts when filtering product lists
- Fixed scheduled reports showing wrong timezone for some regions
- Fixed a rare issue where the app could slow down after staying connected for a long time

## Security

- Fixed a text rendering issue in user profiles that could allow script injection
- Updated a security library to patch a known vulnerability

## Breaking Changes

⚠️ **API v1 Reports Endpoint Deprecated**
The `/api/v1/reports` endpoint is now deprecated. Please migrate to
`/api/v2/analytics` before July 2026 when v1 will be removed.
See our [migration guide](link) for details.

## Upgrade Notes

No action required for most users. If you use our API:
- Update any code using `/api/v1/import/csv` to `/api/v2/import`
- Add rate limit handling (429 responses) to your API clients

---

**Full Changelog**: [v2.3.1...v2.4.0](https://github.com/org/repo/compare/v2.3.1...v2.4.0)
```

### Format C: Developer Release Notes (For Internal Teams)

Technical and detailed, for the engineering team:
```markdown
# Release v2.4.0

**Date**: 2026-02-17
**Tag**: v2.4.0
**Commits**: 47 commits from 8 contributors
**PRs Merged**: 16

## Summary

This release adds OAuth2 authentication, real-time WebSocket notifications,
and bulk data export. Major infrastructure change: email migration from
Mailgun to SendGrid. Performance improvement on search indexing (3x faster).

## Breaking Changes

### API
- `DELETE /api/v1/import/csv` — Removed. Use `POST /api/v2/import` with `format: "csv"` header
- `GET /api/v1/reports` — Deprecated (still works). Returns `Sunset: 2026-07-01` header

### Database
- Migration `20260215_add_oauth_providers` adds `oauth_providers` table and `users.oauth_id` column
- Migration `20260216_add_websocket_sessions` adds `ws_sessions` table

### Environment Variables
- **New Required**: `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`
- **New Required**: `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`
- **New Required**: `SENDGRID_API_KEY` (replaces `MAILGUN_API_KEY`)
- **Removed**: `MAILGUN_API_KEY`, `MAILGUN_DOMAIN`

## Changes by Area

### Authentication (#234, #237)
- Added OAuth2 flow with PKCE for Google and GitHub
- New tables: `oauth_providers`, `oauth_tokens`
- Modified: `users` table (added `oauth_id`, `oauth_provider` columns)
- Patched XSS in profile bio (used DOMPurify for sanitization)
- Files: `src/services/auth/`, `src/api/routes/auth.ts`, `src/db/migrations/`

### Real-time Notifications (#241, #248)
- WebSocket server using `ws` library on `/ws` path
- Redis pub/sub for multi-instance broadcast
- Fixed connection leak (missing `close` event handler)
- Files: `src/services/websocket/`, `src/lib/redis-pubsub.ts`

### Data Export (#238)
- New endpoint: `POST /api/v2/analytics/export`
- Supports CSV and JSON output
- Streams results for large datasets (no memory issues)
- Files: `src/api/routes/analytics.ts`, `src/services/export/`

### Search Performance (#243)
- Batch indexing: 100 items per batch instead of 1-by-1
- Added Redis cache layer (5-minute TTL)
- Reduced average index time from 3.2s to 0.9s
- Files: `src/services/search/`, `src/lib/cache.ts`

### Email Migration (#247)
- Swapped Mailgun SDK for SendGrid v3
- Updated all email templates (same look, different template engine)
- Added email delivery webhooks for bounce tracking
- Files: `src/lib/email.ts`, `src/templates/email/`

### Infrastructure (#245, #246, #250)
- Added express-rate-limit: 100 req/min per IP on public endpoints
- Upgraded Next.js 14.2 → 15.1 (React 19, Server Actions stable)
- Updated jsonwebtoken 8.5.1 → 9.0.2 (CVE fix)

## Deployment Checklist

- [ ] Run database migrations (`npm run db:migrate`)
- [ ] Add new environment variables (see Breaking Changes)
- [ ] Remove old Mailgun env vars
- [ ] Verify SendGrid API key has correct permissions
- [ ] Verify OAuth callback URLs are registered with Google/GitHub
- [ ] Monitor WebSocket connections after deploy (new feature)
- [ ] Monitor email delivery rate (new provider)
- [ ] Verify rate limiting is not blocking legitimate traffic

## Rollback Plan

If issues arise:
1. Revert to v2.3.1 tag: `git checkout v2.3.1`
2. Run down migration: `npm run db:migrate:down -- --to 20260214`
3. Restore old env vars (MAILGUN_*)
4. Redeploy

## Contributors

- @alice — OAuth2 implementation, security fix
- @bob — WebSocket notifications, connection leak fix
- @carol — Data export, search performance
- @dave — Email migration, Next.js upgrade
- @eve — Rate limiting, dependency updates

## Stats

| Metric | Value |
|--------|-------|
| Commits | 47 |
| PRs merged | 16 |
| Files changed | 83 |
| Lines added | +3,241 |
| Lines removed | -1,087 |
| New tests | 34 |
| Contributors | 8 |
```

### Format D: Compact Changelog (For Rapid Releases)

Short format for teams that release frequently:
```markdown
## v2.4.0 (2026-02-17)

**Features**: OAuth2 login (Google/GitHub) (#234), real-time order notifications (#241), bulk data export (#238)
**Fixes**: Duplicate payment charge (#236), pagination count (#239), timezone in reports (#242), WebSocket memory leak (#248)
**Security**: XSS in profile bio (#237), jsonwebtoken CVE (#246)
**Performance**: Search indexing 3x faster (#243)
**Infrastructure**: Next.js 15.1 (#250), Mailgun → SendGrid (#247), API rate limiting (#245)
**Breaking**: Removed `/api/v1/import/csv`, deprecated `/api/v1/reports`
**New env vars**: `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`, `SENDGRID_API_KEY`
```

## 5. Version Bump Suggestion

### Semantic Versioning Rules
Based on the changes detected, suggest the appropriate version bump:

| Change Type | Bump | Example |
|-------------|------|---------|
| Breaking changes (removed API, changed behavior) | **MAJOR** | 2.4.0 → 3.0.0 |
| New features (backward compatible) | **MINOR** | 2.4.0 → 2.5.0 |
| Bug fixes only | **PATCH** | 2.4.0 → 2.4.1 |
```bash
# Suggest version based on commits
HAS_BREAKING=$(git log ${LAST_TAG}..HEAD --format="%B" | grep -ci "BREAKING CHANGE\|breaking:")
HAS_FEAT=$(git log ${LAST_TAG}..HEAD --format="%s" | grep -ci "^feat")
HAS_FIX=$(git log ${LAST_TAG}..HEAD --format="%s" | grep -ci "^fix")

CURRENT=$(git describe --tags --abbrev=0 | sed 's/^v//')
MAJOR=$(echo $CURRENT | cut -d. -f1)
MINOR=$(echo $CURRENT | cut -d. -f2)
PATCH=$(echo $CURRENT | cut -d. -f3)

if [ "$HAS_BREAKING" -gt "0" ]; then
  echo "Suggested: v$((MAJOR+1)).0.0 (MAJOR — breaking changes detected)"
elif [ "$HAS_FEAT" -gt "0" ]; then
  echo "Suggested: v${MAJOR}.$((MINOR+1)).0 (MINOR — new features)"
else
  echo "Suggested: v${MAJOR}.${MINOR}.$((PATCH+1)) (PATCH — bug fixes only)"
fi
```

## 6. Updating the Changelog File

### Prepend to Existing CHANGELOG.md
```bash
# If CHANGELOG.md exists, prepend the new entry
# Keep the header, insert new version after it

# Detect changelog format from existing file
head -20 CHANGELOG.md 2>/dev/null
```

### Rules for Updating
- **Never overwrite** — always prepend new entries at the top (below header)
- **Keep existing entries** — never modify past versions
- **Update [Unreleased] section** — move unreleased items into the new version
- **Add comparison links** — update the link references at the bottom
- **Match existing style** — use the same format as previous entries

## 7. Platform-Specific Release Creation

### GitHub Release
```bash
# Create tag
git tag -a v2.4.0 -m "Release v2.4.0"
git push origin v2.4.0

# Create GitHub release
gh release create v2.4.0 \
  --title "v2.4.0" \
  --notes-file release-notes.md \
  --latest

# Create as pre-release
gh release create v2.4.0-beta.1 \
  --title "v2.4.0 Beta 1" \
  --notes-file release-notes.md \
  --prerelease

# Create draft release for review
gh release create v2.4.0 \
  --title "v2.4.0" \
  --notes-file release-notes.md \
  --draft
```

### GitLab Release
```bash
# Create tag and release
glab release create v2.4.0 \
  --name "v2.4.0" \
  --notes "$(cat release-notes.md)"
```

### npm Publish
```bash
# Update version in package.json
npm version minor  # or major/patch
# This creates a git tag and updates package.json

# Publish
npm publish
```

### PyPI Publish
```bash
# Update version in pyproject.toml
# Build and publish
python -m build
python -m twine upload dist/*
```

### Cargo Publish (Rust)
```bash
# Update version in Cargo.toml
cargo publish
```

## 8. Automation & CI Integration

### GitHub Actions Workflow for Auto-Changelog
```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    tags:
      - 'v*'
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Generate changelog
        run: |
          PREV_TAG=$(git describe --tags --abbrev=0 HEAD^ 2>/dev/null || echo "")
          git log ${PREV_TAG}..HEAD --oneline > changelog.txt
      - name: Create release
        uses: softprops/action-gh-release@v2
        with:
          body_path: changelog.txt
```

### Suggest CI Setup
If no automated changelog exists, suggest setting up:
- Commit message linting (commitlint + husky)
- Automated changelog generation in CI
- Version bumping scripts
- Release automation

## Output Format

Present the changelog as:

### 1. Release Summary
```
Release: v2.4.0 (previous: v2.3.1)
Commits: 47 | PRs: 16 | Contributors: 8
Date range: 2026-01-28 → 2026-02-17
Version bump: MINOR (new features, no breaking changes)
```

### 2. Changelog Content
The full formatted changelog in the appropriate format.

### 3. Actions
- Update CHANGELOG.md (show the exact edit)
- Create git tag
- Create platform release (GitHub/GitLab)
- Publish package (if applicable)

## Adaptation Rules

- **Detect changelog format** — match the existing CHANGELOG.md style if one exists
- **Detect commit style** — if using conventional commits, parse them; otherwise analyze diffs
- **Audience-aware** — use Format B (user-facing) for products, Format A (keep-a-changelog) for libraries, Format C (developer) for internal tools
- **Handle no tags** — if no previous tags exist, use the initial commit as the starting point
- **Handle monorepos** — generate per-package changelogs if changes span multiple packages
- **Respect existing automation** — if tools like standard-version, semantic-release, or changesets are configured, work with them rather than duplicating
- **Include migration steps** — if breaking changes are detected, always include upgrade instructions

## Summary

End every changelog generation with:
1. **Suggested version** — MAJOR/MINOR/PATCH with reasoning
2. **Changelog preview** — the formatted content ready to commit
3. **Breaking changes** — highlighted separately if any exist
4. **Deployment notes** — anything operators need to know
5. **Next steps** — tag, release, publish commands to run
