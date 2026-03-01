---
name: skill-creator
description: Create and scaffold new agent skills with proper structure, validation, and spec compliance. Use when building new skills from scratch.
license: MIT
---

# Skill Creator

Create well-structured agent skills that comply with the Agent Skills Specification v1.0.

## When to Use This Skill

Use skill-creator when you need to:

- Create a new skill from scratch
- Scaffold the directory structure for a skill
- Generate valid SKILL.md with proper YAML frontmatter
- Ensure spec compliance from the start
- Validate an existing skill's structure

## Skill Creation Process

### Phase 1: Planning

#### Gather Requirements

1. Ask for the skill's purpose and scope
2. Determine if scripts, templates, or assets are needed
3. Identify dependencies (Python packages, npm modules)
4. Clarify when the skill should be used

#### Key Questions

- What specific task does this skill address?
- What are the inputs and expected outputs?
- Does it need executable code (scripts)?
- Does it need templates or reference materials?
- What tools will it use?

### Phase 2: Naming and Structure

#### Generate Skill Name

1. Use hyphen-case format (lowercase, hyphens only)
2. Maximum 64 characters
3. Descriptive and specific
4. Examples: `code-complexity-analyzer`, `api-doc-generator`

#### Invalid Names

- ❌ `Code_Analyzer` (underscores)
- ❌ `codeAnalyzer` (camelCase)
- ❌ `CODE-ANALYZER` (uppercase)
- ✓ `code-analyzer` (correct)

#### Determine Directory Structure

Minimal (instructions only):

```text
skill-name/
└── SKILL.md
```

With scripts:

```text
skill-name/
├── SKILL.md
└── scripts/
    └── helper.py
```

Full structure:

```text
skill-name/
├── SKILL.md
├── scripts/
│   └── helper.py
├── templates/
│   └── template.md
├── assets/
│   └── output/
└── references/
    └── docs.md
```

### Phase 3: Generate SKILL.md

#### Create YAML Frontmatter

Required fields:

```yaml
---
name: skill-name
description: Clear explanation of what the skill does and when Claude should use it
---
```

With optional fields:

```yaml
---
name: skill-name
description: Clear explanation of what the skill does and when Claude should use it
license: MIT
allowed-tools:
  - Read
  - Write
  - Bash
metadata:
  author: username
  version: "1.0.0"
---
```

#### Description Guidelines

- ~200 characters (max 1024)
- Explain WHAT the skill does
- Explain WHEN to use it
- Be specific and actionable
- Example: "Analyze code complexity using cyclomatic complexity metrics. Use when assessing code maintainability or identifying refactoring candidates."

#### Write Skill Instructions

Use imperative language:

```markdown
# Skill Name

Brief introduction to what this skill does.

## When to Use This Skill

List specific scenarios...

## Process

### Step 1: Action

Instructions in imperative form...

### Step 2: Action

More instructions...

## Examples

### Example 1: Concrete Scenario

Input: ...
Expected Output: ...
```

#### Best Practices

- Keep under 5,000 words
- Use concrete examples
- Reference bundled resources by relative path (e.g., templates/my-template.md)
- Focus on procedural knowledge
- Avoid redundancy

### Phase 4: Create Supporting Files

#### Scripts

- Place in `scripts/` directory
- Include usage instructions in SKILL.md
- Remind users to run `script --help` first
- Treat scripts as black boxes

Example reference in SKILL.md:

```markdown
Run the validation script:
\`\`\`bash
python scripts/validate.py --help
python scripts/validate.py --input data.json
\`\`\`
```

#### Templates

- Place in `templates/` directory
- Reference from SKILL.md
- Provide clear descriptions of when to use each template

#### Assets

- Use for output files or resources
- Create subdirectories for organization

### Phase 5: Validation

#### Run Validation Checks

1. **Name validation**
   - Directory name matches YAML `name` field exactly
   - Hyphen-case format
   - ≤64 characters
   - Only lowercase Unicode alphanumeric and hyphens

2. **YAML validation**
   - Valid YAML syntax
   - Required fields present: `name`, `description`
   - Field constraints met

3. **File reference validation**
   - All referenced files exist
   - Paths are correct relative to skill root

4. **Content validation**
   - Description explains what AND when
   - Instructions are imperative
   - Examples are concrete
   - Word count reasonable (<5,000)

#### Use Validation Script

```bash
python scripts/validate_skill.py /path/to/skill-name
```

### Phase 6: Documentation

#### In SKILL.md, Include

- Clear title and introduction
- "When to Use This Skill" section
- Step-by-step process
- At least 2 concrete examples
- Dependencies clearly stated
- References to bundled resources

#### Avoid

- Redundant explanations
- Vague instructions
- Implementation details (focus on what to do, not how)
- Hardcoded credentials or secrets

## Templates

Use these templates as starting points:

### Basic Skill Template

```bash
cat templates/basic-skill.md
```

### Skill with Scripts Template

```bash
cat templates/skill-with-scripts.md
```

## Validation Script

Validate a skill's structure:

```bash
python scripts/validate_skill.py /path/to/skill-directory
```

The script checks:

- YAML frontmatter validity
- Name format and matching
- Required fields present
- File references exist
- Common anti-patterns

## Examples

### Example 1: Simple Instruction-Only Skill

**Input**: "Create a skill for applying brand guidelines"

**Steps**:

1. Name: `brand-guidelines`
2. Structure: Minimal (SKILL.md only)
3. SKILL.md:

```yaml
---
name: brand-guidelines
description: Apply brand visual identity guidelines including colors, typography, and spacing. Use when creating branded materials or reviewing designs.
license: MIT
---

# Brand Guidelines

Apply consistent brand visual identity.

## Colors
- Primary: #1A1A1A
- Secondary: #4A90E2

## Typography
- Headings: Inter, 24px, 600 weight
- Body: Inter, 16px, 400 weight

## Spacing
- Base unit: 8px
- Margins: 16px, 24px, 32px
```

### Example 2: Skill with Scripts

**Input**: "Create a skill for testing web applications with Playwright"

**Steps**:

1. Name: `webapp-testing`
2. Structure: With scripts
3. Create helper script in scripts/ directory
4. SKILL.md references the script with usage examples showing --help flag and actual invocation

## Common Pitfalls

- **Name mismatch**: Directory name must match YAML `name` field exactly
- **Vague description**: Must explain both WHAT and WHEN
- **Missing examples**: Always include concrete examples
- **Too broad**: One skill, one focused task
- **Broken references**: Validate all file paths
- **Hardcoded secrets**: Never include credentials

## Validation Checklist

Before finalizing a skill:

- [ ] Directory name is hyphen-case, ≤64 chars
- [ ] Directory name matches YAML `name` field
- [ ] YAML frontmatter is valid
- [ ] Description explains WHAT and WHEN (~200 chars)
- [ ] Instructions are imperative
- [ ] At least 2 concrete examples provided
- [ ] All file references exist
- [ ] No hardcoded credentials
- [ ] SKILL.md under 5,000 words
- [ ] Dependencies clearly stated

## Resources

- Agent Skills Specification: <https://github.com/anthropics/skills/blob/main/agent_skills_spec.md>
- Validation script: `scripts/validate_skill.py`
- Templates: `templates/` directory
