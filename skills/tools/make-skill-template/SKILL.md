---
name: make-skill-template
description: Create new Agent Skills for GitHub Copilot from prompts or by duplicating this template. Use when asked to "create a skill", "make a new skill", "scaffold a skill", or when building specialized AI capabilities with bundled resources for OnoCoro. Generates SKILL.md files with proper frontmatter, directory structure, and optional scripts/references/assets folders.
compatibility: Works with GitHub Copilot
---

# Make Skill Template (OnoCoro)

Create new Agent Skills for GitHub Copilot from prompts or by duplicating this template. This template is customized for **OnoCoro specialized capabilities** with bundled resources (scripts, references, guides).

Use when asked to:
- **"Create a skill"** — Build a new Agent Skill from scratch
- **"Make a new skill"** — Scaffold new capability
- **"Build a skill for X"** — Specialized skill for OnoCoro feature
- Building specialized AI capabilities with bundled resources

## What is an Agent Skill?

An Agent Skill is a self-contained folder containing:
- **SKILL.md** — Main instruction file with guidance and examples
- **scripts/** — PowerShell/Python scripts for validation or automation
- **references/** — Supporting documentation, checklists, patterns
- **assets/** — Code samples, templates, configuration files

## Skill Directory Structure

```
.github/skills/[skill-name]/
├── SKILL.md                       # Main skill definition
├── scripts/                       # (Optional) Automation scripts
│   └── validate-skill.ps1
├── references/                    # (Optional) Supporting docs
│   ├── pattern-guide.md
│   ├── checklist.md
│   └── examples.md
└── assets/                        # (Optional) Templates/samples
    ├── template.cs
    └── config.yml
```

## Creating a New OnoCoro Skill

### Step 1: Define the Skill Purpose

```
Skill Name: [Name]
Purpose: [What the skill does]
Target Audience: [AI Agent / Developer]
Use Cases:
  - [Use case 1]
  - [Use case 2]
```

### Step 2: Create Skill Directory

```powershell
# Create skill folder
New-Item -Type Directory -Path ".github/skills/[skill-name]"

# Create SKILL.md
New-Item -Type File -Path ".github/skills/[skill-name]/SKILL.md"

# Create subfolders if needed
New-Item -Type Directory -Path ".github/skills/[skill-name]/scripts"
New-Item -Type Directory -Path ".github/skills/[skill-name]/references"
```

### Step 3: Write SKILL.md

Use this template:

```markdown
---
name: [skill-name]
description: [One-sentence description of what the skill does. Include "OnoCoro" context.]
compatibility: [e.g., "Works with GitHub Copilot", "Requires X MCP Server"]
---

# [Skill Name]

[Purpose and overview]

## When to Use

- [Use case 1]
- [Use case 2]
- [Use case 3]

## Key Features

| Feature | Description |
|---------|-------------|
| [Feature 1] | [Description] |
| [Feature 2] | [Description] |

## Usage Examples

### Example 1

[Describe scenario]

```csharp
// Code example
```

### Example 2

[Describe scenario]

```powershell
# Script example
```

## Best Practices

- [Practice 1]
- [Practice 2]
- [Practice 3]

## Related Documentation

- [AGENTS.md](../../../AGENTS.md)
- [Other docs...]

---

**Last Updated**: 2026-01-20
```

### Step 4: Add Supporting Files (Optional)

**scripts/validate-skill.ps1:**
```powershell
# PowerShell script for validation or automation
# Example: Check for missing configurations

param(
    [string]$ProjectPath = "g:\unity\OnoCoro2026"
)

Write-Host "Validating skill requirements..."

if (-not (Test-Path "$ProjectPath/.github/skills")) {
    Write-Error "Skills folder not found"
    exit 1
}

Write-Host "Validation passed!"
```

**references/pattern-guide.md:**
```markdown
# [Skill Name] Patterns

## Pattern 1

[Description]

```csharp
// Code pattern
```

## Checklist

- [ ] Item 1
- [ ] Item 2
```

**assets/template.cs:**
```csharp
// Template for common code patterns in this skill
```

### Step 5: Document in Skills README

Add entry to [.github/skills/README.md](.github/skills/README.md):

```markdown
| [Skill Name] | [Description] | scripts/ references/ |
|--------------|---------------|---------------------|
```

## OnoCoro Skill Examples

### Recommended Skills to Create

#### 1. `unity-recovery-validator`

**Purpose**: Validate Recovery phase code against AGENTS.md standards

**Files**:
- `SKILL.md` — Validation guidance
- `scripts/validate-recovery.ps1` — Check for null checks, magic numbers
- `references/recovery-checklist.md` — Pre-commit checklist
- `references/recovery-patterns.md` — Approved patterns

#### 2. `plateau-data-processor`

**Purpose**: PLATEAU SDK data processing and validation

**Files**:
- `SKILL.md` — PLATEAU processing guidance
- `references/crs-guide.md` — Coordinate Reference System guide
- `references/mesh-patterns.md` — Mesh generation patterns
- `scripts/validate-plateau.ps1` — Validate geospatial data

#### 3. `prefab-manager-assistant`

**Purpose**: PrefabManager usage and management

**Files**:
- `SKILL.md` — PrefabManager patterns
- `references/prefab-checklist.md` — Integration checklist
- `scripts/find-resources-load.ps1` — Detect Resources.Load calls
- `assets/PrefabManager-template.cs` — Template for new managers

## Skill Best Practices

### Documentation

- Clear, actionable instructions
- Real OnoCoro code examples
- Links to [AGENTS.md](../../../AGENTS.md) and other guidelines
- Before/after examples showing improvement

### Scripts

- Make idempotent (safe to run multiple times)
- Include error checking
- Output clear success/failure messages
- Use PowerShell 5.1 (Windows compatibility)

### References

- Provide checklists for validation
- Include pattern examples
- Link to official documentation
- Keep focused on one topic per file

### Organization

- One skill per feature/capability
- Self-contained (don't reference other skills)
- Include LICENSE.txt for clarity
- Add Last Updated date to SKILL.md

## Skill Lifecycle

1. **Create** — Use this template
2. **Test** — Verify with example code
3. **Document** — Add to .github/skills/README.md
4. **Integrate** — Reference in .github/copilot/README.md
5. **Maintain** — Update as standards evolve

## Related Documentation

- **awesome-copilot Skills**: https://github.com/github/awesome-copilot/tree/main/skills
- **Agent Skills Spec**: https://agentskills.io/specification
- **OnoCoro AGENTS.md**: [AGENTS.md](../../../AGENTS.md)
- **OnoCoro Instructions**: [.github/instructions/](../../instructions/)

---

**Last Updated**: 2026-01-20
**Template Version**: 1.0
