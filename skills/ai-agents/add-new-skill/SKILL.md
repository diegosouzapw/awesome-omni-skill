---
name: add-new-skill
description: Create a new skill for this repository following the agentskills.io specification with proper structure and packaging
---

# Add New Skill to Skills Repository

This meta skill guides the creation of new skills for this repository, following the [agentskills.io specification](https://agentskills.io/specification).

## Repository Context

This repository houses skills that are:
- Packaged as individual `.zip` artifacts via GitHub Actions
- Structured according to agentskills.io specification
- Consumable by Claude Code, Codex, and other AI tools

## agentskills.io Specification Summary

### Required Structure

Every skill MUST have:
```
skill-name/
└── SKILL.md          # Required file with YAML frontmatter
```

### Optional Directories

Skills MAY include:
```
skill-name/
├── SKILL.md          # Required
├── scripts/          # Executable code (Python, Bash, JavaScript)
├── references/       # Additional docs loaded on-demand
└── assets/           # Templates, images, data files
```

## SKILL.md Format

### Required Frontmatter

```yaml
---
name: skill-name
description: A clear description of what this skill does and when to use it (1-1024 characters).
---
```

### Frontmatter Field Rules

**name** (REQUIRED):
- 1-64 characters
- Lowercase alphanumeric and hyphens only
- Cannot start/end with hyphens
- Cannot contain consecutive hyphens
- MUST match parent directory name

**description** (REQUIRED):
- 1-1024 characters
- Should explain what the skill does and when to use it
- Be specific and actionable

### Optional Frontmatter Fields

```yaml
---
name: skill-name
description: Description here
license: MIT
compatibility: Requires Python 3.8+, works on Linux/macOS
metadata:
  version: "1.0.0"
  author: "Your Name"
allowed-tools: Read Write Bash
---
```

### Content Guidelines

After frontmatter, include:
- **Overview**: What the skill does
- **When to Use**: Clear use cases
- **Step-by-Step Instructions**: Detailed workflow
- **Examples**: Real-world usage patterns
- **Templates**: Output formats or code snippets
- **Validation**: Checklists or verification steps

Keep SKILL.md under 5000 tokens (recommended) for efficient loading.

## Workflow: Adding a New Skill

### 1. Create Skill Directory

```bash
mkdir -p skill-name
```

**CRITICAL**: Directory name MUST match the `name` field in frontmatter.

### 2. Write SKILL.md

Create `skill-name/SKILL.md` with:
- Required YAML frontmatter (name, description)
- Clear markdown instructions
- Examples and templates
- Validation steps

### 3. Add Optional Resources (If Needed)

**scripts/** - Add executable code:
```bash
mkdir -p skill-name/scripts
# Add Python/Bash/JS files
```

**references/** - Add supporting docs:
```bash
mkdir -p skill-name/references
# Add REFERENCE.md, FORMS.md, etc.
```

**assets/** - Add static files:
```bash
mkdir -p skill-name/assets
# Add templates, images, data files
```

### 4. Validate Skill Structure

Check that:
- [ ] Directory name matches frontmatter `name` field
- [ ] Name is lowercase, alphanumeric + hyphens only
- [ ] Name doesn't start/end with hyphens
- [ ] No consecutive hyphens in name
- [ ] Description is 1-1024 characters
- [ ] SKILL.md has valid YAML frontmatter
- [ ] Instructions are clear and actionable

### 5. Test Locally (Optional)

Verify zip structure matches specification:
```bash
cd skill-name
zip -r ../test-skill.zip . -x "*.git*"
cd ..
unzip -l test-skill.zip
# Should show: skill-name/SKILL.md, skill-name/scripts/, etc.
rm test-skill.zip
```

### 6. Commit and Push

```bash
git add skill-name/
git commit -m "feat: add skill-name skill

Brief description of what this skill does.

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
git push
```

### 7. Verify GitHub Actions

After push:
1. Go to Actions tab in GitHub repository
2. Wait for "Package Skills" workflow to complete
3. Check that new skill artifact was created
4. Download and verify zip structure contains parent directory

## GitHub Actions Workflow

The repository includes `.github/workflows/package-skills.yml` which:
- Discovers all directories containing SKILL.md
- Packages each skill with proper directory structure
- Creates individual artifacts per skill (e.g., `skill-name.zip`)
- Runs on push to main, pull requests, and manual dispatch

**Package Structure**: Each zip contains the parent directory:
```
skill-name.zip
└── skill-name/
    ├── SKILL.md
    ├── scripts/
    ├── references/
    └── assets/
```

This matches the agentskills.io specification for tool consumption.

## Examples from This Repository

### Example 1: Infrastructure Automation Skill

**Directory**: `new-homelab-guest/`

**Characteristics**:
- Single SKILL.md file with detailed Terraform workflow
- No scripts (relies on external repository)
- Includes prerequisites, step-by-step instructions
- Has troubleshooting section and examples

**Good for**: Complex multi-step workflows, infrastructure tasks

### Example 2: Data Processing Skill

**Directory**: `digitize-pounce-scoresheet/`

**Characteristics**:
- Single SKILL.md with data extraction workflow
- Includes output format template
- Has validation checklist
- Explains domain-specific context (game rules)
- Provides common patterns guide

**Good for**: Data transformation, format conversion, OCR tasks

### Example 3: Meta Skill (This One!)

**Directory**: `add-new-skill/`

**Characteristics**:
- Documents repository conventions
- Links to external specification
- Provides step-by-step workflow
- Includes validation checklist
- References existing examples

**Good for**: Repository documentation, process guides

## Common Skill Patterns

### Pattern 1: Step-by-Step Workflow
For tasks with clear sequential steps:
```markdown
## Workflow

### 1. Prepare Environment
Instructions here...

### 2. Execute Task
Instructions here...

### 3. Validate Output
Checklist...
```

### Pattern 2: Template-Based Output
For skills that generate structured output:
```markdown
## Output Format Template

```yaml
key: value
nested:
  - item1
  - item2
```
```

### Pattern 3: Context + Instructions
For domain-specific tasks:
```markdown
## Domain Context
Background information...

## Processing Steps
1. Step one
2. Step two

## Validation
Checklist...
```

## Naming Conventions

### Good Skill Names
- `new-homelab-guest` - Action-oriented, clear purpose
- `digitize-pounce-scoresheet` - Verb + specific task
- `add-new-skill` - Clear action and object
- `parse-terraform-state` - Specific transformation
- `generate-api-docs` - Clear output type

### Bad Skill Names
- `Homelab` - Too generic, not action-oriented
- `scoresheet_digitizer` - Uses underscore (use hyphens)
- `new_skill` - Uses underscore
- `my-awesome-skill` - Not descriptive
- `-skill-name-` - Invalid (starts/ends with hyphen)
- `skill--name` - Invalid (consecutive hyphens)

## Tips for Writing Effective Skills

1. **Be Specific**: Describe exactly what the skill does and when to use it
2. **Include Context**: Explain domain-specific concepts needed to execute the task
3. **Provide Examples**: Show real-world usage patterns
4. **Add Validation**: Include checklists or verification steps
5. **Use Templates**: Provide output format templates when applicable
6. **Keep It Focused**: One skill should do one thing well
7. **Consider Progressive Disclosure**: Put essential info in SKILL.md, details in references/
8. **Test Instructions**: Verify steps are complete and executable

## Troubleshooting

### Skill Not Packaged
**Problem**: New skill doesn't appear in GitHub Actions artifacts
**Solution**:
- Verify SKILL.md exists in subdirectory
- Check directory depth (should be 2 levels: `./skill-name/SKILL.md`)
- Review Actions logs for errors

### Invalid Directory Name
**Problem**: Directory name doesn't match frontmatter
**Solution**:
- Rename directory: `git mv old-name new-name`
- Or update frontmatter `name` field to match directory

### Workflow Fails on Push
**Problem**: GitHub Actions fails with zip error
**Solution**:
- Check for special characters in filenames
- Verify YAML frontmatter is valid
- Review workflow logs for specific error

## Progressive Disclosure Strategy

Per agentskills.io specification:

**Tier 1 - Metadata (~100 tokens)**: Loaded at startup
- Just the frontmatter fields

**Tier 2 - SKILL.md (<5000 tokens)**: Loaded when skill is activated
- Full instructions in SKILL.md

**Tier 3 - Supporting files**: Loaded on demand
- Files in scripts/, references/, assets/

Keep SKILL.md concise. Move detailed docs to `references/`, long code to `scripts/`, and data/templates to `assets/`.

## Validation Checklist

Before committing a new skill:
- [ ] Directory name matches frontmatter `name` field exactly
- [ ] Name uses only lowercase, alphanumeric, and hyphens
- [ ] Name doesn't start or end with hyphens
- [ ] Name doesn't contain consecutive hyphens
- [ ] Description is between 1-1024 characters
- [ ] SKILL.md has valid YAML frontmatter (use `---` delimiters)
- [ ] Instructions are clear and actionable
- [ ] Examples or templates are provided where applicable
- [ ] Validation steps or checklists included
- [ ] SKILL.md is under 5000 tokens (if possible)
- [ ] Optional directories (scripts/, references/, assets/) are used appropriately
- [ ] Git commit message follows conventional format

## Resources

- **agentskills.io Specification**: https://agentskills.io/specification
- **agentskills.io Home**: https://agentskills.io/
- **This Repository**: Check existing skills for patterns and examples
- **GitHub Actions Workflow**: `.github/workflows/package-skills.yml`
