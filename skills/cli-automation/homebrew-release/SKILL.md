---
name: homebrew-release
description: Create a new version release for a Homebrew formula in the homebrew-tools tap. Handles git tagging, GitHub releases, SHA256 computation, and formula updates. Use when the user wants to publish a new version of a tool in their Homebrew tap.
---

# Homebrew Formula Release

Automate the complete release process for Homebrew formulae in the `homebrew-tools` tap.

## When to Use This Skill

Use this skill when:
- User wants to publish a new version of a tool (e.g., "release v0.2.0 of mistral-ocr")
- User wants to create a new release after making changes to a tool
- User says "create a release", "publish new version", "bump version", etc.
- Changes have been committed to main and are ready for release

**Do NOT use this skill when:**
- User just wants to commit changes (use regular git workflow)
- User wants to update their local installation (use `brew upgrade`)
- No formula exists yet (tool needs to be added to tap first)

## Prerequisites

Before running this skill, verify:
1. User is in the `homebrew-tools` repository directory
2. Changes are committed to the main branch
3. User has `gh` CLI installed and authenticated (`gh auth status`)
4. Formula file exists in `Formula/` directory

## Workflow

### Step 1: Gather Information

Ask the user for required information:

```bash
# Identify the formula to release
# Example questions:
# - "Which tool are you releasing?" (e.g., mistral-ocr)
# - "What version number?" (e.g., v0.2.0, must start with 'v')
# - "Release notes / changelog?" (brief description of changes)
```

**Validation:**
- Version must follow format: `v{major}.{minor}.{patch}` (e.g., v0.1.1, v1.0.0)
- Formula file must exist at `Formula/{tool-name}.rb`
- Current directory must be a git repository

**Tag Naming Convention:**
Since the homebrew-tools tap contains multiple tools with independent version lifecycles, use tool-specific tags:
- Format: `{tool-name}-v{version}`
- Example: `mistral-ocr-v0.1.1`, `tool2-v1.0.0`
- This allows each tool to have its own version history
- Alternative: Use simple `v{version}` tags if releasing the entire tap as a single versioned unit (less common)

### Step 2: Create Git Tag and Push

```bash
# Create annotated tag with tool-specific version
git tag -a {tool-name}-v{version} -m "{tool-name} v{version}"

# Push tag to GitHub
git push origin {tool-name}-v{version}
```

**Error Handling:**
- If tag already exists: Ask user if they want to delete and recreate
- If push fails: Check git remote configuration and authentication

### Step 3: Create GitHub Release

```bash
# Create release using gh CLI with tool-specific tag
gh release create {tool-name}-v{version} \
  --title "{tool-name} v{version}" \
  --notes "{release-notes}"
```

**Example release notes format:**
```
Fix CLI help text to show correct command name
- Update all references from 'python script.py' to 'tool-name'
- Improve error messages
```

**Error Handling:**
- If `gh` not installed: Provide installation instructions
- If authentication fails: Run `gh auth login`
- If release already exists: Ask to delete or skip

### Step 4: Download Release Tarball and Compute SHA256

```bash
# Download the auto-generated tarball from GitHub using tool-specific tag
curl -L https://github.com/mhismail3/homebrew-tools/archive/refs/tags/{tool-name}-v{version}.tar.gz -o {tool-name}-v{version}.tar.gz

# Compute SHA256 checksum
shasum -a 256 {tool-name}-v{version}.tar.gz
```

**Expected output format:**
```
{64-character-hex-hash}  {tool-name}-v{version}.tar.gz
```

**Important:**
- Extract ONLY the hash (first 64 characters before the space)
- Store this for formula update in next step

### Step 5: Update Formula File

Update the formula file at `Formula/{tool-name}.rb`:

**Find and replace:**
```ruby
# OLD
url "https://github.com/mhismail3/homebrew-tools/archive/refs/tags/{tool-name}-v{old-version}.tar.gz"
sha256 "{old-sha256}"

# NEW
url "https://github.com/mhismail3/homebrew-tools/archive/refs/tags/{tool-name}-v{new-version}.tar.gz"
sha256 "{new-sha256}"
```

**Important:**
- Update BOTH the URL and SHA256
- Keep all other formula content unchanged
- Preserve exact Ruby formatting and indentation

### Step 6: Clean Up and Commit

```bash
# Remove downloaded tarball
rm -f {tool-name}-v{version}.tar.gz

# Stage formula changes
git add Formula/{tool-name}.rb

# Commit with clear message
git commit -m "Bump {tool-name} to v{version}

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"

# Push to GitHub
git push
```

### Step 7: Verify Release

After pushing, verify the release is available:

```bash
# Update Homebrew tap
brew update

# Check formula info shows new version
brew info mhismail3/tools/{tool-name}
```

**Expected output should show:**
- New version number
- Correct SHA256
- Formula is installable

## Example Usage

### User Request:
"Release v0.1.1 of mistral-ocr with the help text fixes"

### Skill Execution:

1. **Gather info:**
   - Tool: mistral-ocr
   - Version: v0.1.1
   - Tag: mistral-ocr-v0.1.1
   - Notes: "Fix CLI help text to show 'mistral-ocr' instead of 'python ocr.py'"

2. **Create tag:**
   ```bash
   git tag -a mistral-ocr-v0.1.1 -m "mistral-ocr v0.1.1"
   git push origin mistral-ocr-v0.1.1
   ```

3. **Create GitHub release:**
   ```bash
   gh release create mistral-ocr-v0.1.1 \
     --title "mistral-ocr v0.1.1" \
     --notes "Fix CLI help text to show 'mistral-ocr' instead of 'python ocr.py'"
   ```

4. **Get SHA256:**
   ```bash
   curl -L https://github.com/mhismail3/homebrew-tools/archive/refs/tags/mistral-ocr-v0.1.1.tar.gz -o mistral-ocr-v0.1.1.tar.gz
   shasum -a 256 mistral-ocr-v0.1.1.tar.gz
   # Output: 64046b76c347bc1cfc69acd932725ac74ca10b776d6aa36b3795ee88a034cfd6
   ```

5. **Update formula:**
   ```ruby
   # In Formula/mistral-ocr.rb
   url "https://github.com/mhismail3/homebrew-tools/archive/refs/tags/mistral-ocr-v0.1.1.tar.gz"
   sha256 "64046b76c347bc1cfc69acd932725ac74ca10b776d6aa36b3795ee88a034cfd6"
   ```

6. **Commit and push:**
   ```bash
   rm -f mistral-ocr-v0.1.1.tar.gz
   git add Formula/mistral-ocr.rb
   git commit -m "Bump mistral-ocr to v0.1.1"
   git push
   ```

7. **Verify:**
   ```bash
   brew update
   brew info mhismail3/tools/mistral-ocr
   ```

## Error Recovery

### Tag Already Exists
```bash
# Delete local tag
git tag -d {tool-name}-v{version}

# Delete remote tag
git push origin :refs/tags/{tool-name}-v{version}

# Recreate tag
git tag -a {tool-name}-v{version} -m "{tool-name} v{version}"
git push origin {tool-name}-v{version}
```

### Release Already Exists
```bash
# Delete existing release
gh release delete {tool-name}-v{version} --yes

# Recreate release
gh release create {tool-name}-v{version} --title "{tool-name} v{version}" --notes "..."
```

### Wrong SHA256
```bash
# Re-download tarball (GitHub may have cached old version)
curl -L https://github.com/mhismail3/homebrew-tools/archive/refs/tags/{tool-name}-v{version}.tar.gz -o {tool-name}-v{version}.tar.gz

# Recompute
shasum -a 256 {tool-name}-v{version}.tar.gz
```

### Formula Update Failed
- Verify formula file path is correct
- Check Ruby syntax with `ruby -c Formula/{tool-name}.rb`
- Ensure URL and SHA256 are on separate lines
- Preserve exact indentation (2 spaces in Homebrew formulae)

## Post-Release Checklist

After successful release, inform the user:

1. ✅ Version v{version} released to GitHub
2. ✅ Formula updated with new SHA256
3. ✅ Changes pushed to homebrew-tools repository
4. 📝 Users can now install with: `brew upgrade {tool-name}`
5. 📝 New installations will automatically get v{version}

## Notes

- **Do NOT include** steps for updating the user's local installation - they can do that separately with `brew upgrade`
- Always use annotated tags (`git tag -a`) not lightweight tags
- SHA256 must be exactly 64 hexadecimal characters
- GitHub auto-generates release tarballs from tags - we don't manually upload anything
- The formula URL should always point to `/archive/refs/tags/v{version}.tar.gz`
- Users can check their tap's status with: `brew tap-info mhismail3/tools`

## Troubleshooting

### "gh: command not found"
```bash
# Install GitHub CLI
brew install gh

# Authenticate
gh auth login
```

### "permission denied" on git push
```bash
# Check authentication
gh auth status

# Or check SSH keys
ssh -T git@github.com
```

### Formula won't install after update
```bash
# Test formula syntax
brew audit --strict mhismail3/tools/{tool-name}

# Try installing from source
brew install --build-from-source mhismail3/tools/{tool-name}
```

### SHA256 mismatch during user installation
- Verify the SHA256 was computed from the correct tarball
- Ensure GitHub release was created successfully
- Check that the tag points to the correct commit
- Re-download the tarball and recompute SHA256
