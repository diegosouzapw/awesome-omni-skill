---
name: developing-claude-code-plugins
description: "Build, test, and distribute Claude Code plugins with slash commands, agents, skills, hooks, MCP servers, and LSP servers. MUST be loaded when creating, reviewing, debugging, or distributing plugins. Use PROACTIVELY when user mentions plugins, extensions, marketplaces, or wants to add custom commands/agents to Claude Code."
---

<essential_principles>
Plugins extend Claude Code with reusable functionality: custom slash commands, agents, skills, hooks, MCP servers, and LSP servers. Unlike standalone `.claude/` configurations, plugins are namespaced, versioned, and distributable through marketplaces.

**1. Plugin Structure Is Sacred**

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # ONLY manifest here (required)
├── commands/                 # Slash commands (plugin root)
├── agents/                   # Subagent definitions (plugin root)
├── skills/                   # Agent Skills (plugin root)
├── hooks/
│   └── hooks.json           # Event handlers (plugin root)
├── .mcp.json                # MCP server configs (plugin root)
└── .lsp.json                # LSP server configs (plugin root)
```

**CRITICAL**: Never put commands/, agents/, skills/, or hooks/ inside `.claude-plugin/`. Only `plugin.json` goes there.

**2. Namespacing Prevents Conflicts**

Plugin commands use format `/plugin-name:command-name`. The `name` field in plugin.json becomes the namespace prefix. Choose names carefully - they're public-facing.

**3. Use `${CLAUDE_PLUGIN_ROOT}` for Paths**

All paths in hooks, MCP servers, and scripts must use `${CLAUDE_PLUGIN_ROOT}` to reference plugin files. Plugins are copied to a cache directory, so relative paths like `../` won't work.

**4. Test with `--plugin-dir` During Development**

```bash
claude --plugin-dir ./my-plugin
```

This loads your plugin without installation. Restart Claude Code after changes.

</essential_principles>

<intake>
**What would you like to do?**

1. Create a new plugin
2. Add a component (command, agent, skill, hook, MCP, LSP)
3. Test or debug a plugin
4. Create a marketplace for distribution
5. Something else

**Wait for response before proceeding.**
</intake>

<routing>
| Response | Workflow |
|----------|----------|
| 1, "new", "create", "build", "start", "plugin" | `workflows/create-plugin.md` |
| 2, "add", "component", "command", "agent", "skill", "hook", "mcp", "lsp" | `workflows/add-component.md` |
| 3, "test", "debug", "fix", "error", "not working", "broken" | `workflows/test-debug-plugin.md` |
| 4, "marketplace", "distribute", "share", "publish" | `workflows/create-marketplace.md` |
| 5, other | Clarify intent, then route to appropriate workflow |

**After reading the workflow, follow it exactly.**
</routing>

<verification_loop>
**After Every Change:**

1. Validate plugin structure:
```bash
claude plugin validate ./my-plugin
```

2. Test plugin loads:
```bash
claude --plugin-dir ./my-plugin
```

3. Verify components appear:
- Commands: `/help` shows plugin commands
- Agents: `/agents` lists plugin agents
- MCP: `/mcp` shows server status

Report to user:
- "Plugin validates: ✓"
- "Commands registered: X"
- "Ready for testing"
</verification_loop>

<reference_index>
All in `references/`:

**Structure:** plugin-structure.md
**Components:** commands.md, agents.md, skills.md, hooks.md, mcp-lsp.md
**Distribution:** marketplace.md
**Issues:** troubleshooting.md
</reference_index>

<workflows_index>
All in `workflows/`:

| Workflow | Purpose |
|----------|---------|
| create-plugin.md | Create new plugin from scratch |
| add-component.md | Add command, agent, skill, hook, MCP, or LSP |
| test-debug-plugin.md | Test locally and fix issues |
| create-marketplace.md | Create and host plugin marketplace |
</workflows_index>

<success_criteria>
A well-built plugin:
- Has valid plugin.json manifest in `.claude-plugin/`
- Components are in correct directories (commands/, agents/, skills/, hooks/)
- All paths use `${CLAUDE_PLUGIN_ROOT}` variable
- Passes `claude plugin validate`
- Loads correctly with `--plugin-dir`
- Commands appear in `/help` output
</success_criteria>
