---
name: skill-manager
description: This skill should be used when the user asks to "create a skill", "improve a skill", "edit a skill", "list skills", "analyze skill", "skill作成", "スキルを作る", "スキルを改善", "スキル一覧", "スキル編集", or wants to manage Claude Code skills.
argument-hint: "<create|improve|list> [skill-name]"
version: 2.0.0
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

# Skill Manager

Manage Claude Code skills: create, improve, list, and analyze.

## Usage

```text
/skill-manager create [skill-name]   # Create a new skill
/skill-manager improve [skill-name]  # Improve an existing skill
/skill-manager list                  # List all skills
/skill-manager [skill-name]          # Analyze/improve (default)
```

Parse `$ARGUMENTS` to determine the action. If no action is specified, default to `improve` if a skill name is given.

## Current Skills

Global skills:
!`ls -1 ~/.claude/skills/ 2>/dev/null || echo "(none)"`

Project skills:
!`ls -1 .claude/skills/ 2>/dev/null || echo "(none)"`

---

## Action: create

Create a new skill from scratch.

### Workflow

1. **Gather Requirements** via AskUserQuestion:
   - Skill name (kebab-case)
   - Purpose and description
   - Trigger phrases (Japanese + English)
   - Required tools
   - Location: `~/.claude/skills/` (global) or `.claude/skills/` (project)

2. **Create Directory Structure**:

   ```text
   skill-name/
   ├── SKILL.md              # Required
   ├── references/           # Optional: Reference docs
   ├── templates/            # Optional: Template files
   └── examples/             # Optional: Usage examples
   ```

3. **Write SKILL.md** using the template in `templates/basic.md` or `templates/advanced.md`

4. **Report Success**: Show the invoke command `/skill-name`

---

## Action: improve

Improve an existing skill.

### Workflow

1. **Locate the Skill**:
   - Check `~/.claude/skills/{skill-name}/SKILL.md`
   - Check `.claude/skills/{skill-name}/SKILL.md`
   - If not found, list available skills and ask user

2. **Analyze Current State**:
   - Read SKILL.md and all related files
   - Identify issues:
     - Vague or missing trigger phrases
     - Overly broad or missing tool restrictions
     - Unclear instructions
     - Missing error handling
     - No examples

3. **Suggest Improvements** via AskUserQuestion:
   - Present findings and recommendations
   - Ask what the user wants to improve:
     - Trigger phrases (more specific, bilingual)
     - Instructions (clearer workflow)
     - Tool restrictions (security)
     - Examples (usage clarity)
     - Structure (add references/, templates/)

4. **Apply Changes**: Edit SKILL.md and related files

5. **Report Changes**: Summarize what was improved

---

## Action: list

List all available skills with their status.

### Workflow

1. **Scan Directories**:
   - `~/.claude/skills/` (global)
   - `.claude/skills/` (project-local)

2. **For Each Skill, Report**:
   - Name and version
   - Description (first line)
   - Trigger phrases count
   - Allowed tools
   - Location (global/project)

3. **Output Format**:

   ```text
   📁 Global Skills (~/.claude/skills/)
   ├── skill-name (v1.0.0) - Description
   │   └── Triggers: "phrase1", "phrase2"

   📁 Project Skills (.claude/skills/)
   └── (none)
   ```

---

## Frontmatter Reference

```yaml
---
name: Skill Name                    # Required: Display name
description: This skill should be used when the user asks to "trigger1", "trigger2".  # Required
argument-hint: "[args]"             # Optional: Shows in /command help
version: 1.0.0                      # Optional: Semantic version
allowed-tools:                      # Optional: Restrict tools
  - Read
  - Write
disable-model-invocation: false     # Optional: true = manual only
---
```

### Writing Good Trigger Phrases

**Good:**

```yaml
description: This skill should be used when the user asks to "create a migration", "generate migration", "マイグレーション作成".
```

**Bad:**

```yaml
description: Helps with database tasks.  # Too vague, won't trigger
```

---

## Dynamic Content

Embed shell output in SKILL.md:

```markdown
!`git status --short`
```

## Variables

| Variable | Description |
|----------|-------------|
| `$ARGUMENTS` | Args from `/skill-name arg1 arg2` |
| `${CLAUDE_SESSION_ID}` | Current session ID |

---

## Best Practices

1. **Specific triggers**: Include exact phrases users will say (quoted)
2. **Bilingual**: Japanese + English trigger phrases
3. **Restrict tools**: Use `allowed-tools` for security
4. **Keep SKILL.md concise**: Put reference docs in `references/`
5. **Version your skills**: Update version on changes
6. **Test triggers**: Verify the skill activates on expected phrases
