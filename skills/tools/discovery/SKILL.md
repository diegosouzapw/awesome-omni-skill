---
name: discovery
description: "Skill and MCP tool discovery guide — teaches agents how to find skills and tools."
category: core
alwaysApply: true
injectionFormat: content
metadata:
  gobby:
    audience: all
---

## Skills & Discovery

**Skills** -- context-aware prompts for common workflows.
- Browse: `list_skills()` on `gobby-skills`
- Load: `get_skill(name="skillname")` on `gobby-skills`
- Hubs: `search_hub(query)` on `gobby-skills`

**MCP Tools** -- progressive disclosure:
1. `list_mcp_servers()` -- discover servers
2. `list_tools(server)` -- discover tools
3. `get_tool_schema(server, tool)` -- get params
4. `call_tool(server, tool, arguments)` -- execute
