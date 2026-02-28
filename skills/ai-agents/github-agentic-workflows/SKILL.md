---
name: github-agentic-workflows
description: GitHub Agentic Workflows with MCP tools, Copilot coding agent orchestration, safe outputs, and OWASP Agentic security
license: Apache-2.0
---

# GitHub Agentic Workflows Skill

## Purpose

Comprehensive guidance for creating, deploying, and securing GitHub Agentic Workflows — AI-powered automations using Copilot coding agent, MCP servers, and custom agents for autonomous task execution.

## When to Use

- ✅ Orchestrating Copilot coding agent assignments
- ✅ Building multi-step agentic workflows with stacked PRs
- ✅ Configuring MCP servers for agent tools
- ✅ Implementing safe output handling for AI-generated content
- ✅ Securing agentic pipelines against prompt injection

## Core Concepts

### Copilot Coding Agent Assignment

```javascript
// Basic assignment
assign_copilot_to_issue({ owner: "Hack23", repo: "European-Parliament-MCP-Server", issue_number: 100 })

// Advanced with base_ref and custom instructions
assign_copilot_to_issue({
  owner: "Hack23", repo: "European-Parliament-MCP-Server",
  issue_number: 100,
  base_ref: "feature/new-tools",
  custom_instructions: "Use TypeScript strict mode. Follow MCP protocol. Add Vitest tests with 80%+ coverage."
})

// Direct PR creation with custom agent
create_pull_request_with_copilot({
  owner: "Hack23", repo: "European-Parliament-MCP-Server",
  title: "Add new MCP tool", body: "Implementation details",
  base_ref: "main", custom_agent: "frontend-specialist"
})

// Track progress
get_copilot_job_status({ owner: "Hack23", repo: "European-Parliament-MCP-Server", job_id: "abc123" })
```

### Stacked PR Workflow

```
Step 1: Data models (PR → main)
  └── Step 2: API client (PR → Step 1 branch)
       └── Step 3: MCP tools (PR → Step 2 branch)
```

### MCP Server Configuration

MCP server configuration is defined in `.github/copilot-mcp.json`. Secret references (`${{ secrets.* }}`) are resolved by the GitHub Copilot runtime — they are **not** literal JSON values. The MCP client receives actual token values at startup.

**Supply chain note:** The `npx -y @modelcontextprotocol/server-github` pattern downloads the latest version on each invocation. For production environments, pin to a specific version (e.g., `@modelcontextprotocol/server-github@0.x.y`) or vendor the package locally to prevent supply chain attacks when injecting privileged tokens.

See `.github/copilot-mcp.json` in this repository for the canonical configuration.

## OWASP Agentic Security

### Threat Mitigation

| Threat | Mitigation |
|--------|-----------|
| Prompt injection | Input validation, output sanitization |
| Excessive agency | Minimal tool permissions, scope limits |
| Data exfiltration | Sandbox environments, network controls |
| Supply chain | Pinned action versions, dependency scanning |

### Safe Output Handling

- Validate AI-generated code before merging
- Use CodeQL scanning on Copilot PRs
- Require human review for security-critical changes
- Sanitize all outputs before use in downstream tools

## ISMS Policy References

- [Secure Development Policy](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Secure_Development_Policy.md)
- [AI Policy](https://github.com/Hack23/ISMS-PUBLIC/blob/main/AI_Policy.md)
- [Information Security Policy](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Information_Security_Policy.md)
