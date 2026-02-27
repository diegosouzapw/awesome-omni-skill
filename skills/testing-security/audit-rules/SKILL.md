---
name: audit-rules
description: Audit Claude Code rule files for quality and compliance. Use when creating or validating .claude/rules/*.md files, or troubleshooting rule loading issues.
argument-hint: [--force | --skip-validation]
allowed-tools: Read, Bash, Glob, Grep, Task
---

# Audit Rules Command

Audit Claude Code rule files (`.claude/rules/*.md`) for quality and compliance.

## Initialization

Before auditing, initialize the environment:

1. Get the current UTC date for audit timestamps.
2. Capture the project root path for subagent communication.
3. Ensure the temp directory (`.claude/temp/`) exists.
4. Clean up any stale audit files if the user confirms.

The `memory-management` skill provides authoritative validation guidance for rules (auto-loaded when this command runs).

## What Gets Audited

- YAML frontmatter structure (description, globs)
- Glob pattern validity and syntax
- Rule file naming conventions
- Content structure and clarity
- Path-specific rule applicability

## Command Arguments

| Argument | Description |
| --- | --- |
| *(none)* | Audit all discoverable rule files |
| `--force` | Audit regardless of modification status |
| `--skip-validation` | Skip finding validation (faster, but may include false positives) |

## Step 1: Discover Rule Files

Search for rule files in:

- Project rules (`.claude/rules/*.md`)
- User rules (`~/.claude/rules/*.md`)

Build a list of discovered rule files with their scope (project or user) and full path.

If no rule files are found, report this and provide guidance on how to create one.

## Step 2: Parse Arguments

Check if the `--force` flag is present in the command arguments. Build the audit queue based on discovered files and the force flag.

## Step 3: Present Audit Plan

Display audit mode (SMART or FORCE), rule files discovered, and list each with scope and last modified date.

## Step 4: Execute Audits

For each rule file, spawn the `memory-component-auditor` subagent with the following context:

- Scope (project or user)
- Full path to the rule file
- Last audit date or "Never audited"
- Current audit date
- Project root path

Run subagents in parallel when multiple rule files exist.

Subagents write findings to `.claude/temp/` as both JSON (for recovery/aggregation) and markdown (for human review). The main conversation thread collects results and updates audit logs using its Write/Edit tools.

## Step 4.5: Validate Findings

**Unless `--skip-validation` flag is present:**

1. Spawn the `audit-finding-validator` agent with:
   - `project_root`: The captured project root path
   - `audit_type`: "rule"
   - `audit_files`: List of `.claude/temp/audit-*-rule-*.json` file paths
2. Wait for validation to complete
3. Read updated JSON files with validation results
4. Filter out FALSE_POSITIVE findings completely before aggregation
5. Note: Filtered findings are logged to `.claude/temp/audit-filtered-findings.json`

**If `--skip-validation` flag is present:**

- Skip validation phase entirely (current speed preserved)
- Present all findings without filtering
- Note in summary: "Validation: Skipped"

## Step 5: Final Summary

Report total rule files audited, results by scope, and details table. List frontmatter or glob pattern issues with remediation steps.

**Include validation statistics (if validation was performed):**

- Validation performed: Yes/No
- Findings validated: X
- False positives filtered: Y
- Verified findings: Z
- Unverified findings: W

## Important Notes

### Rule File Requirements

Rule files must have valid YAML frontmatter with `description` and optionally `globs` fields. The `globs` field controls which files the rule applies to.

### Rule File Locations

| Location | Purpose |
| --- | --- |
| `.claude/rules/*.md` | Project-specific rules |
| `~/.claude/rules/*.md` | User-wide rules |

### Glob Pattern Syntax

Rules can use glob patterns to apply only to specific files:

```yaml
---
description: TypeScript coding standards
globs: ["**/*.ts", "**/*.tsx"]
---
```

## Audit Log Location

All audit results are written to `.claude/audit/rules.md`.

Use `/audit-log rules` to view current audit status.

## Example Usage

### Example 1: Audit All Rule Files

```text
User: /audit-rules

Claude: Discovering rule files...

## Audit Plan
**Mode**: SMART
**Rule files discovered**: 3

1. [project] .claude/rules/typescript.md
2. [project] .claude/rules/security.md
3. [user] ~/.claude/rules/personal-style.md

[Spawns memory-component-auditor subagents]

## Audit Complete
| Scope | Rule File | Result | Score |
| --- | --- | --- | --- |
| project | typescript.md | PASS | 100/100 |
| project | security.md | PASS | 95/100 |
| user | personal-style.md | PASS WITH WARNINGS | 82/100 |
```

### Example 2: Force Audit

```text
User: /audit-rules --force

Claude: Auditing all rule files (force mode)...
```
