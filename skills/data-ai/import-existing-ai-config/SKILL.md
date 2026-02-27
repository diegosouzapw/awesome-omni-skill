---
name: import-existing-ai-config
description: Import existing AI tool configurations (from Claude, Copilot, or Cursor) into universal-ai-config templates. Converts target-specific files into universal templates.
argumentHint: '[target: claude|copilot|cursor]'
---

# Import Existing AI Config

Convert existing target-specific AI configuration files into universal-ai-config templates. The user must specify which target to import from.

## Usage

The user should specify the source target: `claude`, `copilot`, or `cursor`.

## Import Process

### 1. Identify Source Files

Scan the target's config directory for existing configuration files:

**Claude** (`.claude/`):

- Instructions: `.claude/rules/*.md` — each file has `description` and optional `paths` frontmatter
- Skills: `.claude/skills/*/SKILL.md` — skill directories with frontmatter (`name`, `description`, `allowed-tools`, `model`, `context`, `agent`, `disable-model-invocation`, `user-invocable`, `argument-hint`, `hooks`). Skill directories may also contain extra supporting files (references, examples, scripts) alongside `SKILL.md`.
- Agents: `.claude/agents/*.md` — agent files with frontmatter (`name`, `description`, `tools`, `disallowedTools`, `permissionMode`, `skills`, `hooks`, `memory`, `model`)
- Hooks: `.claude/settings.json` → `hooks` key — JSON with PascalCase event names (`SessionStart`, `SessionEnd`, `UserPromptSubmit`, `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `Stop`, `SubagentStart`, `SubagentStop`, `PreCompact`, `PermissionRequest`, `Notification`)
- MCP: `.mcp.json` — JSON with `mcpServers` wrapper containing server configs (`type`, `command`, `args`, `env`, `url`, `headers`)
- Commands (deprecated): `.claude/commands/*.md` — single-file slash commands with optional frontmatter (`description`, `allowed-tools`, `argument-hint`, `model`). May include subdirectories for namespacing (e.g. `.claude/commands/frontend/component.md` — command name is `component`). Body may use `$ARGUMENTS` placeholder, `!` prefix for bash execution, and `@` prefix for file references.

**Copilot** (`.github/`):

- Instructions: `.github/copilot-instructions.md` (always-apply) and `.github/instructions/*.instructions.md` (with `applyTo` frontmatter)
- Skills: `.github/skills/*/SKILL.md` — skill directories with frontmatter (`name`, `description`, `license`, `compatibility`, `metadata`). May contain extra supporting files alongside `SKILL.md`.
- Agents: `.github/agents/*.agent.md` — agent files with frontmatter (`name`, `description`, `tools`, `model`, `target`, `mcp-servers`, `handoffs`)
- Hooks: `.github/hooks/hooks.json` — JSON with version field and camelCase event names (`sessionStart`, `sessionEnd`, `userPromptSubmitted`, `preToolUse`, `postToolUse`, `errorOccurred`)
- MCP: `.vscode/mcp.json` — JSON with `servers` wrapper (not `mcpServers`), may include `inputs` array for interactive secret prompts

**Cursor** (`.cursor/`):

- Instructions: `.cursor/rules/*.mdc` or `.cursor/rules/*.md` — with `description`, `globs`, `alwaysApply` frontmatter
- Skills: `.cursor/skills/*/SKILL.md` — skill directories with frontmatter (`name`, `description`, `license`, `compatibility`, `metadata`, `disable-model-invocation`). May contain extra supporting files alongside `SKILL.md`.
- Hooks: `.cursor/hooks.json` — JSON with version field and camelCase event names (`sessionStart`, `sessionEnd`, `beforeSubmitPrompt`, `preToolUse`, `postToolUse`, `postToolUseFailure`, `stop`, `subagentStart`, `subagentStop`, `preCompact`, plus Cursor-specific events like `beforeShellExecution`, `afterFileEdit`)
- MCP: `.cursor/mcp.json` — JSON with `mcpServers` wrapper, omits `type` field (Cursor infers transport from `command` vs `url`)
- Note: Cursor does not have agents

### 2. Convert to Universal Format

For each file found, convert it to a universal-ai-config template:

**Frontmatter mapping** (target-specific → universal):

| Claude                     | Copilot                   | Cursor                     | Universal               |
| -------------------------- | ------------------------- | -------------------------- | ----------------------- |
| `paths`                    | `applyTo`                 | `globs`                    | `globs`                 |
| (no paths field)           | (copilot-instructions.md) | `alwaysApply: true`        | `alwaysApply: true`     |
| `disable-model-invocation` | —                         | `disable-model-invocation` | `disableAutoInvocation` |
| `user-invocable`           | —                         | —                          | `userInvocable`         |
| `allowed-tools`            | —                         | —                          | `allowedTools`          |
| `context: fork`            | —                         | —                          | `forkContext: true`     |
| `agent`                    | —                         | —                          | `subagentType`          |
| `argument-hint`            | —                         | —                          | `argumentHint`          |
| `hooks`                    | —                         | —                          | `hooks`                 |
| (commands: manual-only)    | —                         | —                          | `disableAutoInvocation` |
| —                          | `excludeAgent`            | —                          | `excludeAgent`          |
| —                          | `license`                 | `license`                  | `license`               |
| —                          | `compatibility`           | `compatibility`            | `compatibility`         |
| —                          | `metadata`                | `metadata`                 | `metadata`              |
| —                          | `target`                  | —                          | `target`                |
| —                          | `mcp-servers`             | —                          | `mcpServers`            |
| —                          | `handoffs`                | —                          | `handoffs`              |

**Hook event mapping** (target-specific → universal):

| Claude               | Copilot               | Cursor               | Universal            |
| -------------------- | --------------------- | -------------------- | -------------------- |
| `SessionStart`       | `sessionStart`        | `sessionStart`       | `sessionStart`       |
| `SessionEnd`         | `sessionEnd`          | `sessionEnd`         | `sessionEnd`         |
| `UserPromptSubmit`   | `userPromptSubmitted` | `beforeSubmitPrompt` | `userPromptSubmit`   |
| `PreToolUse`         | `preToolUse`          | `preToolUse`         | `preToolUse`         |
| `PostToolUse`        | `postToolUse`         | `postToolUse`        | `postToolUse`        |
| `PostToolUseFailure` | —                     | `postToolUseFailure` | `postToolUseFailure` |
| `Stop`               | —                     | `stop`               | `stop`               |
| `SubagentStart`      | —                     | `subagentStart`      | `subagentStart`      |
| `SubagentStop`       | —                     | `subagentStop`       | `subagentStop`       |
| `PreCompact`         | —                     | `preCompact`         | `preCompact`         |
| `PermissionRequest`  | —                     | —                    | `permissionRequest`  |
| `Notification`       | —                     | —                    | `notification`       |
| —                    | `errorOccurred`       | —                    | `errorOccurred`      |

Cursor-specific events (e.g. `beforeShellExecution`, `afterFileEdit`) should be preserved as-is — they pass through to Cursor and are dropped for other targets.

**Hook handler field mapping:**

| Claude                      | Copilot      | Cursor    | Universal |
| --------------------------- | ------------ | --------- | --------- |
| `command`                   | `bash`       | `command` | `command` |
| `timeout`                   | `timeoutSec` | `timeout` | `timeout` |
| `matcher` (in parent group) | —            | `matcher` | `matcher` |

**MCP server conversion** (target-specific → universal):

The universal MCP format uses `mcpServers` as the wrapper key, with each server having `type`, `command`, `args`, `env`, `url`, and `headers` fields.

| Source                     | Conversion                                                                          |
| -------------------------- | ----------------------------------------------------------------------------------- |
| Claude `.mcp.json`         | Copy `mcpServers` as-is (already matches universal format)                          |
| Copilot `.vscode/mcp.json` | Rename `servers` → `mcpServers`, add `type` field if missing, copy `inputs`         |
| Cursor `.cursor/mcp.json`  | Copy `mcpServers`, add `type: "stdio"` for servers with `command` (Cursor omits it) |

### 3. Write Universal Templates

For each converted file:

1. Use the same base name as the source file (e.g. `.claude/rules/coding-style.md` → `<%= instructionTemplatePath('coding-style') %>`)
2. Write the universal frontmatter and body content. Copy and paste the body content, instead of trying to use EJS `include()` or something equivalent, because the source files will be deleted afterwards.
3. If a template with the same name already exists, **overwrite it** (the import represents the latest version)

**File placement:**

- Instructions → `<%= instructionTemplatePath('{name}') %>`
- Skills → `<%= skillTemplatePath('{name}') %>`
- Agents → `<%= agentTemplatePath('{name}') %>`
- Hooks → `<%= hookTemplatePath('{source-name}') %>`
- MCP → `<%= mcpTemplatePath('{source-name}') %>`
- Commands → `<%= skillTemplatePath('{name}') %>` (commands convert to skills)

### 4. Handle Special Cases

- **Claude hooks**: Extract the `hooks` key from `.claude/settings.json`. Flatten the nested matcher group structure into individual handlers with `matcher` fields.
- **Copilot always-apply**: Convert `.github/copilot-instructions.md` to an instruction with `alwaysApply: true`.
- **Copilot hook `bash` field**: Convert to universal `command` field.
- **Copilot hook `timeoutSec` field**: Convert to universal `timeout` field (both use seconds).
- **Cursor `.mdc` files**: Read as regular markdown (the `.mdc` extension is just a convention).
- **Copilot MCP `servers` key**: Rename to `mcpServers` in the universal template.
- **Copilot MCP `inputs` array**: Preserve as-is — it's included in Copilot output only, ignored by Claude/Cursor.
- **Cursor MCP missing `type`**: Add `"type": "stdio"` for servers with `command`, or `"type": "sse"` for servers with `url`.
- **MCP env var references**: Leave `${ENV_VAR}` syntax as-is — it's passed through to generated output. If values look like they could be config variables, consider converting to `{{varName}}` syntax and adding a `variables` entry in the config file.
- **Fields that only exist for one target**: Preserve them as-is. They'll be passed through to matching targets and ignored by others.
- **Skill extra files**: When importing skill directories, copy **all** files in the directory — not just `SKILL.md`. Extra files (references, examples, scripts, data) should be placed in the same relative paths within the universal template's skill directory (e.g. `.claude/skills/my-skill/references/example.md` → `<%= skillTemplatePath('my-skill') %>/../references/example.md`, i.e. `skills/my-skill/references/example.md` in the templates dir). During generation, `.md` extra files are rendered through EJS (with access to `target`, `config`, path helpers), while non-`.md` files are copied as-is.
- **Claude commands → skills**: Commands are single `.md` files but skills are directories. Create a skill directory and place the converted command content as `SKILL.md` inside it. Use the filename (without `.md`) as the skill name.
- **Claude commands: `disableAutoInvocation`**: Commands are manual-only (no auto-invocation by the AI). Set `disableAutoInvocation: true` on the converted skill to preserve this behavior.
- **Claude commands: namespaced subdirectories**: Subdirectories in `.claude/commands/` are organizational only — they don't affect the command name. Use the filename as the skill name. If two commands from different subdirectories share a filename, disambiguate by prefixing with the subdirectory name (e.g. `frontend-component`).
- **Claude commands: rewrite body to agent-agnostic language**: Command bodies use Claude-specific syntax that must be converted to plain, agent-agnostic instructions:
  - `$ARGUMENTS` → Describe what input is expected. E.g., `Fix issue #$ARGUMENTS` becomes "The user will specify an issue number when invoking this skill. Fix the GitHub issue specified by the user."
  - `!\`command\``(pre-executed bash) → Convert to step-by-step instructions for the agent to run itself. E.g.,`!\`git status\``becomes "Run`git status` to see current changes". The agent runs these as tool calls rather than having them pre-executed.
  - `@filepath` (file references) → Convert to instructions to read those files. E.g., `@src/utils/helpers.js` becomes "Read the file `src/utils/helpers.js`".
- **Claude commands: personal commands**: Personal commands from `~/.claude/commands/` are user-level, not project-level. Only import project commands (`.claude/commands/`) by default — flag personal commands and ask the user whether to include them.

### 5. Verify

After importing, run `uac generate` targeting all configured targets and compare the generated output against the original source files to ensure the conversion is accurate.

Report to the user:

- How many files were imported per type
- Any files that couldn't be converted (with reasons)
- Whether the generated output matches the originals
