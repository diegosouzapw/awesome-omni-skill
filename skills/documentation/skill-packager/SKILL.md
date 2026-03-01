---
name: skill-packager
description: Package custom skills into versioned distributable ZIP files with installation and user documentation. Use when preparing a skill for sharing, distribution, or archival. Supports semantic versioning, automatic documentation generation, and version management.
version: 1.0.0
license: Complete terms in LICENSE.txt
---

# Skill Packager

Package custom Claude Code skills into distributable ZIP files with comprehensive documentation and semantic versioning support.

## Overview

This skill automates the process of packaging custom skills for distribution. It validates the skill structure, generates installation and user documentation, creates versioned ZIP archives, and manages version increments according to semantic versioning principles.

## When to Use This Skill

Invoke this skill when you need to:

- Package a skill for sharing with other users
- Create a versioned release of a skill
- Generate installation and usage documentation
- Archive skill versions for distribution
- Update an existing skill package with a new version

## Workflow

### 1. Skill Selection and Validation

**Action:** Prompt the user to provide the path to the skill directory or paste the SKILL.md content.

**Validation Steps:**

- Verify SKILL.md exists in the skill directory
- Validate YAML frontmatter is present and properly formatted
- Confirm required fields exist: `name` and `description`
- Check naming conventions (hyphen-case, lowercase, max 40 chars)
- Verify no angle brackets in description
- Check for version field in frontmatter (add if missing, default to 0.1.0)

**Error Handling:**

- If validation fails, report specific errors and stop
- Suggest corrections for common issues
- Offer to fix minor issues (e.g., add missing version field)

### 2. Download Directory Selection

**Prompt for Download Location:**
Ask the user where to save the packaged skill:

- **Default:** `{current-working-directory}/downloads/{skill-name}/`
- Display the full path that will be used
- Allow user to accept default or specify custom base directory
- Validate that the path is writable and accessible

**Directory Structure:**

- The base directory provided by user (or default)
- Skill-specific subdirectory is always appended: `{base-dir}/{skill-name}/`
- Final structure: `{base-dir}/{skill-name}/{skill-name}-v{version}.zip`

**Validation:**

- Ensure directory path is valid
- Check write permissions
- Create directory if it doesn't exist (with user confirmation)
- Warn if directory already contains previous versions (this is expected/normal)

### 3. Version Management

**Read Current Version:**

- Extract current version from SKILL.md frontmatter
- Parse as semantic version (MAJOR.MINOR.PATCH)
- Display current version to user
- If no version exists, default to 0.1.0

**Determine New Version (Hybrid Approach):**

The script supports both auto-increment and explicit version specification:

**Auto-Increment (Default Behavior):**
- **Default: Minor increment** - Automatically bumps minor version (0.1.0 → 0.2.0)
- No `--version` flag needed
- Best for most releases (new features, backward-compatible changes)

Ask the user to choose bump type:
- **Patch** (`--bump-type patch`) - Bug fixes, documentation updates (0.1.0 → 0.1.1)
- **Minor** (`--bump-type minor`) - New features, backward-compatible changes (0.1.0 → 0.2.0) [DEFAULT]
- **Major** (`--bump-type major`) - Breaking changes, major refactor (0.1.0 → 1.0.0) [REQUIRES CONFIRMATION]

**Major Version Bump Safeguard:**
Before allowing a major version bump, ask the user:
> "This will create a MAJOR version bump (breaking change). Are you sure this release includes breaking changes or major refactoring?"

Only proceed with `--bump-type major` if user confirms.

**Explicit Version Override:**
For custom version numbers, use `--version X.Y.Z` flag:
- Bypasses auto-increment logic
- Useful for version corrections or specific numbering requirements
- Example: `--version 2.0.0`

**Version Validation:**

- Ensure format is MAJOR.MINOR.PATCH (e.g., 1.2.3)
- Warn if new version is lower than current version
- Confirm with user if version seems unusual

### 4. Documentation Generation

**Generate Download.md (Installation Instructions):**

Use the template at `templates/download_template.md` to create skill-specific installation documentation including:

- Skill name and version
- Prerequisites and requirements
- Installation steps (extract ZIP, copy to skills directory)
- Verification steps (check skill appears in available skills)
- Compatibility notes (Claude Code version, OS compatibility)
- Next steps and getting started

**Generate {SkillName}-doc.md (User Documentation):**

Use the template at `templates/doc_template.md` to create user-facing documentation including:

- Skill overview and purpose
- When to use this skill
- Usage instructions and workflow
- Examples and common use cases
- Troubleshooting tips
- Reference to bundled resources (scripts/, references/)

**Template Variable Substitution:**

- `{{SKILL_NAME}}` - Skill name from frontmatter
- `{{SKILL_VERSION}}` - Current version being packaged
- `{{SKILL_DESCRIPTION}}` - Description from frontmatter
- `{{INSTALLATION_DATE}}` - Current date in ISO format
- `{{SKILL_DIR_NAME}}` - Directory name of the skill

### 5. Package Creation

**Prepare Package Directory:**

- Use the download directory chosen in step 2 (or default: `{current-working-directory}/downloads/`)
- Create base directory if it doesn't exist
- Create skill-specific subdirectory: `{base-dir}/{skill-name}/`

**Execute Packaging Script:**

Call `scripts/package_and_document.py` with parameters:

**Auto-increment (recommended):**
```bash
# Default: minor version bump (0.1.0 → 0.2.0)
python scripts/package_and_document.py \
  --skill-path /path/to/skill \
  --output-dir {base-dir}/{skill-name}

# Patch version bump (0.1.0 → 0.1.1)
python scripts/package_and_document.py \
  --skill-path /path/to/skill \
  --bump-type patch \
  --output-dir {base-dir}/{skill-name}

# Major version bump (0.1.0 → 1.0.0) - requires confirmation
python scripts/package_and_document.py \
  --skill-path /path/to/skill \
  --bump-type major \
  --output-dir {base-dir}/{skill-name}
```

**Explicit version override:**
```bash
python scripts/package_and_document.py \
  --skill-path ~/.claude/skills/my-skill \
  --version 1.0.0 \
  --output-dir ./downloads/my-skill
```

**Script Responsibilities:**

- Validate skill structure (reuse validation from step 1)
- Create ZIP file with versioned name: `{skill-name}-v{version}.zip`
- Include all skill files: SKILL.md, scripts/, references/, assets/, LICENSE.txt, etc.
- Exclude unnecessary files: .DS_Store, **pycache**, *.pyc, .git
- Maintain directory structure within ZIP
- Generate checksums (SHA-256) for integrity verification

**Output Files:**

```text
{base-dir}/{skill-name}/
├── {skill-name}-v1.0.0.zip
├── {skill-name}-v1.1.0.zip (if previous versions exist)
├── {skill-name}-Download.md (updated with latest version)
└── {skill-name}-doc.md (updated with latest version)
```

**Example with default directory:**

```text
./downloads/my-skill/
├── my-skill-v1.0.0.zip
├── my-skill-Download.md
└── my-skill-doc.md
```

### 6. Update Skill Metadata

**Update SKILL.md Frontmatter:**

- Replace version field with new version number
- Preserve all other frontmatter fields
- Maintain formatting and structure

**Example Update:**

```yaml
# Before
---
name: my-skill
description: Does something useful
version: 0.1.0
---

# After
---
name: my-skill
description: Does something useful
version: 1.0.0
---
```

### 7. Package Summary and Next Steps

**Display Summary:**

```text
✓ Skill packaged successfully!

Package Details:
- Skill: my-skill
- Version: v1.0.0
- Location: {base-dir}/my-skill/

Generated Files:
- my-skill-v1.0.0.zip (2.3 MB)
- my-skill-Download.md (installation guide)
- my-skill-doc.md (user documentation)

Updated:
- SKILL.md frontmatter (version: 1.0.0)

Previous Versions:
- v0.1.0 (still available)
```

**Next Steps:**

- Share the ZIP file with others
- Distribute Download.md for installation instructions
- Provide doc.md for usage guidance
- Consider publishing to a skill registry (if available)

## Progressive Disclosure: Bundled Resources

### Scripts

#### package_and_document.py

- Main packaging automation script
- Handles ZIP creation, validation, and file organization
- Generates documentation from templates
- Manages version numbering in filenames
- Called automatically during workflow step 4

### Templates

#### download_template.md

- Template for generating installation instructions
- Loaded and processed during step 3
- Contains placeholders for skill-specific information

#### doc_template.md

- Template for generating user documentation
- Loaded and processed during step 3
- Extracts information from SKILL.md and resources

### References

#### versioning-guide.md

- Detailed semantic versioning guidelines for skills
- When to use MAJOR vs MINOR vs PATCH increments
- Examples and best practices
- Load when user needs guidance on version selection

## Error Handling and Edge Cases

### Missing Version Field

- Offer to add version field to SKILL.md
- Suggest starting at 0.1.0 for initial packages
- Update frontmatter before packaging

### Duplicate Version

- Warn if version already exists in downloads directory
- Offer to increment automatically or overwrite
- Confirm action with user

### Invalid Skill Structure

- Report validation errors clearly
- Suggest fixes based on skill-creator guidelines
- Do not proceed with packaging until fixed

### Large File Sizes

- Warn if ZIP exceeds 10MB (may indicate unnecessary files)
- Suggest checking for .git directories, node_modules, etc.
- Offer to show file size breakdown

### Missing Templates

- Check for template files before starting
- Provide fallback generic templates if missing
- Warn user that documentation may be basic

## Integration with Existing Tools

### Reuse from skill-creator

- Validation logic from `quick_validate.py`
- Directory structure conventions
- Naming pattern enforcement

### Complement to package_skill.py

- Extends basic packaging with versioning
- Adds documentation generation
- Provides centralized distribution directory
- Maintains version history

## Success Criteria

A successful packaging operation results in:

1. ✓ Valid semantic version in SKILL.md frontmatter
2. ✓ Versioned ZIP file in skill-specific downloads directory
3. ✓ Installation instructions (Download.md) generated
4. ✓ User documentation ({SkillName}-doc.md) generated
5. ✓ All previous versions preserved
6. ✓ Package ready for distribution

## Example Invocation

User: "Package the my-data-analyzer skill"

Skill Response:

1. Locates skill at `~/.claude/skills/my-data-analyzer/`
2. Validates structure and reads current version (0.2.3)
3. Asks: "Where should I save the package? Default: ./downloads/my-data-analyzer/"
4. User accepts default or provides custom path
5. Asks: "Current version is 0.2.3. What version action? (keep/patch/minor/major/custom)"
6. User selects "minor" → new version 0.3.0
7. Generates my-data-analyzer-Download.md and my-data-analyzer-doc.md
8. Creates my-data-analyzer-v0.3.0.zip in chosen directory
9. Updates SKILL.md frontmatter with version: 0.3.0
10. Reports success with full package location path
