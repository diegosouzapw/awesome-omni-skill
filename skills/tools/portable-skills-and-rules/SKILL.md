---
name: portable-skills-and-rules
description: Defines SKILL.md structure, valid frontmatter fields, and cross-tool setup for Claude Code, Cursor, OpenCode, and Cline. ALWAYS read before creating or editing any SKILL.md file. This skill should be used when "writing a skill", "creating a skill", "editing a skill", "skill frontmatter", "SKILL.md format", "AGENTS.md", "CLAUDE.md", or setting up skills across multiple AI tools.
user-invocable: true
disable-model-invocation: false
---

# Cross-Tool Interoperability

Write skills, rules, and commands that work across Claude Code, OpenCode, Cursor, and Cline without duplication.

## Quick Start

All four tools canonically search `.claude/skills/` per the spec. However, Cursor has several bugs that require a workaround — see [Cursor Bugs](#cursor-bugs) below.

**In practice, always write new skills to `.cursor/skills/` and symlink `.claude/skills/` to it:**

```bash
# One-time setup per project
ln -sfn ../.cursor/skills .claude/skills
```

Skills use combined frontmatter that each tool reads selectively:

```yaml
---
name: my-skill
description: "Does X. Use when user asks to Y or mentions Z."
user-invocable: true
disable-model-invocation: false
---
```

Always set both invocation fields explicitly for visible defaults across all tools.

## Core Strategy

### Skills: Fully Shareable

The spec says `.claude/skills/` is the canonical location. In practice, Cursor bugs (see below) require the physical files to live in `.cursor/skills/`, with `.claude/skills/` as a symlink.

**Always write new skills to `.cursor/skills/`.** The symlink makes Claude Code and other tools pick them up automatically.

Create one SKILL.md with combined frontmatter. Each tool reads what it understands, ignores the rest.

**Notes**:
- `.cursor/skills/` holds the physical files — always write here
- `.claude/skills/` is a symlink to `.cursor/skills/` — never write directly to it
- Cline skills require enabling in Settings → Features → Enable Skills (experimental)
- Skills can be invoked via `/<skill-name>` slash commands in Claude Code, Cursor IDE, and Cursor CLI

### Cursor Bugs

Cursor has three known issues that drive the `.cursor/skills/` workaround:

1. **Doesn't search `.claude/skills/`** — Despite the spec, Cursor IDE does not reliably read skills from `.claude/skills/`. It only reliably finds skills in `.cursor/skills/`.

2. **Doesn't traverse above the open directory** — Cursor only searches for skills within the directory the IDE is opened to. It won't walk up to the git root. In a monorepo where the IDE is opened to a sub-project, skills defined at the repo root are invisible. **Workaround**: symlink the `.cursor/` directory into each sub-project that needs access.

3. **Skills don't survive two symlinks** — If `.claude/skills/` is a symlink and the skills inside are themselves symlinks (e.g., pointing up to a parent directory), Cursor fails to read them. Skills must be physically present at the resolved path, with at most one level of symlink indirection.

**Consequence**: Skills must be physically in `.cursor/skills/`. The symlink goes in the other direction: `.claude/skills/` → `.cursor/skills/`.

### Rules: Symlink Between Names

Claude Code uses `CLAUDE.md`. OpenCode, Cursor, and Cline use `AGENTS.md`. Content is identical—only the filename differs.

```bash
# Project level: pick one as source, symlink the other
ln -s AGENTS.md CLAUDE.md   # or vice versa

# Global level
ln -s ~/dotfiles/AGENTS.md ~/.claude/CLAUDE.md
ln -s ~/dotfiles/AGENTS.md ~/.config/opencode/AGENTS.md
```

Cline also supports a `.clinerules/` folder with multiple rule files. For portability, use `AGENTS.md`.

### Cursor Rules Bug: `.mdc` Extension Required

Cursor currently only detects `.mdc` files in `.cursor/rules/`, not `.md` files, despite documentation stating both formats are supported. This breaks the symlink strategy for modular rules.

**Temporary workaround (until bug is fixed):**
- Maintain separate `.cursor/rules/*.mdc` files alongside `.claude/rules/*.md`
- Keep content synchronized between the two (manually or via script)
- Use identical frontmatter (Claude reads `paths:`, Cursor reads `globs:`, both ignore unknown keys)
- When Cursor fixes the bug, delete `.cursor/rules/*.mdc` and replace with symlink: `.cursor/rules/` → `.claude/rules/`

**Note**: This only affects modular rules in `.cursor/rules/`. Root-level `AGENTS.md` works fine in Cursor without needing `.mdc` extension.

### Claude Code Rules Extensions

Claude Code supports several rules features beyond basic `CLAUDE.md`. These are Claude Code-only but degrade gracefully (other tools simply ignore them).

**CLAUDE.local.md** — Personal project-specific preferences. Auto-added to `.gitignore`. Place next to `CLAUDE.md` for private overrides (sandbox URLs, test data, personal workflow notes).

**CLAUDE.md imports** — Use `@path/to/file` syntax to import other files into CLAUDE.md:
```markdown
See @README for project overview and @docs/git-workflow.md for git conventions.
```
Relative paths resolve from the importing file. Recursive imports supported (max depth 5). Other tools see `@path` as literal text—harmless but not functional.

**Modular rules with `.claude/rules/`** — Split rules into focused files:
```
.claude/rules/
├── code-style.md       # Unconditional: always loaded
├── testing.md
└── api-design.md       # Can use paths: frontmatter for file-scoped rules
```
All `.md` files load automatically. Rules can be scoped to specific files via `paths` frontmatter with glob patterns. Cursor (`.cursor/rules/` with `globs`) and Cline (`.clinerules/` with `paths`) have similar systems but use their own directories — these are not portable across tools.

**Cursor bug**: Cursor currently requires `.mdc` extension for rules in `.cursor/rules/`, not `.md`. See [Cursor Rules Bug](#cursor-rules-bug-mdc-extension-required) section.

### Commands: Use Skills Instead

Claude Code has merged commands into skills — `.claude/commands/` files still work but skills take precedence if names collide. For maximum portability, write skills instead of commands. They work as:
- Slash commands in Claude Code, Cursor IDE, and Cursor CLI (`/<skill-name>`)
- Auto-discovered capabilities in OpenCode and Cline

**Migration**: Cursor 2.4+ includes `/migrate-to-skills` to convert existing commands to skills with `disable-model-invocation: true`.

Cline has a separate "workflows" system (`.clinerules/workflows/`), but skills are more portable.

### OpenCode: Command Wrappers for Slash Invocation

OpenCode skills are NOT slash-invocable — they are loaded on-demand by the agent via the `skill` tool. To get `/skill-name` slash invocation in OpenCode, **create a thin command wrapper** that tells the agent to load the skill.

Place command wrappers in `.opencode/commands/` (project) or `~/.config/opencode/commands/` (global):

```markdown
<!-- .opencode/commands/my-skill.md -->
---
description: Short description of what the skill does
---

Load the `my-skill` skill and follow its instructions. $ARGUMENTS
```

The filename becomes the command name (e.g., `my-skill.md` → `/my-skill`). The agent receives the prompt, calls the `skill` tool to load the SKILL.md content, and follows it.

**When creating a new skill, ALWAYS create a matching OpenCode command wrapper.** If commands are symlinked between `~/.claude/commands/` and `~/.config/opencode/commands/`, one file serves both Claude Code (as a command) and OpenCode (as a slash command).

**Key differences from skills:**
- Commands support `$ARGUMENTS` and positional args (`$1`, `$2`)
- Commands support `!`command`` for shell output injection
- Commands support `@filename` for file content inclusion
- Commands can specify `agent:` and `model:` overrides in frontmatter

## Writing Portable Skills

### Required Fields (All Tools)

| Field | Requirements |
|-------|-------------|
| `name` | 1-64 chars, lowercase, hyphens only, must match folder name, pattern: `^[a-z0-9]+(-[a-z0-9]+)*$` |
| `description` | Max 1024 chars, third person, include trigger phrases |
| `user-invocable` | Always set to `true` (Claude Code: show in slash menu) |
| `disable-model-invocation` | Always set to `false` (Cursor: allow auto-discovery) |

### Invocation Control Fields (Required)

**Always set both fields explicitly** for visible defaults. All skills should be user-invokable and auto-discoverable:

```yaml
user-invocable: true              # Claude Code: show in slash menu
disable-model-invocation: false   # Cursor: allow auto-discovery
```

| Field | Tool | Value | Effect |
|-------|------|-------|--------|
| `user-invocable: true` | Claude Code | Required | Show in slash menu, allow `/<skill-name>` |
| `disable-model-invocation: false` | Cursor | Required | Allow auto-discovery based on description |

**Note**: These fields have opposite semantics but setting both explicitly ensures consistent behavior:
- Claude Code reads `user-invocable`, ignores `disable-model-invocation`
- Cursor reads `disable-model-invocation`, ignores `user-invocable`
- OpenCode and Cline ignore both (auto-discover via `skill` tool; for slash commands in OpenCode, use command wrappers)

### Optional Fields (Include Only When Specified)

Do not add these fields by default. Only include when the user explicitly requests them:

- `allowed-tools` - Restricts tool access (Claude Code, experimental in Cursor)
- `license` - Software license (informational, recognized by Cursor + OpenCode)
- `metadata` - Author, version info (informational, recognized by Cursor + OpenCode)
- `compatibility` - Environment requirements (recognized by Cursor + OpenCode)

These fields degrade gracefully—tools ignore what they don't recognize.

**Example with all possible fields** (only add what you need):

```yaml
---
name: review-code
description: "Reviews code for quality and bugs. Use when asked to review changes."

# === REQUIRED: Invocation control (always set explicitly) ===
user-invocable: true              # Claude Code: show in slash menu
disable-model-invocation: false   # Cursor: allow auto-discovery

# === OPTIONAL: Add only when specified ===

# Shared (Cursor + OpenCode + Agent Skills spec)
license: MIT
compatibility: "requires git"
metadata:
  author: your-name
  version: 1.0.0

# Claude Code + Cursor (experimental)
allowed-tools: Read, Grep, Glob

# Claude Code only
model: claude-sonnet-4-20250514
context: fork
agent: Explore
argument-hint: "[file-or-directory]"
---
```

### File Structure

Use the same structure across all tools:

```
skill-name/
├── SKILL.md          # Required: instructions + combined frontmatter
├── scripts/          # Optional: executable code agents can run
├── references/       # Optional: detailed docs (loaded on-demand)
├── examples/         # Optional: usage examples
└── assets/           # Optional: templates, images, data files (Cursor)
```

## Setting Up Global Interoperability

Physical skills live in `.cursor/skills/`. All other tool locations symlink to it.

```bash
# Skills: physical files in .cursor/skills
mkdir -p ~/.cursor/skills

# Claude Code: symlink .claude/skills → .cursor/skills
ln -sfn ~/.cursor/skills ~/.claude/skills

# OpenCode: symlink to .cursor/skills
ln -sfn ~/.cursor/skills ~/.config/opencode/skills

# Cline: symlink to .cursor/skills
mkdir -p ~/.cline
ln -sfn ~/.cursor/skills ~/.cline/skills

# Rules: single source of truth
ln -sf ~/dotfiles/AGENTS.md ~/.claude/CLAUDE.md
ln -sf ~/dotfiles/AGENTS.md ~/.config/opencode/AGENTS.md
```

For project-level skills in a monorepo, symlink `.cursor/` into each sub-project so Cursor can find it (it won't traverse above the open directory):

```bash
# In each sub-project that needs access to skills
ln -sfn ../.cursor .cursor          # if sub-project is one level deep
ln -sfn ../.cursor/skills .claude/skills
```

See [scripts/setup-interop.sh](scripts/setup-interop.sh) for full automation.

## Hard Limitations

| Limitation | Impact | Workaround |
|------------|--------|------------|
| `hooks`, `context`, `agent` | Claude Code only | Degrade gracefully |
| `allowed-tools` | Claude Code + Cursor (experimental) | Degrade gracefully in OpenCode/Cline |
| `` !`command` `` dynamic injection | Claude Code skills + OpenCode commands | Appears as literal text in Cursor/Cline |
| `$ARGUMENTS` expansion | Claude Code + OpenCode expand | Cursor/Cline show unexpanded |
| Positional arg indexing | Claude Code: 0-based (`$0`, `$1`); OpenCode: 1-based (`$1`, `$2`) | Avoid positional args in portable skills |
| `${CLAUDE_SESSION_ID}` | Claude Code only | Other tools show unexpanded |
| `/<skill-name>` invocation | Claude Code + Cursor (IDE & CLI) | OpenCode: create command wrapper (see above). Cline: auto-discovery only |
| `@path` imports in CLAUDE.md | Claude Code only | Other tools see literal `@path` text |
| `CLAUDE.local.md` | Claude Code only | Other tools ignore it; no equivalent |
| File-scoped rules | Claude Code (`paths`), Cursor (`globs`), Cline (`paths`) — each in own rule dir | Not portable across tools; use tool-specific rule dirs |
| Cursor `.md` rules bug | Cursor only detects `.mdc` in `.cursor/rules/`, not `.md` | Duplicate rules as `.mdc` until bug fixed |
| Cline skills experimental | Must enable in settings | Toggle in Settings → Features |
| Cline `.clinerules/` folder | Cline-specific rules system | Use `AGENTS.md` for portability |
| Cline global skill precedence | Global overrides project skills | Use unique names across locations |

## Validation Checklist

Before finalizing a portable skill:

- [ ] `name` is lowercase with hyphens only, matches folder name
- [ ] `description` is under 1024 chars, third person
- [ ] `description` includes trigger phrases
- [ ] `user-invocable: true` is set explicitly
- [ ] `disable-model-invocation: false` is set explicitly
- [ ] SKILL.md body uses imperative form
- [ ] No tool-specific features in core logic
- [ ] Tool-specific frontmatter clearly commented
- [ ] OpenCode command wrapper created (if slash invocation needed)
- [ ] Tested in target tools

## References

### Tool-Specific Documentation

- **[references/claude-code.md](references/claude-code.md)** - Claude Code configuration and capabilities
- **[references/opencode.md](references/opencode.md)** - OpenCode configuration and capabilities
- **[references/cursor.md](references/cursor.md)** - Cursor configuration and capabilities
- **[references/cline.md](references/cline.md)** - Cline configuration and capabilities

### Examples and Scripts

- **[examples/universal-skill.md](examples/universal-skill.md)** - Complete portable skill template
- **[scripts/setup-interop.sh](scripts/setup-interop.sh)** - Automated setup script
