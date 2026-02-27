---
name: command-to-subagent
description: Use when converting a Ruly command to use subagent execution, when a command has heavy MCP/API logic that should be isolated, or when asked to "make this a subagent"
---

# Converting Commands to Subagents

## Overview

Extract execution logic from a command into a dedicated subagent, keeping only the user-facing workflow (preview, confirmation) in the command. This reduces recipe context by isolating API/MCP complexity.

## When to Use

- Command has heavy MCP or API call logic
- Want to reduce token usage in a recipe
- Need to isolate execution complexity from user workflow
- Command follows preview → confirm → execute pattern

## Quick Reference

| Component | Location | Contains |
|-----------|----------|----------|
| Command | `commands/{name}.md` | User workflow (preview, confirm, dispatch) |
| Subagent | `agents/{name}.md` | Execution logic (MCP calls, API operations) |
| Recipe | `recipes.yml` | Subagent registration |

## Conversion Steps

### Step 1: Identify the Split Point

Find where user interaction ends and execution begins:

```
User workflow (COMMAND):     Execution (SUBAGENT):
├─ Parse input               ├─ API/MCP calls
├─ Validate/lookup           ├─ Error handling
├─ Show preview              ├─ Response formatting
├─ Handle revisions          └─ Return structured result
└─ Dispatch on confirm
```

**Split at confirmation** - everything after "send it" / "post it" goes to subagent.

### Step 2: Create the Subagent File

Location: `rules/{domain}/agents/{name}.md`

**Structure:**
```markdown
---
description: Instructions for the {name} subagent when dispatched
alwaysApply: true
requires:
  - ../common.md        # Shared patterns
  - ../../accounts.md   # If needs user lookups
---
# {Name} Subagent - EXECUTION REQUIRED

## ⚠️ CRITICAL: You MUST Execute

**Your ONLY job is to {action}.** If you complete with "0 tool uses", you have FAILED.

## Input Format

You will receive a prompt like:
{show expected input structure}

## Execution Steps

### Step 1: {First action}
{MCP/API call with example}

### Step 2: {Next action}
...

### Step N: Return Result

**On success:**
```json
{"status": "success", "field": "value"}
```

**On error:**
```json
{"status": "error", "error": "message", "step": "which failed"}
```

## Failure Conditions

❌ **FAILED** if you:
- Complete with "0 tool uses"
- Return without executing
- Ask clarifying questions

✅ **SUCCESS** if you:
- Execute all required steps
- Return structured result
```

### Step 3: Update the Command

Remove execution logic, add subagent dispatch:

**Before (inline execution):**
```markdown
### Step 4: Send Message
mcp__teams__send_chat_message with:
- chatId: ...
```

**After (dispatch to subagent):**
```markdown
### Step 4: Dispatch Subagent

When user confirms, dispatch the `{subagent_name}` subagent:

Task tool:
  subagent_type: "{subagent_name}"
  prompt: |
    {Action}:
    - Field1: {value1}
    - Field2: {value2}

### Step 5: Show Result

Display the subagent's response:
- On success: ✅ {success message}
- On error: ❌ {error message}
```

### Step 4: Register in Recipe

Add to `recipes.yml`:

```yaml
# Subagent recipe (what the subagent gets loaded with)
{subagent-name}:
  description: "{Action} (subagent execution)"
  files:
    - /path/to/agents/{name}.md
    - /path/to/common.md
    - /path/to/accounts.md  # if needed
  mcp_servers:
    - {required-mcp}

# Parent recipes that use the command
{parent-recipe}:
  files:
    - /path/to/commands/{name}.md
  subagents:
    - name: {subagent_name}  # underscores for Task tool
      recipe: {subagent-name}  # hyphens for recipe name
```

### Step 5: Sync Config Files

```bash
cp recipes.yml ~/.config/ruly/recipes.yml
cp recipes.yml ~/Projects/chezmoi/config/ruly/recipes.yml
```

## Example: MS Teams DM

**Before:** `dm.md` had all Teams MCP logic inline

**After:**
- `commands/dm.md` - Preview workflow, dispatches on "send it"
- `agents/ms-teams-dm.md` - Teams MCP execution

**Recipe registration:**
```yaml
ms-teams-dm:
  files:
    - rules/comms/ms-teams/agents/ms-teams-dm.md
    - rules/comms/ms-teams/common.md
  mcp_servers:
    - teams

comms:
  files:
    - rules/comms/ms-teams/commands/dm.md
  subagents:
    - name: ms_teams_dm
      recipe: ms-teams-dm
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Subagent asks questions | Add "EXECUTION REQUIRED" header, forbid questions |
| Subagent returns without acting | Add "0 tool uses = FAILED" warning |
| Missing MCP server in recipe | Subagent can't call APIs without `mcp_servers` |
| Forgot to sync config files | Always sync to `~/.config/ruly/` and chezmoi |
| Using wrong name format | Recipe names: hyphens. Task tool names: underscores |

## Directory Convention

```
rules/{domain}/
├── common.md           # Shared patterns
├── commands/
│   └── {name}.md       # User-facing command
└── agents/
    └── {name}.md       # Subagent execution
```

- `commands/` = User-invocable via `/command-name`
- `agents/` = Dispatched programmatically (not user-invocable)
