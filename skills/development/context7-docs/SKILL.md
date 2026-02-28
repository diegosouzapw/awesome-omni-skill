---
name: context7-docs
description: "Fetch official library docs via Context7 MCP. Use for Tailwind CSS docs (grid, responsive variants), React, Next.js, Vue, MCP, OpenCode, or any npm library. Always use before external web search."
allowed-tools: Bash(npx:*) Bash(./scripts/*) Read Glob Grep
context: fork
---

# Context7 Docs - Library Documentation

Fetch up-to-date library documentation via the Context7 MCP server. Use this skill when you need to research any npm library, framework, or tool.

## Quick Reference

| Action | Command | Description |
|--------|---------|-------------|
| search | `./scripts/docs.sh search <library>` | Find library ID for a given name |
| docs | `./scripts/docs.sh docs <library> [topic] [--tokens N]` | Get documentation for a library |
| help | `./scripts/docs.sh help` | Show usage help |

## How to Use

### Natural Language
- "Get React hooks documentation"
- "Search for Next.js App Router docs"
- "Find Vue 3 composition API documentation"
- "Look up Tailwind CSS utilities"

### Script Commands
```bash
# Search for a library
./scripts/docs.sh search react
./scripts/docs.sh search "next.js"

# Get documentation (automatically resolves library ID)
./scripts/docs.sh docs react hooks
./scripts/docs.sh docs next.js "app router"
./scripts/docs.sh docs tailwindcss utilities

# Get general documentation without topic filter
./scripts/docs.sh docs vue
```

## Available Actions

### search
Find the Context7-compatible library ID for a given library name.

**Parameters:**
- `library` (required): Library name to search for (e.g., "react", "next.js")

**Example:**
```bash
./scripts/docs.sh search react
# Returns: /npm/react, /websites/react_dev, etc.
```

### docs
Get documentation for a library, optionally filtered by topic.

**Parameters:**
- `library` (required): Library name (will be resolved to ID automatically)
- `topic` (optional): Topic to filter documentation (e.g., "hooks", "routing")

**Example:**
```bash
# Get React hooks documentation
./scripts/docs.sh docs react hooks

# Get general Next.js documentation
./scripts/docs.sh docs next.js

# Get specific Vue documentation
./scripts/docs.sh docs vue "composition api"
```

## Workflow

This skill encodes the Context7 two-step workflow:

1. **Resolve Library ID**: Converts human-readable library name to Context7 ID
2. **Fetch Documentation**: Gets documentation using the resolved ID

This is handled automatically by the `docs` action.

## Supported Libraries

Context7 supports thousands of libraries including:
- **Frontend**: React, Vue, Angular, Svelte, Solid
- **Meta-frameworks**: Next.js, Nuxt, Remix, Astro, SvelteKit
- **Styling**: Tailwind CSS, styled-components, Emotion
- **State**: Redux, Zustand, Jotai, Pinia
- **Backend**: Express, Fastify, Hono, Nest.js
- **Databases**: Prisma, Drizzle, TypeORM, Mongoose
- **Testing**: Jest, Vitest, Playwright, Cypress
- **And many more...**

## Prerequisites

- Node.js 18+ installed
- curl installed
- **Option A:** Install MCPorter via Homebrew: `brew tap steipete/tap && brew install mcporter`
- **Option B:** Use via npx (no install required): `npx mcporter`

> **Note:** Context7 server configuration is optional. The skill automatically falls back to the Context7 public URL (`https://mcp.context7.com/mcp`) if no local server is configured.

## Environment Variables

- `CONTEXT7_SERVER`: MCP server name (default: `context7`)
- `CONTEXT7_API_BASE`: Direct Context7 REST base URL (default: `https://context7.com/api/v2`)
- `CONTEXT7_REST_FALLBACK`: Enable direct REST fallback when MCP calls fail (default: `1`)
- `MCPORTER_TIMEOUT`: Timeout in seconds for MCPorter calls (default: `20`)

## Notes

- Use this skill BEFORE searching external documentation
- Topic filtering helps reduce context size - use specific topics when possible
- Use `--tokens N` to control response size when context budget is tight
- If a library isn't found, try alternative names (e.g., "nextjs" vs "next.js")
- For generic MCP access, use the `mcporter` skill instead
- Falls back to Context7 public MCP URL when server not configured locally
- If MCP calls fail (including quota exhaustion), falls back to direct Context7 REST API when enabled
