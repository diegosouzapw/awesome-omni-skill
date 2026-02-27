---
name: list
description: List Claude Code components. Actions: env-vars, sessions, settings, skills, tools.
argument-hint: <what> (env-vars|sessions|settings|skills|tools)
allowed-tools: Bash, Read, Glob, Grep, Task, Skill, WebFetch, WebSearch
---

# List Claude Code Components

List various Claude Code components based on the requested action.

## Argument Routing

| Action | Description |
|--------|-------------|
| `skills` (default) | List all available skills with descriptions |
| `env-vars` | List environment variables (configured or researched) |
| `sessions` | List recent sessions with sizes, ages, resumability |
| `settings` | List settings.json configuration options |
| `tools` | List core Claude Code tools with parameters |

When invoked without arguments, default to `skills`.

Parse `$ARGUMENTS` to determine the action. The first token is the action keyword. Remaining tokens are passed as sub-arguments to the action handler.

---

## Action: skills

List ALL available Claude Code skills by running the list_skills.py script.

### Step 1: Find the Script

Locate list_skills.py in the claude-ecosystem plugin:

```bash
# Check installed plugins first
find ~/.claude/plugins -name "list_skills.py" -path "*claude-ecosystem*" 2>/dev/null | head -1
```

If not found, check current repo (for development):

```bash
find . -name "list_skills.py" -path "*claude-ecosystem*" 2>/dev/null | head -1
```

### Step 2: Run the Script

Execute with Python using the path found in Step 1:

```bash
python "<found-path>/list_skills.py"
```

### What the Script Scans

The script finds ALL skills from:

- **Personal skills**: `~/.claude/skills/*/SKILL.md`
- **Project skills**: `.claude/skills/*/SKILL.md` (current working directory)
- **Plugin skills**: `~/.claude/plugins/marketplaces/*/*/skills/*/SKILL.md`

### Output Format

The script outputs formatted markdown:

```text
## Personal Skills (~/.claude/skills/)
### **skill-name**
Description from SKILL.md frontmatter.
---

## Project Skills (.claude/skills/)
### **skill-name**
Description from SKILL.md frontmatter.
---

## Plugin Skills
### **plugin-name:skill-name**
Description from SKILL.md frontmatter.
---

**Total: X skills** (Y personal, Z project, W plugin)
```

Output the script results directly - they are already formatted.

---

## Action: env-vars

Research and list ALL known Claude Code environment variables from multiple sources, or show currently configured variables across all settings scopes.

### Sub-arguments

| Argument | Description |
|----------|-------------|
| `--configured` | Show currently configured env vars from all settings files |
| `--official-only` | Only show variables documented in official docs |
| `--changelog-only` | Only extract from CHANGELOG.md |
| `--full-research` | Run comprehensive MCP research (slower, more complete) |
| (none) | Default: official docs + changelog extraction |

### Behavior Matrix

| Arguments | Behavior |
|-----------|----------|
| (none) | Research: official docs + changelog |
| `--configured` | Read current settings files only (no research) |
| `--official-only` | Research: official docs only |
| `--changelog-only` | Research: changelog extraction only |
| `--full-research` | Research: all sources including MCP agents |

### Mode: --configured

When `--configured` is passed, skip all research and read currently configured environment variables from all settings scopes.

#### Settings File Hierarchy (Precedence: Enterprise < User < Project < Local)

| Scope | Path | Description |
|-------|------|-------------|
| Enterprise | `~/.claude/enterprise/settings.json` | IT-managed defaults |
| User/Global | `~/.claude/settings.json` | User preferences |
| Project | `.claude/settings.json` | Project-specific settings |
| Local | `.claude/settings.local.json` | Gitignored local overrides |

#### Workflow

1. Read each settings file if it exists
2. Extract `env` section from each
3. Calculate effective values (later scopes override earlier)
4. Display merged view with source indication

#### Output Format

```markdown
## Currently Configured Environment Variables

| Variable | Value | Source | Overridden By |
|----------|-------|--------|---------------|
| DISABLE_TELEMETRY | "1" | user | project |
| ANTHROPIC_API_KEY | "sk-***" | user | - |
| CLAUDE_HOOK_LOG_DIR | "./logs" | project | - |
| DISABLE_AUTOUPDATER | "0" | enterprise | - |

### By Scope

**Enterprise** (~/.claude/enterprise/settings.json)
- DISABLE_AUTOUPDATER = "0"

**User** (~/.claude/settings.json)
- DISABLE_TELEMETRY = "1"
- ANTHROPIC_API_KEY = "sk-***" (masked)

**Project** (.claude/settings.json)
- DISABLE_TELEMETRY = "0" (overrides user)
- CLAUDE_HOOK_LOG_DIR = "./logs"

**Local** (.claude/settings.local.json)
(none configured)
```

#### Security Note

Mask sensitive values in output:

- `ANTHROPIC_API_KEY` -> `"sk-***"` (show prefix only)
- `AWS_*` credentials -> masked
- `GOOGLE_*` credentials -> masked

### Mode: Research (Default)

When no `--configured` flag, research environment variables from multiple sources.

#### Step 1: Parallel Research via Agents

Launch research agents in parallel to gather env var information from multiple sources.

##### Agent 1: MCP Research (full-research mode only)

Spawn `mcp-research` agent:

```text
Research Claude Code environment variables using perplexity and firecrawl.

Search for:
- "Claude Code environment variables configuration"
- "anthropic claude cli env vars"
- "ANTHROPIC_API_KEY CLAUDE_CODE environment"

Extract ALL environment variables mentioned, including:
- Variable name
- Default value (if mentioned)
- Purpose/description
- Source URL

Return structured findings with source citations.
```

##### Agent 2: Official Documentation

Spawn `claude-code-guide` agent:

```text
First WebFetch https://code.claude.com/docs/en/claude_code_docs_map.md to find
relevant doc pages about environment variables and settings.

Then WebFetch these specific pages:
- Settings/configuration pages
- Environment variables documentation
- Any pages mentioning ANTHROPIC_*, CLAUDE_*, DISABLE_*, ENABLE_*

Do NOT use Skill tool (not available).
Return complete env var tables with source URLs.
```

##### Skill: Local Documentation Cache

Invoke `docs-management` skill with query:

```text
Search for "environment variables" and "env" in Claude Code documentation.
Include any settings, configuration, or customization topics.
```

#### Step 2: Changelog Extraction

WebFetch the CHANGELOG.md to extract environment variables:

```text
URL: https://raw.githubusercontent.com/anthropics/claude-code/main/CHANGELOG.md
```

Extract all patterns matching:

- `ANTHROPIC_*`
- `CLAUDE_*`
- `DISABLE_*`
- `ENABLE_*`
- `*_TIMEOUT`
- `*_API_KEY`
- `HTTP_PROXY`, `HTTPS_PROXY`, `NO_PROXY`

Note the version where each variable was introduced.

#### Step 3: Consolidate and Categorize

Merge findings from all sources, deduplicate, and categorize:

| Category | Criteria |
|----------|----------|
| **OFFICIAL** | Documented on code.claude.com/docs |
| **CHANGELOG** | In CHANGELOG but not in official docs |
| **DISCOVERED** | Found via MCP research only |

#### Step 4: Output Report

Format the complete list as structured tables:

```markdown
## Claude Code Environment Variables

### Official (Documented)
| Variable | Default | Purpose | Category |
|----------|---------|---------|----------|
| ANTHROPIC_API_KEY | (required) | API key for Anthropic API | Auth |
| CLAUDE_CODE_USE_BEDROCK | "0" | Use AWS Bedrock instead of Anthropic API | Provider |
| CLAUDE_CODE_USE_VERTEX | "0" | Use Google Vertex AI | Provider |
| ... | ... | ... | ... |

### Changelog (Undocumented)
| Variable | Default | Purpose | Version Added |
|----------|---------|---------|---------------|
| CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR | "0" | Keep project working directory | v2.x |
| ... | ... | ... | ... |

### Discovered (MCP Research)
| Variable | Default | Purpose | Source |
|----------|---------|---------|--------|
| ... | ... | ... | [source URL] |

---

## Sources
- Official: https://code.claude.com/docs/en/settings
- Changelog: https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
- MCP: perplexity, firecrawl (if --full-research)
```

### Known Environment Variables (Quick Reference)

These are commonly used variables (verify against current docs):

#### Authentication & Provider

- `ANTHROPIC_API_KEY` - API key (required unless using Bedrock/Vertex)
- `CLAUDE_CODE_USE_BEDROCK` - Use AWS Bedrock
- `CLAUDE_CODE_USE_VERTEX` - Use Google Vertex AI

#### Proxy & Network

- `HTTP_PROXY` / `HTTPS_PROXY` - Proxy configuration
- `NO_PROXY` - Proxy bypass list

#### Behavior

- `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` - Working directory behavior
- `DISABLE_AUTOUPDATER` - Disable auto-updates
- `DISABLE_TELEMETRY` - Disable telemetry

#### Hooks

- `CLAUDE_HOOK_*` - Custom hook environment variables (repo-specific)

**Note:** This quick reference may be outdated. Always run the full command for current information.

---

## Action: sessions

Display recent Claude Code sessions for the current project (or all projects), showing size, age, and whether they can be resumed.

### Sub-arguments

| Argument | Description | Default |
|----------|-------------|---------|
| `--all` | List sessions from all projects | false |
| `--limit N` | Maximum sessions to display | 10 |
| `--agents` | Include agent transcripts in listing | false |

### Step 1: Determine Target Directory

```bash
if ALL_PROJECTS; then
  TARGET_DIR="$HOME/.claude/projects"
else
  PROJECT_PATH=$(pwd | sed 's/[\/:]/-/g' | sed 's/^-//')
  TARGET_DIR="$HOME/.claude/projects/$PROJECT_PATH"
fi
```

### Step 2: List Sessions

```bash
# Find session files (not agent files), sorted by modification time
find "$TARGET_DIR" -name "*.jsonl" ! -name "agent-*" -type f -printf '%T@ %s %p\n' 2>/dev/null | \
  sort -rn | head -$LIMIT | while read timestamp size path; do
    filename=$(basename "$path")
    age=$(( ($(date +%s) - ${timestamp%.*}) / 86400 ))
    size_h=$(numfmt --to=iec-i --suffix=B $size)
    echo "$filename  $size_h  ${age}d ago"
done
```

### Step 3: Display Results

```text
Recent Sessions
===============
Project: D--repos-gh-melodic-claude-code-plugins

 #  Session ID                              Size    Age     Resumable
--- --------------------------------------- ------- ------- ---------
 1  4165caab-4f86-4fc8-a901-fdcf2307a7cd   8.4M    today   Yes
 2  faf4b0f3-1f1f-438a-b40b-1a7ef4cb1718   7.9M    today   Yes
 3  55e42424-f5b6-4a26-98d5-1bed9176bf50   15M     1d      Yes
 4  336198a5-dfa4-4266-86dc-3d90158b8e30   15M     2d      Yes
 5  83d6bc93-3337-4b9e-8818-7141effbbcf0   14M     4d      Yes
...

Total: 507 sessions (645M)
Tip: Use /cleanup-sessions 7 to remove old sessions
```

### Notes

- Sessions can be resumed using `claude --resume {session-id}`
- Session IDs are UUIDs assigned by Claude Code
- Large sessions (>10MB) indicate long conversations
- Use `/session-stats` for aggregate statistics

---

## Action: settings

Research and list all known Claude Code `settings.json` configuration options, show the current settings hierarchy with precedence, or display only current project settings.

**Note:** Environment variables are documented separately. Use the `env-vars` action for environment variable documentation.

### Sub-arguments

| Argument | Description |
|----------|-------------|
| (none) | Default: Full hierarchy - show user/project/local/managed settings with precedence |
| `--fields-only` | Schema reference - list all available fields with types and defaults |
| `--current` | Show current project settings values only (no hierarchy) |
| `--category <name>` | Filter by category: `core`, `permissions`, `sandbox`, `hooks`, `plugins`, `attribution` |
| `--scope <scope>` | Limit to specific scope: `user`, `project`, `local`, `managed` |

### Behavior Matrix

| Arguments | Behavior |
|-----------|----------|
| (none) | Read all settings files, display hierarchy with precedence |
| `--fields-only` | Research schema, display available fields with types/defaults |
| `--current` | Read project settings only, display explicit values |
| `--category X` | Filter output to specific category (works with all modes) |
| `--scope X` | Limit hierarchy to specific scope (hierarchy mode only) |

### Settings File Hierarchy

Claude Code settings follow a precedence order (higher overrides lower):

| Priority | Scope | Path | Description |
|----------|-------|------|-------------|
| 1 (Highest) | Managed | `~/.claude/managed/settings.json` | IT-enforced (cannot be overridden) |
| 2 | CLI Args | `--model`, etc. | Command line overrides |
| 3 | Local | `.claude/settings.local.json` | Gitignored local overrides |
| 4 | Project | `.claude/settings.json` | Shared project settings |
| 5 (Lowest) | User | `~/.claude/settings.json` | User preferences |

### Mode: Default (Full Hierarchy)

When no arguments are provided, display the complete settings hierarchy showing precedence.

#### Workflow

1. **Read all settings files** that exist:
   - `~/.claude/managed/settings.json` (managed)
   - `~/.claude/settings.json` (user)
   - `.claude/settings.json` (project)
   - `.claude/settings.local.json` (local)

2. **Calculate effective values** - Later scopes override earlier (except managed which is enforced)

3. **Display merged view** with source indication

#### Output Format

```markdown
## Settings Hierarchy (Effective Configuration)

### Precedence Order (highest to lowest)
1. Managed settings (enforced) - cannot be overridden
2. Command line arguments
3. Local project settings (.claude/settings.local.json)
4. Shared project settings (.claude/settings.json)
5. User settings (~/.claude/settings.json)

### Core Settings
| Setting | Effective Value | Source | Overridden? |
|---------|-----------------|--------|-------------|
| model | opus | project | - |
| cleanupPeriodDays | 30 | default | - |
| respectGitignore | true | default | - |
| plansDirectory | ~/.claude/plans | default | - |
| showTurnDuration | true | default | - |

### Permissions Settings
| Setting | Effective Value | Source | Overridden? |
|---------|-----------------|--------|-------------|
| defaultMode | bypassPermissions | project | user |
| allow | [list...] | project | - |
| deny | [list...] | project | - |

### Sandbox Settings
(Show if sandbox configured, otherwise note "Not configured - defaults to disabled")

### Hook Settings
| Setting | Effective Value | Source | Overridden? |
|---------|-----------------|--------|-------------|
| disableAllHooks | false | project | - |
| hooks | {} | default | - |

### Status Line
(Show statusLine configuration if set)

### Plugin Settings
(Show enabledPlugins if configured)

---

## Environment Variables
Use `list env-vars` action for environment variable documentation.
```

### Mode: --fields-only (Schema Reference)

When `--fields-only` is passed, research and display all available settings fields with types and defaults.

#### Workflow

1. **Research schema** via docs-management skill and MCP servers
2. **Extract all known fields** from official docs, changelog, and schema
3. **Categorize and display** with types, defaults, and version info

#### Research Sources

##### Source 1: docs-management Skill

Invoke `docs-management` skill:

```text
Search for "settings.json" and "configuration" in Claude Code documentation.
Include settings schema, available fields, and configuration options.
```

##### Source 2: MCP Research (comprehensive)

Spawn `mcp-research` agent:

```text
Research Claude Code settings.json schema and configuration options using perplexity and firecrawl.

Search for:
- "Claude Code settings.json schema"
- "claude code configuration options"
- Site:code.claude.com settings

Extract ALL settings fields with:
- Field name
- Type (string, boolean, number, object, array)
- Default value
- Description
- Version introduced (if known)

Return structured findings with source citations.
```

##### Source 3: JSON Schema

Reference the schema at: `https://json.schemastore.org/claude-code-settings.json`

#### Output Format

```markdown
## Available Settings Fields

### Core Settings
| Field | Type | Default | Description | Since |
|-------|------|---------|-------------|-------|
| $schema | string | - | JSON schema reference | - |
| model | string | "sonnet" | Default model alias or full ID | - |
| language | string | (system) | Interface language | v2.1.0 |
| autoUpdatesChannel | enum | "latest" | "stable" or "latest" | - |
| cleanupPeriodDays | number | 30 | Days before session cleanup | - |
| respectGitignore | boolean | true | Honor .gitignore patterns | - |
| outputStyle | string | null | Output formatting style | - |
| alwaysThinkingEnabled | boolean | false | Enable extended thinking | - |
| plansDirectory | string | ~/.claude/plans | Plan file storage | v2.1.9 |
| showTurnDuration | boolean | true | Show turn timing | v2.1.7 |

### Permissions Settings
| Field | Type | Default | Description | Since |
|-------|------|---------|-------------|-------|
| permissions.defaultMode | enum | "default" | default/acceptEdits/plan/dontAsk/bypassPermissions | - |
| permissions.allow | array | [] | Allowed tool patterns | - |
| permissions.deny | array | [] | Denied tool patterns | - |
| permissions.ask | array | [] | Always-ask tool patterns | - |
| permissions.additionalDirectories | array | [] | Extra allowed directories | - |
| permissions.disableBypassPermissionsMode | string | null | Set to "disable" to block --dangerously-skip-permissions | - |

### Sandbox Settings (macOS/Linux only)
| Field | Type | Default | Description | Since |
|-------|------|---------|-------------|-------|
| sandbox.enabled | boolean | false | Enable sandboxed bash | - |
| sandbox.autoAllowBashIfSandboxed | boolean | true | Auto-allow bash in sandbox | - |
| sandbox.excludedCommands | array | [] | Commands bypassing sandbox | - |
| sandbox.allowUnsandboxedCommands | boolean | true | Allow unsandboxed commands | - |
| sandbox.enableWeakerNestedSandbox | boolean | false | Linux nested sandbox | - |
| sandbox.network.allowUnixSockets | array | [] | Allowed Unix sockets | - |
| sandbox.network.allowLocalBinding | boolean | false | Allow local port binding | - |
| sandbox.network.httpProxyPort | number | null | HTTP proxy port | - |
| sandbox.network.socksProxyPort | number | null | SOCKS proxy port | - |

### Hook Settings
| Field | Type | Default | Description | Since |
|-------|------|---------|-------------|-------|
| hooks | object | {} | Hook configuration by event | - |
| disableAllHooks | boolean | false | Disable all hooks globally | - |
| allowManagedHooksOnly | boolean | false | Only allow managed hooks | - |

### Attribution Settings
| Field | Type | Default | Description | Since |
|-------|------|---------|-------------|-------|
| attribution.commit | string | null | Custom commit attribution | - |
| attribution.pr | string | null | Custom PR attribution | - |

### Status Line Settings
| Field | Type | Default | Description | Since |
|-------|------|---------|-------------|-------|
| statusLine.type | enum | - | "command" for custom status | - |
| statusLine.command | string | - | Status line command | - |
| statusLine.padding | number | 0 | Status line padding | - |
| statusLine.timeout | number | 5000 | Command timeout (ms) | - |

### Plugin Settings
| Field | Type | Default | Description | Since |
|-------|------|---------|-------------|-------|
| enabledPlugins | object | {} | Enabled plugins by namespace | - |
| extraKnownMarketplaces | object | {} | Additional plugin marketplaces | - |

### Environment Variables
| Field | Type | Default | Description | Since |
|-------|------|---------|-------------|-------|
| env | object | {} | Environment variables | - |

**Note:** For detailed environment variable documentation, use the `env-vars` action.

---

## Sources
- Schema: https://json.schemastore.org/claude-code-settings.json
- Docs: https://code.claude.com/docs/en/settings
- [MCP research citations]
```

### Mode: --current (Current Project Only)

When `--current` is passed, display only the explicitly configured values in the current project's settings.

#### Workflow

1. Read `.claude/settings.json` (project settings)
2. Display only explicitly set values (not defaults)
3. Skip research and hierarchy calculation

#### Output Format

```markdown
## Current Project Settings

**File:** `.claude/settings.json`

### Explicitly Configured Values

#### Core Settings
(none explicitly set - using defaults)

#### Environment Variables
54 variables configured - see `.claude/settings.json` or use `list env-vars --configured`

#### Permissions
- defaultMode: "bypassPermissions"
- allow: 26 patterns configured
- deny: 24 patterns configured

#### Hooks
- disableAllHooks: false

#### Status Line
- type: "command"
- command: "bun x ccusage statusline ..."
- padding: 0
- timeout: 5000

---

Use `--fields-only` to see all available settings fields.
Use `list env-vars` for environment variable details.
```

### Category Definitions

| Category | Fields Included |
|----------|-----------------|
| `core` | model, language, autoUpdatesChannel, cleanupPeriodDays, respectGitignore, outputStyle, alwaysThinkingEnabled, plansDirectory, showTurnDuration |
| `permissions` | permissions.* (all permission fields) |
| `sandbox` | sandbox.* (all sandbox fields) |
| `hooks` | hooks, disableAllHooks, allowManagedHooksOnly |
| `plugins` | enabledPlugins, extraKnownMarketplaces |
| `attribution` | attribution.commit, attribution.pr |

---

## Action: tools

Introspect your current tool context and enumerate all core Claude Code tools with their complete parameter schemas.

### Scope

List only **core built-in tools** (not installed MCP server tools):

- **File Operations**: Read, Write, Edit, Glob, Grep, NotebookEdit
- **Execution**: Bash, Task, TaskOutput, KillShell
- **User Interaction**: AskUserQuestion, TodoWrite
- **Planning & Workflow**: EnterPlanMode, ExitPlanMode
- **Web & Network**: WebFetch, WebSearch
- **Code Intelligence**: LSP, Skill
- **MCP Resources**: ListMcpResourcesTool, ReadMcpResourceTool

### Sub-arguments

| Argument | Description |
|----------|-------------|
| `--category <cat>` | Filter by category: `file`, `execution`, `interaction`, `planning`, `web`, `code`, `mcp` |
| `--tool <name>` | Show detailed info for a specific tool (e.g., `--tool Read`) |

### Execution

#### Step 1: Parse Arguments

Parse any provided arguments:

- If `--category` specified, note the category filter
- If `--tool` specified, note the specific tool name

#### Step 2: Introspect Tool Context

Examine your current system context to enumerate all available core tools. For each tool, extract:

1. **Tool name** - The function name
2. **Description** - What the tool does (from the tool definition)
3. **Parameters** - All parameters with:
   - Name
   - Type (string, number, boolean, array, object, enum)
   - Required or optional
   - Default value (if any)
   - Constraints (min, max, enum values)
   - Description

#### Step 3: Categorize

Group tools into these categories:

| Category | Tools |
|----------|-------|
| `file` | Read, Write, Edit, Glob, Grep, NotebookEdit |
| `execution` | Bash, Task, TaskOutput, KillShell |
| `interaction` | AskUserQuestion, TodoWrite |
| `planning` | EnterPlanMode, ExitPlanMode |
| `web` | WebFetch, WebSearch |
| `code` | LSP, Skill |
| `mcp` | ListMcpResourcesTool, ReadMcpResourceTool |

#### Step 4: Apply Filters

- If `--category` was specified, show only tools in that category
- If `--tool` was specified, show only that tool with full detailed view

#### Step 5: Output

**Summary View (default or --category):**

```markdown
## Core Claude Code Tools (N tools)

### Category Name (N tools)

| Tool | Description |
|------|-------------|
| ToolName | Brief description |
| ... | ... |
```

**Detailed View (--tool ToolName):**

```markdown
## ToolName

Full description from tool definition.

### Parameters

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| param_name | type | Yes/No | value | Description |

### Constraints

- List any enum values, min/max, format requirements

### Usage Notes

- Key behaviors
- Limitations
- Best practices
```
