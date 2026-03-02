---
name: optimise-cursor-repo
description: Audit a repository and produce prioritised recommendations for improving Cursor performance and developer experience. Use when the user wants to optimise their repo for Cursor, improve indexing, add rules, or assess their Cursor configuration.
---

# Cursor Repo Optimisation

Audit a repository's Cursor configuration, indexing setup, rules, hooks, documentation, and workflows. Produce a prioritised report of recommendations.

**Reference files in this skill:**

- `references/audit-commands.md` — Bash commands for each audit area
- `references/audit-process.md` — Parallel and sequential audit workflows
- `references/migration-examples.md` — Migration formats and example recommendations
- `scripts/validate-artefacts.sh` — Validate frontmatter for skills, rules, and subagents

## Important: Consult the Cursor Documentation

Cursor evolves rapidly. Always consult these sources for the latest information:

- <https://cursor.com/docs/context/rules> — Rules configuration
- <https://cursor.com/docs/context/skills> — Skills configuration
- <https://cursor.com/docs/context/subagents> — Subagents configuration
- <https://cursor.com/docs/agent/hooks> — Hooks system
- <https://cursor.com/docs/plugins> — Plugins and marketplace
- <https://cursor.com/changelog> — Changelog

## Scope

This skill is for **audit and recommendation only**. Do not make changes to the repository. Present findings as a prioritised report.

For AGENTS.md creation and updates, refer to the **deepinit** skill.

---

## Verify Before Recommending

**CRITICAL:** Before including any recommendation in the report, verify that it is actually needed by reading the relevant files.

**Common verification failures:**

- Recommending a capability that already exists in a different form (e.g., a command that orchestrates something you thought was missing)
- Suggesting migrations for artefacts that are already correctly configured
- Flagging files for indexing exclusion when they're already in `.cursorindexingignore`
- Proposing new workflows without checking if existing skills/commands already implement them

**For each potential recommendation:**

1. Read the relevant configuration files to confirm the gap exists
2. If recommending changes to commands/skills/rules, read those files first
3. If suggesting new capabilities, check whether they already exist in a different form
4. Cross-reference with existing `.cursor/` artefacts to avoid redundant suggestions

**Example:** Before recommending "add MCP server for Linear", check if Linear integration already exists via `.cursor/mcp.json` or an existing skill.

---

## Decision Tree: Where Should This Information Live?

```
Is this information needed on EVERY request?
├─ YES → Is it under ~5 lines?
│   ├─ YES → Root AGENTS.md
│   └─ NO → Always-apply rule (but question whether it truly needs to be always-on)
├─ NO → Is it scoped to specific files or directories?
│   ├─ YES → Glob-scoped rule (e.g., globs: api/**/*.py,api/**/*.md)
│   └─ NO → Does the agent need to decide when it's relevant?
│       ├─ YES → Skill (SKILL.md)
│       └─ NO → Is it triggered by an explicit user action?
│           ├─ YES → Skill with disable-model-invocation: true
│           └─ NO → Subagent (.cursor/agents/) if it needs isolated
│                   context, otherwise skill
```

### Artefact Type Reference

| Artefact                         | Nature                   | Loading                        | Best For                                                            |
| -------------------------------- | ------------------------ | ------------------------------ | ------------------------------------------------------------------- |
| AGENTS.md                        | Passive, always loaded   | Every request                  | Project identity, directory map, universal constraints (~100 lines) |
| Always-apply rule                | Passive, always loaded   | Every request                  | Universal coding style, language preferences (use sparingly)        |
| Glob-scoped rule                 | Passive, auto-attached   | When matching files in context | Area-specific conventions (API patterns, migrations, tests)         |
| Skill                            | Active, agent-discovered | When task matches description  | Procedural workflows, domain expertise, multi-step "how-to"         |
| Skill (disable-model-invocation) | Active, user-invoked     | Only when user types `/skill`  | Saved prompts, repeatable workflows user triggers explicitly        |
| Subagent                         | Active, delegated        | When parent agent delegates    | Tasks needing isolated context, parallel execution                  |

#### Legacy Artefact Types

| Artefact                    | Status          | Notes                                                                                                          |
| --------------------------- | --------------- | -------------------------------------------------------------------------------------------------------------- |
| Apply-intelligently rule    | Still supported | Consider migrating to skill if it contains multi-step procedures or would benefit from `references/` structure |
| Command (.cursor/commands/) | Still supported | Skills with `disable-model-invocation: true` are the newer alternative                                         |

**Note:** Cursor includes a `/migrate-to-skills` command that can convert apply-intelligently rules and commands to skills if desired.

**The acid test:** If it tells the agent _how to behave_, it's a rule. If it tells the agent _how to do something_, it's a skill. If it's a prompt you're tired of retyping, it's a skill with `disable-model-invocation: true`. If it needs a clean context window, it's a subagent.

---

## Audit Checklist

For bash commands to run for each area, see `references/audit-commands.md`.

### 1. Indexing Exclusions

Check `.cursorignore`, `.cursorindexingignore`, and `.gitignore`. The key distinction:

- `.cursorignore` — Files invisible to Cursor entirely
- `.cursorindexingignore` — Files excluded from index but manually includable via `@file`

**Flag for exclusion:** Large reference docs, scraped data, database snapshots, generated code, binary content, build artefacts, vendored dependencies.

### 2. Rules Configuration

| Mode                    | Frontmatter                                | Guidance                                                                  |
| ----------------------- | ------------------------------------------ | ------------------------------------------------------------------------- |
| Always Apply            | `alwaysApply: true`                        | Use sparingly — loads on every request                                    |
| Apply to Specific Files | `alwaysApply: false` + `globs: <patterns>` | Preferred for area-specific rules                                         |
| Apply Intelligently     | `description` only, no globs               | Agent decides based on description; consider skill for procedural content |
| Apply Manually          | No globs, no description                   | Only when `@`-mentioned                                                   |

**Anti-patterns:** God rules (>500 lines), procedural rules (should be skills), stale rules, conflicting rules, overly broad globs.

**Sizing:** Individual rules <500 lines, all always-apply rules combined <200 lines.

### 3. AGENTS.md Documentation

**Bloated signs:** >100 lines at root, detailed command references, area-specific conventions, procedural instructions.

**Too thin signs:** No directory map, no tech stack, no build/test commands, no critical constraints.

### 4. Context Weight Optimisation

| Context type                | Budget         |
| --------------------------- | -------------- |
| Root AGENTS.md              | ~100 lines     |
| All always-apply rules      | ~200 lines     |
| **Total always-on context** | **<300 lines** |
| Individual glob-scoped rule | ~200 lines max |
| Skill SKILL.md body         | ~500 lines max |

**Red flags:** Root AGENTS.md >200 lines, >3 always-apply rules, single rule >500 lines.

### 5. Hooks

Check `.cursor/hooks.json` for hook configuration. Format requires `version: 1` and hooks object:

```json
{
  "version": 1,
  "hooks": {
    "hookName": [{ "command": "./script.sh", "matcher": "pattern" }]
  }
}
```

High-value hook patterns:

- `sessionStart` — Inject branch name, ticket, environment context via `env` and `additional_context` output
- `preToolUse` with `matcher: "Shell"` — Validate or transform shell commands
- `beforeShellExecution` — Block dangerous commands (exit code 2 to deny)
- `afterFileEdit` — Run formatters/linters after agent edits
- `stop` — Prompt handoff notes, auto-retry logic via `followup_message`

Available hooks: `sessionStart`, `sessionEnd`, `preToolUse`, `postToolUse`, `postToolUseFailure`, `subagentStart`, `subagentStop`, `beforeShellExecution`, `afterShellExecution`, `beforeMCPExecution`, `afterMCPExecution`, `beforeReadFile`, `afterFileEdit`, `beforeSubmitPrompt`, `preCompact`, `stop`, `afterAgentResponse`, `afterAgentThought`, `beforeTabFileRead`, `afterTabFileEdit`

### 6. Generated and Binary Files

Flag: Auto-generated types (OpenAPI/protobuf), compiled output, SQL dumps, LFS-tracked paths, large test fixtures.

### 7. Codebase Indexing Settings

UI settings to verify (Cursor Settings > Features):

- **Codebase Indexing** enabled
- **Include Project Structure** enabled
- Indexing status shows fully indexed

### 8. Skills

**Skills are the preferred format for:**

- Multi-step procedural workflows
- Domain-specific expertise that benefits from `references/` subdirectories
- User-invoked prompts (with `disable-model-invocation: true`)

**Check:** Does `.cursor/skills/` exist? Do skills use progressive disclosure (lean SKILL.md, heavy docs in `references/`)? Would any apply-intelligently rules benefit from skill structure?

**Note:** Cursor includes a built-in `/migrate-to-skills` command that can convert apply-intelligently rules and commands to skills.

### 9. Artefact Validation

**IMPORTANT:** Read each artefact file to verify correct structure. Don't just check existence.

Run `scripts/validate-artefacts.sh` from the project root for automated validation of skills, rules, and subagent frontmatter.

#### Skill Frontmatter (Required)

```yaml
---
name: skill-name
description: What this skill does and when to use it
---
```

**Optional fields:**

- `disable-model-invocation: true` — For user-invoked-only skills (replacement for commands)

**Common errors:**

- Missing frontmatter entirely (no `---` block)
- Missing `name` field
- Missing `description` field (agent won't discover the skill)
- Empty or placeholder description

#### Rule Frontmatter

| Rule Type    | Required Frontmatter                           |
| ------------ | ---------------------------------------------- |
| Always-apply | `alwaysApply: true`                            |
| Glob-scoped  | `alwaysApply: false` + `globs: <patterns>`     |
| Manual-only  | No `alwaysApply`, no `globs`, no `description` |

**Common errors:**

- `globs:` present but empty
- Both `alwaysApply: true` AND `globs` (redundant — always-apply ignores globs)

**Note:** `alwaysApply: false` without `globs` but with a `description` is the valid "Apply Intelligently" pattern.

#### Subagent Frontmatter

All fields are optional but recommended for discoverability:

```yaml
---
name: subagent-name          # Defaults to filename without extension
description: What this subagent does and when to use it
model: inherit               # Options: fast, inherit, or specific model ID
readonly: true               # For read-only subagents
is_background: false         # Run asynchronously without blocking parent
---
```

**Best practices:**

- Include `description` for automatic delegation (agent reads this to decide when to use the subagent)
- Use `readonly: true` for review/audit subagents
- Use `is_background: true` for long-running tasks that don't need to block
- Include "use proactively" in description to encourage automatic delegation

### 10. Commands (Legacy)

Commands in `.cursor/commands/` still work but skills with `disable-model-invocation: true` are the newer alternative. Cursor's built-in `/migrate-to-skills` command can convert them automatically. See `references/migration-examples.md` for manual migration format.

### 11. Subagents

Check `.cursor/agents/` for focused, well-described subagents. The `model` field is optional (`inherit` is the default).

High-value patterns: debugger, security auditor, test writer, documentation specialist, code reviewer.

### 12. MCP Server Configuration

Check `.cursor/mcp.json` or `mcp.json`. Only suggest MCP servers for services the project actually uses (verify in package.json, requirements.txt, docker-compose).

### 13. Plugins and Marketplace

Plugins are installed from the [Cursor Marketplace](https://cursor.com/marketplace) and bundle rules, skills, agents, commands, MCP servers, and hooks. Check Cursor Settings > Rules for installed plugins.

For creating plugins: requires `.cursor-plugin/plugin.json` manifest. See <https://cursor.com/docs/plugins/building> for format.

### 14. Sandbox Configuration

Check `sandbox.json` for network and filesystem access controls.

---

## Audit Process

See `references/audit-process.md` for detailed parallel and sequential audit workflows.

**Quick summary:**

1. Read config files (.cursorignore, .cursorindexingignore, .gitignore, .cursorrules, hooks.json, mcp.json, sandbox.json, AGENTS.md)
2. List `.cursor/` directory contents
3. Identify legacy artefacts that may benefit from migration (apply-intelligently rules, commands)
4. Check for index pollution (large directories, generated code, binary files)
5. Measure context weight
6. Check Cursor documentation for new features
7. **Verify each recommendation** — read relevant files to confirm gaps exist before including in report

---

## Output Format

Present findings as a prioritised report:

```
## Cursor Optimisation Report

### P0 — Critical (high impact, low effort)
[Recommendations that significantly improve indexing, search quality, or context efficiency]

### P1 — Important (high impact, moderate effort)
[Recommendations that improve developer experience or reduce repetitive work]

### P2 — Recommended (moderate impact)
[Recommendations that would improve the setup but are not urgent]

### P3 — Nice to Have (lower impact or beta features)
[Recommendations for newer or experimental features]

### Already Well Configured
[Areas that are already set up correctly — acknowledge good practices]

### Settings to Verify Manually
[UI-only settings the user should check in Cursor Settings]
```

For each recommendation, include:

- **What** to change
- **Why** it matters
- **Specific file contents or changes** to make

See `references/migration-examples.md` for example recommendations.

---

## After the Report

Once the prioritised report is complete:

1. **List every actionable recommendation** as a numbered summary (one line each, referencing the priority level, e.g. `[P0] Exclude backend/vector_db/public_documents/ from indexing`)

2. **Ask the user** which items they would like implemented:

   > Which of these would you like me to implement? You can specify by number (e.g. "1, 3, 5"), a priority level (e.g. "all P0 and P1"), or say "all".

3. **Wait for the user's response**, then implement only the selected items.
