---
name: create-new-rule
description: Create a new agent rule or steering file from chat context. Detects the current IDE (Cursor or Kiro) and creates the file in the correct format and location.
---

# Create a New Rule / Steering File

Create a new agent rule or steering file based on the current chat context. The rule should be clear, concise, and placed in the correct location for the current IDE environment.

## Environment Detection

Before creating any file, detect which IDE you're running in:

- **Kiro workspace** (`.kiro/` directory exists): Create a `.kiro/steering/*.md` file with Kiro frontmatter
- **Cursor workspace** (`.cursor/` directory exists): Create a `.cursor/rules/*.mdc` file with Cursor frontmatter
- **Both exist** (home directory or multi-IDE workspace): Ask the user which IDE to target, or create for both using the sync script (`bash ~/scripts/ide-sync.sh sync --from <source> --to all`)

If running from the home directory (`~`), default to the current IDE's format and offer to sync.

## Process

1. **Analyze the chat context:**
   - Review the conversation to identify the specific preference, pattern, or instruction the user wants to establish
   - Extract the core requirement into a single, clear statement
   - Determine scope: always active, file-type-specific, or manually invoked

2. **Create the file in the correct format for the detected IDE**

3. **Before finalizing, ask the user to review:**
   - Present the proposed rule content (including frontmatter and the rule body)
   - Ask: "Please review and let me know if you'd like any changes before I create the file"
   - Wait for user approval or requested modifications

4. **After user approval:**
   - Create the file in the correct location
   - Offer to sync to other IDEs if the sync script is available

## Format Reference

### Kiro Steering (`.kiro/steering/<name>.md`)

Always-active rule:
```markdown
---
inclusion: auto
---

Your clear, concise rule here.
```

File-match rule:
```markdown
---
inclusion: fileMatch
fileMatchPattern: "*.ts"
---

Your clear, concise rule here.
```

Manually invoked rule:
```markdown
---
inclusion: manual
---

Your clear, concise rule here.
```

### Cursor Rules (`.cursor/rules/<name>.mdc`)

Always-active rule:
```
---
alwaysApply: true
---

Your clear, concise rule here.
```

File-glob rule:
```
---
globs: "*.ts"
alwaysApply: true
---

Your clear, concise rule here.
```

Agent-requested rule:
```
---
description: "Brief description of when to use this rule"
alwaysApply: false
---

Your clear, concise rule here.
```

## Scope Mapping

| Intent | Kiro | Cursor |
|---|---|---|
| Always active | `inclusion: auto` | `alwaysApply: true` |
| File-type specific | `inclusion: fileMatch` + `fileMatchPattern` | `globs` + `alwaysApply: true` |
| On-demand / manual | `inclusion: manual` | `alwaysApply: false` + `description` |

## Guidelines

- Rules should be concise and actionable
- Use kebab-case for filenames based on the rule's purpose
- Default to always-active unless the user specifies otherwise
- For home-directory rules, these apply globally across all projects
- After creating, offer to sync across IDEs if `~/scripts/ide-sync.sh` exists
