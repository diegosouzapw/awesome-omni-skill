---
name: ash-library-hotfix
description: Handles emergency hotfix process for critical bugs in ash_cookie_consent library including branch creation, minimal fixes, testing, and rapid release. Use when user asks to "create hotfix", "emergency fix", "patch critical bug", or "hotfix for version".
---

# AshCookieConsent Emergency Hotfix

This skill guides you through creating and releasing an emergency hotfix for critical bugs.

## When to Use This Skill

Activate when user requests:
- "Create hotfix for vX.Y.Z"
- "Emergency fix for critical bug"
- "Patch version X.Y.Z"
- "Hotfix for security issue"

## Hotfix Criteria

Only use hotfix process for:
- **Critical bugs** affecting production users
- **Security vulnerabilities** requiring immediate patch
- **Data loss/corruption** issues
- **Application crashes** or unavailability

For non-critical bugs, use normal release process.

## Hotfix Workflow

### Step 1: Verify Critical Nature

Ask user to confirm:
- What is the bug?
- Who is affected?
- What is the impact?
- Is this truly critical?

If not critical, recommend normal release process instead.

### Step 2: Create Hotfix Branch

```bash
# Get the version to hotfix
git fetch --tags

# Create hotfix branch from the tag
git checkout -b hotfix/X.Y.Z vX.Y.Z

# Example: git checkout -b hotfix/0.1.1 v0.1.0
```

### Step 3: Make Minimal Fix

**Guidelines**:
- Fix ONLY the critical issue
- No refactoring or cleanup
- No unrelated changes
- Add regression test to prevent recurrence

**Update CHANGELOG.md**:
```markdown
## [X.Y.Z+1] - YYYY-MM-DD

### Fixed
- **CRITICAL**: Description of bug and fix (#issue)
```

### Step 4: Bump PATCH Version

Edit `mix.exs`:
```elixir
@version "X.Y.Z+1"  # Increment patch number
```

### Step 5: Test Thoroughly

```bash
# Run full test suite
mix test

# Verify the specific bug is fixed
# Add manual verification steps here

# Run quality checks (if time permits)
mix credo
mix dialyzer
```

**Minimum requirement**: Tests must pass. Other checks optional if time-critical.

### Step 6: Commit and Tag

```bash
git add .
git commit -m "Fix critical bug: [brief description] (#issue)"

git tag -a vX.Y.Z+1 -m "Hotfix vX.Y.Z+1"
```

### Step 7: Merge to Main

```bash
git checkout main
git merge hotfix/X.Y.Z+1 --no-ff

# Resolve conflicts if any (prioritize hotfix changes)

git push origin main
git push origin vX.Y.Z+1
```

### Step 8: Publish Immediately

```bash
mix hex.build
mix hex.publish
```

Confirm with `y` when prompted.

### Step 9: Retire Vulnerable Version (If Security Issue)

```bash
mix hex.retire ash_cookie_consent X.Y.Z \
  --reason security \
  --message "Security vulnerability fixed in vX.Y.Z+1. Please upgrade immediately."
```

### Step 10: Create GitHub Release

```bash
gh release create vX.Y.Z+1 \
  --title "vX.Y.Z+1 - HOTFIX: Brief Description" \
  --notes "**Critical hotfix for vX.Y.Z**

🚨 This release fixes a critical bug affecting [description].

### Fixed
- [Description of fix] (#issue)

### Upgrade Instructions
Update your mix.exs:
\`\`\`elixir
{:ash_cookie_consent, \"~> X.Y.Z+1\"}
\`\`\`

See [CHANGELOG](link) for full details."
```

### Step 11: Notify Users

**Immediate notification for**:
- Security vulnerabilities
- Data loss issues
- Breaking bugs

**Notification channels**:
1. Update Hex.pm package page (automatic)
2. Post on Elixir Forum if widely used
3. Update GitHub README with notice
4. Email known users if contact list exists
5. Post on Ash Discord

**Template**:
```markdown
🚨 **Hotfix Released: v X.Y.Z+1**

A critical bug was discovered in v X.Y.Z: [description]

**Impact**: [Who is affected and how]

**Fix**: Upgrade to v X.Y.Z+1 immediately

**Update command**:
\`\`\`bash
mix deps.update ash_cookie_consent
\`\`\`

See [release notes](link) for details.
```

## Post-Hotfix Cleanup

After releasing hotfix:

1. **Delete hotfix branch** (optional):
   ```bash
   git branch -d hotfix/X.Y.Z+1
   ```

2. **Verify fix in production**:
   - Monitor for new reports
   - Check Hex.pm download stats
   - Watch GitHub issues

3. **Update roadmap** (if needed):
   - Adjust next release timeline
   - Document lessons learned

4. **Follow-up tasks**:
   - Add to regression test suite
   - Review related code for similar issues
   - Update documentation if needed

## Time Targets

**For critical security issues**:
- Fix: < 4 hours
- Release: < 6 hours
- Notification: < 8 hours

**For critical bugs**:
- Fix: < 24 hours
- Release: < 36 hours
- Notification: < 48 hours

## Verification Checklist

Before publishing hotfix:

- [ ] Bug confirmed fixed
- [ ] Tests pass (minimum: related tests)
- [ ] No new bugs introduced
- [ ] Version bumped (PATCH only)
- [ ] CHANGELOG updated
- [ ] Commit message clear
- [ ] Git tag created
- [ ] Merged to main

## Common Hotfix Scenarios

### Security Vulnerability

```markdown
## [0.1.1] - 2025-11-XX

### Security
- **CRITICAL**: Fix XSS vulnerability in consent modal (#42)
  - Escape user-provided cookie group names
  - Add Content-Security-Policy headers
```

### Data Loss Bug

```markdown
## [0.1.1] - 2025-11-XX

### Fixed
- **CRITICAL**: Fix consent data being lost on browser restart (#43)
  - Correct cookie max-age calculation
  - Add migration for existing cookies
```

### Application Crash

```markdown
## [0.1.1] - 2025-11-XX

### Fixed
- **CRITICAL**: Fix crash when consent data malformed (#44)
  - Add validation and error handling
  - Gracefully handle legacy cookie format
```

## When NOT to Use Hotfix

Use normal release process instead if:
- Bug is not critical (doesn't affect production)
- Fix requires significant changes
- No users reported the issue yet
- Fix can wait for next planned release

## Reference

Full details: `.claude/MAINTAINER_GUIDE.md` - "Hotfix Process" section
