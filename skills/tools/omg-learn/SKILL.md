---
name: omg-learn
version: 2.0.0
description: Learning from user corrections by creating skills and patterns. Patterns can prevent mistakes (block/warn/ask) or inject helpful context into prompts. Use when user says "omg!" during corrections, when user provides feedback to create persistent knowledge, or when user wants to ensure AI never makes the same mistake again.
dependencies:
  - skill-creator
---

# OMG! Learning from Corrections

When the user says "omg!" during a correction, create or update a skill AND optionally a preventive pattern.

**ALWAYS load the skill-creator skill first.**

## Quick Reference

- **CLI tool:** `omg-learn list|show|enable|disable|test|simulate|sync|export|import`
- **Pattern files:** 
  - Claude Code: `~/.claude/omg-learn-patterns.json` (global), `.claude/omg-learn-patterns.json` (local)
  - Cursor: `~/.cursor/omg-learn-patterns.json` (global), `.cursor/omg-learn-patterns.json` (local)
- **Detailed guides:** See `references/` directory

## Workflow

When user says "omg!" during a correction:

### 0. Check Platform Hooks (First Time Only)

Check if hooks are installed:
- Claude Code: Check for `~/.claude/hooks/pretool-checker.sh` or `.claude/hooks/pretool-checker.sh`
- Cursor: Check for `~/.cursor/hooks.json` or `.cursor/hooks.json`

**If NOT installed:**
- Explain platform hooks catch mistakes BEFORE they happen
- Ask: "Would you like to install hooks now?"
- If yes: Run `./scripts/install-hooks.sh`
- Platform hooks are OPTIONAL but recommended

**Platform Hooks vs Skill Hooks:**
- **Platform hooks:** Work on both Claude Code and Cursor, always active, prevent first mistake
- **Skill hooks:** Claude Code ONLY feature - hooks defined within skills themselves. Cursor does not support skill hooks.
- This skill primarily uses platform hooks for maximum compatibility

### 1. Analyze the Correction

Ask:
- What did you do wrong?
- What is the correct behavior?
- What knowledge should be preserved?
- Could this be prevented with a pattern?

**Skill creation tip:** Include trigger contexts in description.
Example: "Use when analyzing database code" or "When deploying to production"

### 2. Ask About Scope (ALWAYS)

**ALWAYS explicitly ask the user about scope - never assume!**

Ask: "Should this skill be global (all projects) or local (this project only)?"

**Options:**
- **Global** → `~/.claude/skills/` (Claude Code) or `~/.cursor/skills/` (Cursor)
  - Use for: General programming knowledge, universal best practices, cross-project patterns
  - Example: "git best practices", "database migration patterns", "security guidelines"

- **Local** → `.claude/skills/` (Claude Code) or `.cursor/skills/` (Cursor)
  - Use for: Project-specific conventions, architecture decisions, team guidelines
  - Example: "this project's auth flow", "our deployment checklist", "team code style"

**Note:** Also supports `.agents/skills/` or `.agent/skills/` directories if present.

**Default suggestion:** If unsure, suggest local first (less intrusive, easier to test).

### 3. Search for Existing Skills

Search appropriate location for related skills.

If found → Propose update
If not found → Propose new skill

### 4. Propose Changes

- For updates: Show before/after diff
- For new skills: Show full YAML with frontmatter
- Indicate storage location (global vs project-local)

### 5. Create or Update Skill

After user approval, create/update the skill file.

### 6. Generate Companion Context Injection Pattern (DEFAULT)

**Automatically propose a context injection pattern to remind Claude when to use this skill:**

**Step 6.1: Extract trigger keywords from skill description**

Look for:
- "Use when...", "When..." clauses in the skill description
- Technical terms and domain vocabulary (database, git, production, migration, etc.)
- User-provided trigger contexts

**Example:**
- Skill description: "Use when working with database schema changes, migrations, alters"
- Extracted keywords: database, migration, schema, alter

**Step 6.2: Show proposed pattern**

Present the pattern structure:
```json
{
  "id": "remind-[skill-name]-skill",
  "description": "Reminds Claude to use [skill-name] skill",
  "hook": "UserPromptSubmit",
  "pattern": "(keyword1|keyword2|keyword3)",
  "action": "warn",
  "message": "💡 Consider using the [skill-name] skill. It covers: [summary]",
  "skill_reference": "[skill-name]",
  "enabled": true,
  "note": "Companion pattern for skill"
}
```

**Step 6.3: Ask for confirmation**

Show: "Extracted keywords: database, migration, schema"

Ask: "Create pattern with these keywords? (or customize/skip)"

Options:
- **Yes** → Continue to Step 6.4
- **Customize** → Let user edit keywords list, then continue to Step 6.4
- **Skip** → Don't create pattern (user can add manually later)

**Step 6.4: Ask about pattern scope (ALWAYS)**

**ALWAYS explicitly ask where to save the pattern - never assume!**

Ask: "Should this pattern be global (all projects) or local (this project only)?"

**Options:**
- **Global** → `~/.claude/omg-learn-patterns.json` (Claude Code) or `~/.cursor/omg-learn-patterns.json` (Cursor)
  - Use for: Universal patterns that apply everywhere
  - Example: "no piping to head", "git safety checks"

- **Local** → `.claude/omg-learn-patterns.json` (Claude Code) or `.cursor/omg-learn-patterns.json` (Cursor)
  - Use for: Project-specific patterns
  - Example: "this project's conventions", "team-specific reminders"

**Default suggestion:** Match the skill scope (if skill is global, suggest global pattern; if skill is local, suggest local pattern)

**Step 6.5: Create and add pattern**

Generate the pattern JSON using the template from `references/skill-pattern-template.md`

Add to appropriate patterns file (global or local) based on user's choice

Link back to skill with `skill_reference` field

**Why this is the magic sauce:**
- **Proactive:** Claude is reminded BEFORE attempting the task
- **Non-disruptive:** Just adds helpful context to Claude's prompt
- **Discoverable:** Users learn about skills as they work
- **Educational:** Claude sees guidance and learns when to apply skills

**Skip pattern creation only if:**
- No hooks installed (warn user about missing capability)
- Skill has no clear trigger context (pure reference material)
- User explicitly says "no pattern needed"

**Pattern types (for reference - prefer top to bottom):**
1. **Context injection patterns (🌟 MAGIC SAUCE):** Add guidance to Claude's prompt based on keywords - proactive, non-disruptive (Claude Code: true context, Cursor: warning)
2. **PostToolUse patterns (AUTOMATION):** Run commands after tool execution - auto-formatting, linting, testing
3. **Preventive patterns:** Block or warn before dangerous operations - reactive, more disruptive
4. **Ask patterns (LAST RESORT):** Confirmation dialogs - most disruptive, use sparingly

### 6a. PostToolUse Patterns for Automation (Optional)

In addition to skill-linked context injection patterns, you can create PostToolUse patterns for automation:

**Use PostToolUse for:**
- Auto-formatting files (ruff, prettier, black, etc.)
- Running linters after save (ruff check, eslint, etc.)
- Compiling code automatically
- Running tests on file changes
- Any automation that should happen AFTER tool execution

**PostToolUse pattern structure:**
```json
{
  "id": "auto-format-python",
  "description": "Auto-format Python files with ruff",
  "hook": "PostToolUse",
  "matcher": "Write|Edit",
  "file_pattern": "\\.py$",
  "action": "run",
  "command": "ruff format {file_path}",
  "command_on_success": true,
  "timeout": 10,
  "show_output": false,
  "enabled": true
}
```

**New fields for PostToolUse:**
- `file_pattern`: Regex to match file extensions (e.g., `\\.py$`)
- `command`: Shell command with template variables: `{file_path}`, `{file_name}`, `{file_dir}`, `{file_ext}`
- `command_on_success`: Only run if tool succeeded (default: false)
- `timeout`: Command timeout in seconds (default: 30)
- `show_output`: Show command output to user (default: false)

**See:** `examples/basic-patterns.md` for complete PostToolUse examples

### 7. Generate and Test Pattern

See: `references/pattern-generation-guide.md` for full details.

**Step 7.1: Analyze the Mistake**

Extract key information:
- What tool was used? (Bash, Write, Edit, or prompt?)
- What was the problematic input?
- Why was it wrong?
- How can we detect it? (Regex or check script?)

**Step 7.2: Generate Pattern Components**

**Hook Type:**
- `PreToolUse` for tool interception (Bash/Write/Edit)
- `UserPromptSubmit` for prompt analysis

**Matcher:** (PreToolUse only)
- `Bash` for shell commands
- `Write` for file writes
- `Edit` for file edits
- `*` for all tools

**Pattern Regex:**
Create regex that matches the error pattern.

Examples:
- `\|.*\bhead\b` → Matches pipe to head
- `git\s+commit` → Matches git commit
- `git\s+push.*(-f|--force)` → Matches force push

**Exclude Pattern:** (optional)
Prevent false positives.

Example: `(cat |<|\bhead\s+[^|])` excludes `head package.json`

**Check Script:** (optional)
For complex logic that can't be regex.

Example: Branch checking, environment validation

**Action (hierarchy - best to worst):**

1. **Context injection (UserPromptSubmit with warn/ask)** - 🌟 **MAGIC SAUCE - USE THIS FIRST!**
   - Proactive: Guides Claude's decisions BEFORE they happen
   - Non-disruptive: No permission dialogs
   - Claude Code: True context injection into Claude's prompt
   - Cursor: Warning message to user (not AI context, but still useful)
   - **When to use:** Detect situations from user's prompt keywords
   - **Best for:** Guidance, education, reminders, conventions
   - **Not for:** Dangerous operations that need hard stops (use block instead)

2. **block (PreToolUse)** - For dangerous operations that need hard stops
   - Reactive: Stops action after Claude decided to do it
   - Use when action will definitely cause damage (commit secrets, delete files, drop database)
   - Recommended for prevention that can't be caught via context injection
   - **Critical:** Some things MUST be blocked (secrets, destructive operations)

3. **warn (PreToolUse)** - For educational messages about operations
   - Allow action but show warning
   - Less disruptive than ask

4. **ask (PreToolUse)** - ⚠️ **LAST RESORT - MOST DISRUPTIVE**
   - Cursor: VERY disruptive - AVOID
   - Claude Code: Less disruptive but still prefer context injection
   - Only use when block is too strict AND context injection can't detect it

**Key insight:** Context injection is proactive (guides before decision), ask is reactive (blocks after bad decision). Proactive wins!

**Message:**
Clear explanation with suggested fix.

Format:
```
ERROR: [What's wrong]
[Why it matters]

[Suggested alternative]
```

**Step 7.3: Show Generated Pattern**

Present the complete pattern:

```json
{
  "id": "pattern-id",
  "description": "What this prevents",
  "hook": "PreToolUse",
  "matcher": "Bash",
  "pattern": "regex here",
  "exclude_pattern": "optional exclusion",
  "action": "block",
  "message": "Clear error message",
  "enabled": true
}
```

Include explanation:
- What it matches
- What it allows (excludes)
- Test cases showing behavior

Example:
```
This pattern will:
✓ Block: npm test | head -20
✓ Block: git log | head -5
✗ Allow: head package.json
✗ Allow: cat file | grep foo
```

**Step 7.4: Test Pattern**

```bash
omg-learn test <pattern-id> "test input"
```

Verify matches/excludes work correctly.

**Step 7.5: Enable Pattern**

Ask scope and enable:

```bash
omg-learn enable <pattern-id> --global
# or
omg-learn enable <pattern-id> --local
```

### 8. Register the Skill

**Claude Code:** Skills are auto-discovered from `~/.claude/skills/` or `.claude/skills/`

**Cursor:** Skills are loaded via rules in `~/.cursor/rules/`. Use `generate-cursor-rule` script to create a rule that points to the SKILL.md.

**Done!** The skill and pattern (if created) are now active.

## Advanced Features

**Pattern Management:**
```bash
omg-learn list              # List all patterns
omg-learn show <id>         # Show pattern details
omg-learn enable <id>       # Enable a pattern
omg-learn disable <id>      # Disable a pattern
omg-learn test <id> "input" # Test pattern
omg-learn simulate "cmd"    # Simulate hook execution
```

**Cross-Project:**
```bash
omg-learn sync              # Show sync status
omg-learn export <ids> -o file.zip  # Export patterns
omg-learn import file.zip   # Import patterns
```

See `references/cli-reference.md` for full documentation.

## Platform Support

| Feature | Claude Code | Cursor | Notes |
|---------|-------------|--------|-------|
| Platform hooks | ✅ | ✅ | Different event names |
| Skill hooks | ✅ | ❌ | Claude Code only - not supported in Cursor |
| Skill discovery | ✅ Auto | ✅ Via rules | Claude auto-discovers, Cursor uses .mdc rules |
| CLI tool | ✅ | ✅ | Both platforms |
| Pattern regex | ✅ | ✅ | Same syntax |
| Check scripts | ✅ | ✅ | Bash scripts |

**Event Names:**
- Claude Code: `PreToolUse`, `UserPromptSubmit`
- Cursor: `beforeShellExecution`, `beforeSubmitPrompt`

**Skill Hooks vs Platform Hooks:**
- **Skill hooks** are defined within a skill's YAML frontmatter (Claude Code only)
- **Platform hooks** are defined in `.claude/hooks/` or `.cursor/hooks.json` (both platforms)
- This skill uses **platform hooks** for cross-platform compatibility

## Installation

**Claude Code:**
```bash
cd ~/.claude/skills/omg-learn
./scripts/install-hooks.sh
```

**Cursor:**
```bash
# Install skill
cd ~/.cursor/skills/omg-learn
./scripts/install-hooks.sh

# Generate and install Cursor rule
./scripts/generate-cursor-rule SKILL.md --install
```

**Supported directories:** `.claude/`, `.cursor/`, `.agents/`, `.agent/`

See README.md for full installation guide.

## Common Pattern Examples

See `examples/` directory for detailed examples.

**Preventive patterns:**
- Block commits to main
- Warn on force push
- Prevent piping to head
- Protect important files
- Detect secrets before commit

**Context injection patterns:**
- Remind Claude about project conventions
- Add instructions based on keywords
- Fun surprises (pizza party example!)
- Educational hints for specific tools

**Advanced patterns:**
- Environment-specific checks
- Multi-condition validation
- Custom check scripts
- Team patterns

## References

- `references/pattern-generation-guide.md` - Complete pattern generation guide
- `references/cli-reference.md` - Full CLI documentation
- `references/pattern-structure.md` - JSON schema reference
- `references/hook-system.md` - Technical hook details
- `examples/basic-patterns.md` - Simple pattern examples
- `examples/advanced-patterns.md` - Complex patterns
- `examples/workflows.md` - Common workflows

## Troubleshooting

**Hooks not working?**

Claude Code:
```bash
# Check installation
ls ~/.claude/hooks/pretool-checker.sh
cat ~/.claude/settings.json | grep hooks

# Test manually
echo '{"tool_name":"Bash","tool_input":{"command":"git commit"}}' | ~/.claude/hooks/pretool-checker.sh
```

Cursor:
```bash
# Check installation
ls ~/.cursor/hooks/before-shell.sh
cat ~/.cursor/hooks.json

# Test manually
echo '{"command":"git commit"}' | ~/.cursor/hooks/before-shell.sh
```

**Pattern not matching?**
```bash
# Test the pattern
omg-learn test <pattern-id> "your input"

# Simulate full execution
omg-learn simulate "your command"
```

**See logs:**
Uncomment debug logging in hook scripts:
```bash
# In pretool-checker.sh line 36:
echo "DEBUG: Tool: $TOOL_NAME, Input: $TOOL_INPUT" >> /tmp/omg-learn-hook.log
```

## License

See LICENSE file in the repository root.
