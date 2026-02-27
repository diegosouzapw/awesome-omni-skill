---
name: copilot-vscode
description: "VS Code Copilot platform reference: .agent.md all frontmatter fields, built-in tool names, agent types, context engineering, subagents, handoffs, MCP config, SKILL.md integration, custom instructions. Load when working with custom agents, tool lists, context strategy, or any VS Code Copilot platform feature."
---

# SKILL: VS Code Copilot Platform

> Load when the task concerns custom agent files (`.agent.md`), tool selection, context strategy,
> subagents, handoffs, MCP servers in VS Code, SKILL.md configuration, or any VS Code Copilot feature.
>
> Official documentation: https://code.visualstudio.com/docs/copilot
> GitHub source: https://github.com/microsoft/vscode-docs/tree/main/docs/copilot
> Use `#fetch <url>` in chat to read any page listed in the index at the bottom of this skill.

## When to Load This Skill

- Creating or reviewing a `.agent.md` file
- Choosing which tools to include (least-privilege applies — see ai-security SKILL)
- Designing a subagent workflow or handoff chain
- Configuring MCP servers in `.vscode/mcp.json`
- Setting up custom instructions or `copilot-instructions.md`
- Context engineering: what VS Code puts in the context window, and how to control it
- Any question about VS Code Copilot agent types, modes, or settings

---

## `.agent.md` Frontmatter — All Valid Fields

```yaml
---
description: <string>          # shown as placeholder in Chat input; one-sentence role summary
name: <string>                 # display name; if omitted, file name is used
tools: [<tool-name>, ...]      # allowed tools — see Built-in Tool Names below
agents: [<agent-name>, ...]    # allowed subagents; '*' = all, [] = none
model: <string | string[]>     # model name or priority list; if omitted = user's current picker
user-invokable: <bool>         # show in agents dropdown (default: true)
disable-model-invocation: <bool>  # prevent being called as subagent (default: false)
argument-hint: <string>        # hint text in chat input field
handoffs:                      # guided transitions to other agents
  - label: <string>            # button text
    agent: <agent-name>        # target agent id
    prompt: <string>           # pre-filled prompt for target
    send: <bool>               # auto-submit prompt (default: false)
    model: <string>            # model for the handoff (format: 'Model Name (vendor)')
target: vscode | github-copilot  # deployment target (default: vscode)
---
```

**Key rules:**
- `name` must match the file name for reliable detection by VS Code
- VS Code detects any `.agent.md` in `.github/agents/` automatically (no registration needed)
- `user-invokable: false` hides from dropdown but still allows subagent invocation
- `disable-model-invocation: true` blocks subagent invocation but keeps it in dropdown
- These two fields replace the deprecated `infer:` field

---

## Built-in Tool Names for `tools:` List

### File & code tools
| Tool name | What it does |
|---|---|
| `readFile` | Read file contents |
| `editFiles` | Edit existing files |
| `createFiles` | Create new files |
| `fileSearch` | Search files by name/glob |
| `textSearch` | Full-text search across workspace |
| `codebase` | Semantic workspace search (`#codebase`) |
| `usages` | Find symbol usages and references |
| `search` | Combined search tool |

### Execution tools
| Tool name | What it does |
|---|---|
| `runTerminal` | Run terminal commands |
| `problems` | Access Problems panel (linting, compiler errors) |
| `changes` | Access current source control changes |

### Network tools
| Tool name | What it does |
|---|---|
| `fetch` | Fetch URL content |
| `webSearch` | Web search |
| `githubRepo` | Search a GitHub repository |

### Agent tools
| Tool name | What it does |
|---|---|
| `agent` | Required when using `agents:` field (spawn subagents) |

**Tool sets** (reference a group as one name): `#edit`, `#search` (predefined); custom tool sets defined in `toolsets.jsonc`.

**MCP tools**: reference as `<server-name>/<tool-name>` or `<server-name>/*` for all tools of a server.

> Least-privilege rule (A1.6): give each agent only the tools required for its specific role.
> Critics: read-only only (`readFile`, `fileSearch`, `textSearch`).

---

## Tool Approval & Auto-Approve (`chat.tools.autoApprove`)

VS Code Copilot can require user confirmation for tool usage. The setting below can allow specific tools
to run without prompting.

**Architect guidance:**
- Prefer **no auto-approve** by default.
- If you enable auto-approve, scope it to the **lowest-risk, read-only** tools.
- Never auto-approve anything that can make **irreversible changes** (deleting resources, sending messages,
  writing to remote systems).

**Threat model reminder (A1.6):** tool auto-approval increases the blast radius of prompt injection and
mis-scoped agent permissions.

---

## Tool Sets (`toolsets.jsonc`)

Tool sets let you reference a **named bundle of tools** instead of listing each tool repeatedly.
This helps keep `.agent.md` files short and makes least-privilege reviews easier.

**Architect guidance:**
- Use tool sets to encode role capabilities (e.g., `docs-readonly`, `repo-editor`) and keep them stable.
- Review tool sets like an API surface: changes can silently widen agent permissions.
- Prefer multiple small tool sets over one broad “kitchen sink” set.

> Note: the exact `toolsets.jsonc` format and lookup rules are VS Code-specific. Use `#fetch` on the
> official docs index at the bottom of this skill when you need the precise schema.

---

## Agent Types (VS Code)

| Type | Where it runs | When to use |
|---|---|---|
| **Local — Agent** | VS Code, interactive | Complex multi-file tasks, iterative work, needs editor/terminal context |
| **Local — Plan** | VS Code, interactive | Breaking down complex tasks before implementation; generates structured plan |
| **Local — Ask** | VS Code, interactive | Questions about codebase; read-only research |
| **Background agent** | VS Code, background | Well-defined tasks while you keep working |
| **Cloud agent** | GitHub, remote | Automated PRs, tasks requiring team collaboration, CI-style jobs |
| **Third-party agent** | External provider / extension / service | When a non-Microsoft agent workflow is required (policy-approved), e.g., vendor-specific capabilities or integrations not available via built-in agents |
| **Custom agent** | Local or cloud | Specialized persona: security reviewer, planner, architect, etc. |

**Selection guide (architect-facing):**

**Pick the execution *surface* first:**
- Needs continuous back-and-forth, live editor state, stack traces, linting → **Local — Agent**
- Needs a plan artifact first, before any changes → **Local — Plan**
- Needs read-only investigation / explanation → **Local — Ask**
- Needs autonomy while the user keeps coding in VS Code → **Background agent**
- Needs a PR-oriented workflow (review + CI), or work should continue without the user's machine → **Cloud agent**

**Then pick the *persona*:**
- Needs a specialized, stable role with least-privilege tools (critic, architect, compliance reviewer) → **Custom agent** (`.agent.md`)

**Use third-party agents only when necessary:**
- If the requirement is “a tool/integration”, prefer adding an **MCP tool** over introducing a new agent provider.
- If a third-party agent is required, confirm it is **policy-approved** and treat it as a **trust-boundary crossing** (A1.6): minimize shared context, avoid secrets, and prefer read-only tasks.

**Quick decision checklist (copy into your recommendation):**
- **Output**: chat answer vs code changes vs PR
- **Context need**: current editor state vs repo-only context
- **Runtime need**: terminal/linters/tests required?
- **Collaboration**: needs CI + review trail?
- **Risk**: any irreversible actions? → require explicit confirmation
- **Trust boundary**: any external/third-party processing? → minimize context and verify approvals

---

## Subagents

**What they are:** child agents with isolated context windows. Main agent waits for result, receives only the summary — not the full subagent conversation.

**When to use:**
- Offload research so main agent context stays focused
- Run multiple analyses in parallel (VS Code runs subagents concurrently)
- Isolate experimental work (dead ends don't pollute main context)
- Apply specialized behavior (use a security-reviewer custom agent as subagent)

**How to invoke in a prompt:**
```
Use a subagent to research OAuth 2.0 patterns for Node.js. Return a recommendation.
```

**In `.agent.md`:**
```yaml
tools: ['agent', 'readFile', 'fileSearch']
agents: ['forgent-docs-critic', 'forgent-process-critic']  # or '*' for all
```

**Key properties:**
- Synchronous: main agent blocks until subagent returns
- Parallel: multiple subagents can run concurrently
- Subagents start with a clean context window — they do NOT inherit main agent instructions
- `user-invokable: false` + subagent-only agents: used for internal pipeline steps

> Ref: https://code.visualstudio.com/docs/copilot/agents/subagents

---

## Handoffs

Transition from one agent to another with context and a pre-filled prompt. Handoff buttons appear after a response completes.

```yaml
handoffs:
  - label: "Start Implementation"
    agent: forgent-spec-editor
    prompt: "Now implement the plan outlined above."
    send: false           # user reviews before submitting
  - label: "Review Changes"
    agent: forgent-process-critic
    prompt: "Review the changes made in the previous step."
    send: true            # auto-submits
    model: "Claude Sonnet 4.5 (copilot)"
```

**Use for:** plan → implement → review pipelines; multi-step workflows where human reviews each step.

---

## Context Engineering in VS Code

### How VS Code assembles context

1. **System prompt** — agent body from `.agent.md`
2. **Custom instructions** — from `copilot-instructions.md` + any `*.instructions.md`
3. **Conversation history** — compressed when context window fills
4. **Explicit `#`-mentions** — `#file`, `#codebase`, `#web`, `#changes`, etc.
5. **Automatic context** — current editor, workspace index, git state
6. **Tool outputs** — added per iteration in the agent loop

### Context sources reference

| `#`-mention | What it adds |
|---|---|
| `#file:<path>` | Specific file contents |
| `#codebase` | Semantic workspace search result |
| `#changes` | Current source control diff |
| `#problems` | Problems panel contents |
| `#web` | Web search result |
| `#fetch <url>` | URL content |
| `#githubRepo <owner/repo>` | GitHub repository search |

### Workspace index

- Remote index: built from committed state on GitHub/Azure DevOps — best for large codebases
- Local index: uncommitted changes, hybrid with remote
- Build index: **Build Remote Workspace Index** command in Command Palette
- `.gitignore` files are excluded from index (unless opened)

> Ref: https://code.visualstudio.com/docs/copilot/reference/workspace-context

---

## Custom Instructions vs SKILL.md

| | Custom instructions (`copilot-instructions.md`) | SKILL.md |
|---|---|---|
| **Purpose** | Coding standards, guidelines, always-on rules | Specialized capabilities, lazy-loaded knowledge |
| **When loaded** | Always (every chat interaction) | On-demand when relevant |
| **Content** | Instructions only | Instructions + scripts + examples + resources |
| **Standard** | VS Code-specific | Open standard (agentskills.io), portable |
| **Location** | `.github/copilot-instructions.md` | `.github/skills/`, `.agents/skills/`, `~/.copilot/skills/` |

**Rule:** keep `copilot-instructions.md` short (~300 words). Move domain-specific knowledge to SKILL.md.

### SKILL.md detection locations

VS Code searches for skills in:
- `.github/skills/<name>/SKILL.md`
- `.agents/skills/<name>/SKILL.md` ← this repo's convention
- `.claude/skills/<name>/SKILL.md`
- User profile: `~/.copilot/skills/<name>/SKILL.md`

To add more locations: `setting(chat.agentSkillsLocations)` in VS Code settings.

---

## MCP Server Configuration (`.vscode/mcp.json`)

```json
{
  "servers": {
    "<server-name>": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@scope/mcp-server"],
      "env": { "API_KEY": "${input:api-key}" }
    },
    "<remote-server>": {
      "type": "http",
      "url": "https://api.example.com/mcp",
      "headers": { "Authorization": "Bearer ${input:token}" }
    }
  },
  "inputs": [
    { "id": "api-key", "type": "promptString", "description": "API Key", "password": true }
  ]
}
```

**Key rules:**
- `type: stdio` — server runs in foreground, communicates via stdin/stdout (no `-d` for Docker)
- `type: http` — VS Code tries HTTP Stream transport first, falls back to SSE
- Secrets via `${input:...}` — never hardcode credentials
- Tools available as `<server-name>/<tool-name>` in `tools:` lists
- Supply chain risk: every MCP server is a new attack surface (OWASP LLM06)

> Ref: https://code.visualstudio.com/docs/copilot/reference/mcp-configuration

---

## `.vscode/settings.json` — Agent Registration

```jsonc
{
  "chat.agentFilesLocations": {
    ".github/agents": true        // scan this folder for .agent.md files
  },
  "chat.agentSkillsLocations": [
    ".agents/skills"              // additional SKILL.md search path
  ],
  "chat.agent.enabled": true,     // must be true for agent mode
  "chat.tools.autoApprove": {}    // per-tool auto-approval config
}
```

**Architect guidance (minimal safe defaults):**
- Keep `chat.tools.autoApprove` empty unless you have a clear reason and a documented safety boundary.
- If you must add entries, prefer read-only tools first (search/read) and keep the list explicit.

---

## Coverage Checklist — Upstream `customization/` Docs

This skill now includes **explicit coverage** for every page currently in:
`https://github.com/microsoft/vscode-docs/tree/main/docs/copilot/customization`

**Upstream pages (folder contents):**
- `overview.md`
- `custom-instructions.md`
- `prompt-files.md`
- `agent-skills.md`
- `custom-agents.md`
- `hooks.md`
- `language-models.md`
- `mcp-servers.md`

> Design intent for multi-agent systems (A1.2): treat **custom instructions** as baseline constraints,
> **prompt files** as repeatable task macros, **custom agents** as tool-scoped personas,
> **skills** as portable capability packages, **MCP** as external tool surface, **hooks** as deterministic guardrails.

---

## Customization Overview — Choosing the Right Mechanism

Use this as a decision table when designing your agent system.

| Goal | Prefer | Why |
|---|---|---|
| Enforce global repo conventions | Custom instructions | Always included; low operational overhead |
| Different rules per language/folder | File-based instructions | Scoped by `applyTo` patterns |
| Repeatable single task (“do X”) | Prompt file (`.prompt.md`) | Explicit invocation via `/...` |
| Multi-step capability + assets | Agent Skill (SKILL.md + resources) | Progressive disclosure + portability |
| Specialist persona + tool boundary | Custom agent (`.agent.md`) | Least-privilege tools per role |
| External API / DB / browser / SaaS | MCP server | Adds tool capabilities beyond VS Code built-ins |
| Deterministic enforcement / automation | Hooks | Executes code at lifecycle events; can block tools |

**Prompt files vs custom agents:**
- Prompt file = “run this procedure” (task).
- Custom agent = “be this role” (persona + tool boundary + optional handoffs/subagents).

### Project bootstrap & diagnostics
- `/init` can generate a starter `.github/copilot-instructions.md` tailored to the workspace; treat it as a draft to review.
- If customizations don’t apply as expected: open Chat **Configure Chat (gear)** → **Diagnostics** and verify which agents/prompt files/instructions/skills are loaded and whether they have validation errors.

---

## Custom Instructions — Files, Scope, and Precedence

### Instruction types
- **Always-on** (applies to every request)
- **File-based** (applies when files match patterns or description matches task)

### Common file locations (workspace)
- Always-on: `.github/copilot-instructions.md`
- File-based: `.github/instructions/**/*.instructions.md` (default)

### Additional supported sources (use intentionally)
- `AGENTS.md` (always-on instructions file supported by VS Code)
  - In this repo, `AGENTS.md` is also the orchestrator repo map.
  - If you also use it as Copilot instructions: keep it dual-purpose and avoid contradictions.
- `CLAUDE.md` (always-on for Claude-compat tooling)

### `.instructions.md` frontmatter (key fields)
- `applyTo`: glob pattern relative to workspace root (use `**` to apply to all)
- `name`, `description`: UI metadata; keep descriptions explicit so semantic matching works

### Settings you may need (diagnostics-oriented)
- `chat.instructionsFilesLocations` — add extra folders for instructions discovery
- `chat.useAgentsMdFile` — enable/disable `AGENTS.md` detection
- `chat.useNestedAgentsMdFiles` — nested `AGENTS.md` in subfolders (experimental)
- `chat.includeApplyingInstructions` — include pattern-matched instructions automatically
- `chat.includeReferencedInstructions` — include instructions referenced via Markdown links

### Priority model (conflict resolution)
When multiple sources exist, higher priority generally wins:
1. Personal/user instructions
2. Workspace/repository instructions
3. Organization-level instructions

### Portability & team rollout
- Organization-level instructions exist (GitHub org feature); use them for baseline policies across repos.
- Settings Sync can sync user prompt/instruction files across devices (look for the “Prompts and Instructions” sync category).

**Architect checklist (keep it effective and safe):**
- Keep always-on instructions short and stable; move deep procedures into skills.
- Explain the “why” behind non-obvious rules; include 1–2 examples for edge cases.
- Never embed secrets; reference env vars or input prompts only.

---

## Prompt Files (`.prompt.md`) — Slash Commands, Tools, Variables

### Locations
- Workspace: `.github/prompts/**/*.prompt.md`
- User profile: profile `prompts/` folder
- Configure additional locations with `chat.promptFilesLocations`

### Frontmatter fields (high-signal)
- `name`, `description`, `argument-hint`
- `agent`: `ask` | `agent` | `plan` | `<custom-agent-name>`
- `model`: optional; falls back to picker
- `tools`: optional; can include MCP tools and `<server>/*`

### Variable interpolation (design for reuse)
- Workspace: `${workspaceFolder}`, `${workspaceFolderBasename}`
- File context: `${file}`, `${fileBasename}`, `${fileDirname}`, `${fileBasenameNoExtension}`
- Selection: `${selection}`, `${selectedText}`
- Input: `${input:var}` / `${input:var:placeholder}`

### Tool list priority
If both prompt + agent specify tools, effective tools follow:
1. Prompt file tools
2. Tools from referenced custom agent
3. Default tools of selected agent

**Architect checklist:**
- Prefer prompt files for repeatable “one-shot” workflows.
- Use prompt files to *narrow* tools for a specific workflow, not broaden them.
- Reference instructions/skills via Markdown links instead of duplicating rules.

---

## Agent Skills — Progressive Disclosure and Portability

### What a skill is
- A folder containing `SKILL.md` (required) and optional resources/scripts.
- Skills are **auto-loaded on relevance** or **invoked via `/`**.

### Skill discovery and loading (3 levels)
1. **Discovery**: only `name` + `description` metadata is considered
2. **Instructions load**: body of `SKILL.md` is loaded when relevant
3. **Resource access**: other files in the skill folder are accessed on demand

### Skill frontmatter knobs
- `user-invokable: false` → hidden from slash menu, still auto-loadable
- `disable-model-invocation: true` → manual-only (no auto-load)

### Locations
- Project skills: `.github/skills/`, `.agents/skills/`, `.claude/skills/`
- Personal skills: `~/.copilot/skills/`, `~/.agents/skills/`, `~/.claude/skills/`
- Extend discovery with `chat.agentSkillsLocations`

**Architect checklist:**
- Put “how-to” + examples + scripts in skills; keep global instructions minimal.
- Write skill `description` like a classifier: capability + trigger phrases.
- Treat skills as a reusable library; version and review them like code.

---

## Custom Agents (`.agent.md`) — Personas, Tools, Subagents, Handoffs

This skill already contains the complete `.agent.md` frontmatter field list,
agent type guidance, subagent and handoff patterns.

**Add these customization-specific reminders:**
- Custom agents were formerly “custom chat modes”; `.chatmode.md` → `.agent.md`.
- If you declare `agents:` (subagents), ensure `tools:` includes `agent`.
- Handoffs are best for plan → implement → review workflows with human gates.

---

## Hooks — Deterministic Policy and Automation (Preview)

Hooks execute shell commands at lifecycle points and can block/ask/allow tool usage.

### Hook events (current set)
`SessionStart`, `UserPromptSubmit`, `PreToolUse`, `PostToolUse`, `PreCompact`,
`SubagentStart`, `SubagentStop`, `Stop`

### Hook config locations
- Workspace shared: `.github/hooks/*.json`
- Claude-compatible: `.claude/settings.json`, `.claude/settings.local.json`
- User: `~/.claude/settings.json`

### Security posture (A1.3)
- Hooks run with your VS Code permissions; treat them as executable code.
- Do not let agents freely edit hook scripts without manual approval.
- Validate/sanitize hook stdin JSON to avoid injection.

### Troubleshooting essentials
- Use Chat Diagnostics to see loaded hooks and validation errors.
- Check the Output panel channel for Copilot Chat Hooks to inspect stdout/stderr.
- If using `Stop`/`SubagentStop` “block” behavior, guard against infinite loops by respecting `stop_hook_active`.

---

## Language Models — Constraints That Affect Agent Design

Agent mode may restrict model availability to **tool-capable** models.

**Design implications:**
- Prefer explicit `model:` in a custom agent when output consistency matters.
- If relying on BYOK models, verify tool-calling capability and understand that
  some Copilot service steps may still occur (embeddings/indexing/intent routing).
- Treat “Auto model selection” as a performance feature, not a governance control.

**Also relevant knobs:**
- Inline chat can have a different default model via `inlineChat.defaultModel`.
- Inline suggestions use a separate completions model selection path; don’t assume chat model changes affect suggestions.

---

## MCP Servers — External Tool Surface and Trust

This skill already includes a baseline `.vscode/mcp.json` schema example.

**Add these operational/security essentials (A1.3 / LLM06):**
- Installing local MCP servers can run arbitrary code; review publisher and config.
- Prefer workspace `mcp.json` for team-shared config; user profile for personal tools.
- Use input variables / env vars; never hardcode secrets.
- Beyond tools, MCP can provide: **resources**, **prompts**, and **MCP apps**.
- Understand trust prompts: starting from config may bypass interactive trust UX.


## Official Documentation Index

Use `#fetch <url>` to read any page for deep dives:

| Topic | URL |
|---|---|
| Core concepts (agent loop, context, models) | https://code.visualstudio.com/docs/copilot/core-concepts |
| All agent types overview | https://code.visualstudio.com/docs/copilot/agents/overview |
| Local agents (Agent/Plan/Ask) | https://code.visualstudio.com/docs/copilot/agents/local-agents |
| Subagents | https://code.visualstudio.com/docs/copilot/agents/subagents |
| Agent tools reference | https://code.visualstudio.com/docs/copilot/agents/agent-tools |
| Customization overview | https://code.visualstudio.com/docs/copilot/customization/overview |
| Custom agents (`.agent.md`) | https://code.visualstudio.com/docs/copilot/customization/custom-agents |
| Agent Skills (SKILL.md) | https://code.visualstudio.com/docs/copilot/customization/agent-skills |
| Custom instructions | https://code.visualstudio.com/docs/copilot/customization/custom-instructions |
| Prompt files (`.prompt.md`) | https://code.visualstudio.com/docs/copilot/customization/prompt-files |
| Hooks | https://code.visualstudio.com/docs/copilot/customization/hooks |
| Language models | https://code.visualstudio.com/docs/copilot/customization/language-models |
| MCP servers | https://code.visualstudio.com/docs/copilot/customization/mcp-servers |
| MCP config reference | https://code.visualstudio.com/docs/copilot/reference/mcp-configuration |
| Workspace context / `#codebase` | https://code.visualstudio.com/docs/copilot/reference/workspace-context |
| Context engineering guide | https://code.visualstudio.com/docs/copilot/guides/context-engineering-guide |
| Cheat sheet | https://code.visualstudio.com/docs/copilot/reference/copilot-vscode-features |
