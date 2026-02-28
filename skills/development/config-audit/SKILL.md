---
name: config-audit
description: This skill should be used when auditing or comparing Claude Code and Cursor IDE configurations to identify feature gaps, equivalencies, and migration opportunities. Useful when managing AI development tooling across both platforms or deciding how to structure AI workflows.
---

# Config Audit

Audit and compare Claude Code and Cursor IDE configurations to understand feature coverage, identify gaps, and make informed decisions about AI development tooling.

## Overview

This skill helps analyze the `claude/` and `cursor/` directories in projects to:

1. **Inventory Resources** - Catalog all Claude Code commands, skills, agents and Cursor rules, commands
2. **Identify Gaps** - Find resources in one tool without equivalents in the other
3. **Generate Reports** - Produce structured comparisons and actionable suggestions
4. **Inform Decisions** - Understand feature differences to guide tool selection and migration

**When to use**: When managing AI configurations across Claude Code and Cursor, planning migrations, or deciding where to implement new workflows.

## Quick Start

Run the audit command to generate a comprehensive report:

```bash
ai skill config-audit run
```

This scans the current project's `claude/` and `cursor/` directories and outputs a detailed report showing:

- All Claude Code resources (commands, skills, agents)
- All Cursor resources (rules, commands)
- Gap analysis (resources unique to each tool)
- Suggestions for creating equivalents

## Using the Audit Command

### Basic Usage

```bash
# Audit current project
ai skill config-audit run

# Audit specific project
ai skill config-audit run -p /path/to/project

# Get JSON output for programmatic processing
ai skill config-audit run --json
```

### Understanding the Report

The audit report contains four main sections:

#### 1. Claude Code Resources

Lists all resources found in the `claude/` directory:

- **Commands** - User-invoked slash commands (`.claude/commands/`)
- **Skills** - Model-invoked capabilities (`.claude/skills/`)
- **Agents** - Specialized sub-agents (`.claude/agents/`)

Each resource shows:

- Name
- Type
- Description (if available)

#### 2. Cursor Resources

Lists all resources found in the `cursor/` directory:

- **Rules** - AI instructions in `.mdc` format, categorized by type:
  - **Always** - Always loaded (`alwaysApply: true`)
  - **Auto Attached** - Loaded when file globs match
  - **Agent Requested** - AI decides based on description
  - **Manual** - User invokes with `@ruleName`
- **Commands** - Markdown prompt files

Each rule shows:

- Name
- Type (always/auto-attached/agent-requested/manual)
- Description
- Globs (for auto-attached rules)

#### 3. Gap Analysis

Identifies resources that exist in one tool but not the other:

- **Claude Code Without Cursor Equivalents** - Features unique to Claude Code
- **Cursor Without Claude Code Equivalents** - Features unique to Cursor

Resources are matched by name similarity, so a Claude Code skill named "typescript-standards" and a Cursor rule named "typescript-standards" are considered equivalents.

#### 4. Suggestions

Actionable recommendations for creating equivalents:

- Cursor rules to add for Claude Code features
- Claude Code skills/commands to add for Cursor features
- CLAUDE.md additions for Cursor "always" rules

## Feature Comparison

### Key Differences

**Claude Code Capabilities**:

- Executable commands with tool permissions
- Sub-agents with separate context windows
- Helper commands (TypeScript utilities)
- Event-driven hooks
- Progressive resource loading

**Cursor Capabilities**:

- Glob pattern matching for auto-attachment
- Four rule attachment modes (Always/Auto/Agent/Manual)
- Manual invocation with @ruleName syntax
- Lightweight single-file rules

**See @references/feature-comparison.md for comprehensive comparison**

## Common Workflows

### Workflow 1: Initial Audit

When setting up a new project or reviewing existing configuration:

1. Run audit: `ai skill config-audit run`
2. Review gap analysis section
3. Prioritize gaps based on project needs
4. Create missing equivalents as needed

### Workflow 2: Migration Planning

When migrating from Cursor to Claude Code (or vice versa):

1. Run audit with JSON output: `ai skill config-audit run --json`
2. Review all resources in source tool
3. Consult feature comparison guide for migration patterns
4. Systematically create equivalents in target tool

### Workflow 3: Feature Parity Maintenance

When adding new AI workflows:

1. Implement in preferred tool (Claude Code or Cursor)
2. Run audit to identify new gap
3. Decide if equivalent is needed in other tool
4. Create equivalent using appropriate patterns

## Creating Equivalents

### Claude Code Command → Cursor Command

Claude Code commands with `allowed-tools` cannot be directly replicated (Cursor commands don't execute code). Consider:

- Create Cursor command as text-only prompt template
- Or skip if code execution is essential

### Claude Code Skill → Cursor Rule

1. Extract core instructions from SKILL.md
2. Create `.cursor/rules/[name].mdc` with appropriate frontmatter:
   - Choose rule type (Always/Auto/Agent/Manual)
   - Add globs if auto-attached
   - Add description for agent-requested
3. Keep under 500 lines (decompose if needed)
4. Remove helper command references (not supported)

### Cursor Always Rule → CLAUDE.md

For critical always-loaded standards:

1. Extract rule content
2. Add to `.claude/CLAUDE.md` in appropriate section
3. Use XML tags for safe merging if needed

### Cursor Auto-Attached Rule → Claude Code Skill

1. Create skill with description mentioning relevant file types
2. Include instructions from rule body
3. Add helper commands if repetitive operations exist

### Cursor Manual Rule → Claude Code Command

1. Create `.claude/commands/[name].md`
2. Add rule content as command body
3. Add `allowed-tools` if code execution needed
4. Invoke with `/command-name` instead of `@ruleName`

## Helper Commands

This skill provides the `ai skill config-audit run` command for deterministic configuration auditing.

**Implementation**: See `typescript/lib/config-audit-operations.ts` for scanning and comparison logic.

**Command options**:

- `-p, --project-root <path>` - Project root directory (default: current directory)
- `--json` - Output as JSON instead of formatted text

## Resources

### references/feature-comparison.md

Comprehensive comparison of Claude Code and Cursor features including:

- Detailed capability breakdowns
- Functional equivalencies table
- Unique features in each tool
- Migration considerations
- When to use each feature type

Reference this when:

- Planning migrations between tools
- Deciding where to implement new workflows
- Understanding fundamental feature differences

## Interpreting Results

### High Gap Count

Many resources without equivalents suggests:

- **Single-tool focus** - Team primarily using one tool
- **Migration in progress** - Transitioning between tools
- **Specialization** - Leveraging unique capabilities

**Actions**:

- Review whether equivalents are needed
- Focus on high-value workflows first
- Consider tool consolidation if maintaining both is burdensome

### Low Gap Count

Few resources without equivalents suggests:

- **Feature parity** - Comprehensive coverage in both tools
- **Parallel development** - Team actively maintains both
- **Shared workflows** - Common processes across tools

**Actions**:

- Maintain parity for new workflows
- Document equivalent resource pairs
- Streamline maintenance procedures

### Type-Specific Gaps

Patterns in gap types reveal opportunities:

- **Many skills without rules** - Rich Claude Code workflows, lighter Cursor setup
- **Many always rules without CLAUDE.md** - Cursor-heavy standards not in Claude Code
- **Commands without equivalents** - Tool-specific automations

**Actions**:

- Evaluate whether gaps represent intentional choices
- Create equivalents for essential workflows
- Accept gaps for tool-specific features

## Best Practices

**Audit Regularly**:

- Run audit after major changes
- Include in code review process
- Track gaps over time

**Prioritize Thoughtfully**:

- Not all resources need equivalents
- Focus on high-impact workflows
- Consider maintenance burden

**Document Decisions**:

- Record why gaps exist
- Note intentional tool-specific features
- Update comparison guide with learnings

**Maintain Feature Comparison**:

- Update references/feature-comparison.md as tools evolve
- Document new feature types
- Share insights with team
