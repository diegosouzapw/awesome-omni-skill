---
name: ai-tool-converter
description: Convert AI coding assistant configuration files (agents, skills, commands, rules) between formats (Claude Code, Cursor, and future tools). Handles frontmatter mapping, directory structure, and format-specific fields.
---

# AI Tool Converter

Convert agents, skills, commands, and rules between different AI coding assistant formats while preserving functionality and adapting format-specific features.

## Supported File Types

**Both Claude Code and Cursor support these concepts:**

| File Type | Purpose | Claude Code | Cursor |
|-----------|---------|-------------|--------|
| **Agents** | Specialized subagents for delegation | Yes | Yes |
| **Skills** | Reusable capabilities with references | Yes | Yes |
| **Commands** | Custom slash commands | Yes | Yes |
| **Rules** | Project-wide instructions | Yes | Yes |

## File Locations

| File Type | Claude Code | Cursor |
|-----------|-------------|--------|
| **Agents** | `.claude/agents/**/*.md` | `.cursor/agents/**/*.md` |
| **Skills** | `.claude/skills/<name>/SKILL.md` | `.cursor/skills/<name>/SKILL.md` |
| **Commands** | `.claude/commands/*.md` | `.cursor/commands/*.md` |
| **Rules** | `.claude/rules/*.md`, `CLAUDE.md` | `.cursor/rules/*.mdc`, `AGENTS.md` |

**Compatibility**: Cursor reads `.claude/` directories as fallback for agents and skills.

## Parameters

- **`source-format`** - Source format (`claude-code`, `cursor`)
- **`target-format`** - Target format (`claude-code`, `cursor`)
- **`file-path`** - Path to file or directory to convert

## Steps

### Step 1: Identify File Type

Determine the file type from its path:

```
.claude/agents/foo.md      -> Agent
.claude/skills/foo/SKILL.md -> Skill
.claude/commands/foo.md    -> Command
.claude/rules/foo.md       -> Rule
CLAUDE.md                  -> Rule
```

If unclear, ask: "What type of file is this? (agent, skill, command, or rule)"

### Step 2: Validate Input

1. Parse arguments to extract source format, target format, and file path
2. If parameters are unclear, ask user to clarify
3. Validate source and target formats are supported and different
4. Verify file/directory exists

### Step 3: Read and Parse Source

1. Read the source file(s)
2. Parse YAML frontmatter (if present)
3. Extract Markdown body/prompt content
4. Identify format-specific fields

### Step 4: Apply Conversion Rules

#### Agents

**Claude Code -> Cursor:**
- Preserve: `name`, `description`
- Convert model: `haiku`->`fast`, `sonnet`/`opus`->`inherit`
- Remove: `tools`, `disallowedTools`, `permissionMode`, `color`, `skills`, `hooks`, `disabled`
- Add if applicable: `readonly: true`, `is_background: true`

**Cursor -> Claude Code:**
- Preserve: `name`, `description`
- Convert model: `fast`->`haiku`
- Remove: `readonly`, `is_background`
- Add if applicable: `tools`, `permissionMode`

#### Skills

**Claude Code -> Cursor:**
- Keep: `name`, `description`, `disable-model-invocation`
- Remove: `argument-hint`, `user-invocable`, `allowed-tools`, `model`, `context`, `agent`, `hooks`
- Add if needed: `license`, `compatibility`, `metadata`
- **Warning**: `$ARGUMENTS` and `` !`cmd` `` syntax won't work in Cursor

**Cursor -> Claude Code:**
- Keep: `name`, `description`, `disable-model-invocation`
- Remove: `license`, `compatibility`, `metadata`
- Add if needed: `argument-hint`, `allowed-tools`, `model`

**Important**: Copy entire skill directory (SKILL.md + references/ + scripts/), not just SKILL.md

#### Commands

**CRITICAL DIFFERENCE**: Cursor commands have **NO YAML frontmatter** - they are plain Markdown only!

**Claude Code -> Cursor:**
- **Remove ALL frontmatter** - Cursor doesn't support it
- **Remove** `$ARGUMENTS`, `$0` substitutions - won't work
- **Remove** `` !`cmd` `` dynamic context - won't work
- Keep body as plain Markdown only

**Cursor -> Claude Code:**
- Add frontmatter (optional) to enhance functionality
- Consider adding: `description`, `argument-hint`, `disable-model-invocation`
- Keep body unchanged

#### Rules

**Claude Code -> Cursor:**
- File: `CLAUDE.md` -> `.cursor/rules/main.mdc` or `AGENTS.md`
- File: `.claude/rules/*.md` -> `.cursor/rules/*.mdc`
- Convert: `paths` -> `globs`
- Remove: `@import` syntax (not supported)
- Add: `description`, `alwaysApply` if needed

**Cursor -> Claude Code:**
- File: `.cursor/rules/*.mdc` -> `.claude/rules/*.md`
- File: `AGENTS.md` -> `CLAUDE.md`
- Convert: `globs` -> `paths`
- Remove: `description`, `alwaysApply`

### Step 5: Update Internal References

**CRITICAL**: Scan file contents for references to agents, commands, and skills, then update them for the target format.

**Common patterns to find:**

| Pattern Type | Claude Code | Cursor |
|--------------|-------------|--------|
| Agent paths | `.claude/agents/foo` | `.cursor/agents/foo` |
| Skill paths | `.claude/skills/bar` | `.cursor/skills/bar` |
| Command paths | `.claude/commands/baz` | `.cursor/commands/baz` |
| Rule paths | `.claude/rules/qux.md` | `.cursor/rules/qux.mdc` |
| Slash commands | `/foo` | `/foo` (no change) |
| Global rules | `CLAUDE.md` | `AGENTS.md` |

**Update strategy:**
1. Replace directory prefix: `.claude/` <-> `.cursor/`
2. Replace global rule file: `CLAUDE.md` <-> `AGENTS.md`
3. Update rule extensions: `.md` <-> `.mdc` (for rules only)
4. Preserve slash command names (they don't change)
5. Convert agent invocation syntax (see below)

#### Agent Invocation Syntax

**Claude Code -> Cursor conversion:**

Before:
```markdown
Use the Task tool to invoke the reviewer:
- Set `subagent_type: code-reviewer`
- Pass the file path in the prompt
```

After:
```markdown
Invoke the reviewer agent using:
- Explicit: /code-reviewer review this code
- Natural: Use the code-reviewer subagent to review this code
```

**Cursor -> Claude Code conversion:**

Before:
```markdown
Invoke the code reviewer:
- Use /code-reviewer followed by your request
- Or: Have the code-reviewer subagent check for security issues
```

After:
```markdown
Use the Task tool to invoke the code reviewer:
- Set `subagent_type: code-reviewer`
- Include your request in the prompt parameter
- Example: Task tool with subagent_type: code-reviewer, prompt: "Check for security issues"
```

### Step 6: Determine Output Path

Mirror the source structure in the target format:

| Source | Target |
|--------|--------|
| `.claude/agents/foo.md` | `.cursor/agents/foo.md` |
| `.claude/skills/foo/SKILL.md` | `.cursor/skills/foo/SKILL.md` |
| `.claude/commands/foo.md` | `.cursor/commands/foo.md` |
| `.claude/rules/foo.md` | `.cursor/rules/foo.mdc` |

Ask user if they want a different output location.

### Step 7: Write Converted File

1. Create target directory with `mkdir -p` if needed
2. For skills: copy entire directory structure
3. Write converted file with appropriate frontmatter (or none for Cursor commands)
4. Verify file was created successfully

### Step 8: Report Results

```
Converted: [source-file]
Type: [agent|skill|command|rule]
From: [source-format]
To: [target-format]
Output: [output-file]

Changes made:
- [Field changes]
- Removed: [list]
- Added: [list]

Limitations:
- [Features that won't work in target format]

Test the converted file:
[Instructions for target tool]
```

## Examples

### Example 1: Convert Agent

```
claude-code cursor .claude/agents/reviewer.md
```

**Result:**
- Input: `.claude/agents/reviewer.md` (with `model: sonnet`, `tools: [Read, Grep]`)
- Output: `.cursor/agents/reviewer.md` (with `model: inherit`, `readonly: true`)
- Removed: `tools` field
- Added: `readonly: true` (since original had restricted tools)

### Example 2: Convert Skill

```
claude-code cursor .claude/skills/code-review/
```

**Result:**
- Copies entire skill directory to `.cursor/skills/code-review/`
- Converts SKILL.md frontmatter
- Preserves references/ and scripts/ subdirectories
- Warning: `$ARGUMENTS` substitutions won't work

### Example 3: Convert Command

```
claude-code cursor .claude/commands/deploy.md
```

**Result:**
- Input: `.claude/commands/deploy.md` (with frontmatter and `$ARGUMENTS`)
- Output: `.cursor/commands/deploy.md` (plain Markdown, no frontmatter)
- **Removed**: All frontmatter, `$ARGUMENTS` references
- **Warning**: Lost functionality - user must manually pass arguments

## Key Differences Summary

| Feature | Claude Code | Cursor |
|---------|-------------|--------|
| **Command frontmatter** | Full YAML supported | **None** (plain Markdown) |
| **String substitutions** | `$ARGUMENTS`, `$0` | Not supported |
| **Dynamic context** | `` !`cmd` `` | Not supported |
| **Model values** | `haiku`, `sonnet`, `opus` | `fast`, `inherit` |
| **Tool restrictions** | `tools`, `disallowedTools` | `readonly` flag only |
| **Skill arguments** | `argument-hint` | Not supported |
| **Rule scoping** | `paths` array | `globs` field |

## Validation Checklist

After conversion:
- [ ] Correct file type identified
- [ ] YAML frontmatter valid (or correctly removed for Cursor commands)
- [ ] Required fields present for target format
- [ ] No source-format-specific fields remain
- [ ] Internal references updated (`.claude/` <-> `.cursor/`, `CLAUDE.md` <-> `AGENTS.md`)
- [ ] Agent invocation syntax converted
- [ ] Markdown links point to correct target paths
- [ ] Markdown body preserved (except for reference updates)
- [ ] File in correct target directory
- [ ] User warned about lost functionality

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| Unknown file type | Can't determine from path | Ask user to specify |
| Invalid YAML | Malformed frontmatter | Fix source file first |
| Missing required fields | Source incomplete | Add fields before converting |
| Feature not supported | Target lacks capability | Warn user, remove feature |
