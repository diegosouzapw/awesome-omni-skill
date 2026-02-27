---
name: MCP Integration
description: Set up and use MCP (Model Context Protocol) servers to extend Claude Code capabilities including PRISM semantic memory, filesystem access, and database integration. Use when setting up MCP servers, debugging MCP connections, or understanding MCP tool usage.
allowed-tools:
  - Read
  - Bash
---

# MCP Integration Skill

## What is MCP?

**Model Context Protocol (MCP)** is a standardized protocol that extends Claude Code with additional capabilities through specialized servers. Each MCP server provides tools that Claude can invoke to interact with external systems, databases, APIs, and services.

**Benefits:**
- Extends Claude beyond built-in tools (Read, Write, Edit, Grep, Bash)
- Standardized interface for diverse integrations
- Lightweight and composable (mix and match servers)
- Context-efficient (tools described once, invoked many times)

**Trade-offs:**
- Each server consumes context tokens (typically 1.5-3k per server)
- More tools = more context overhead
- Some operations possible via Bash but MCP provides structure

---

## Configuration Architecture

### Three Configuration Locations

| Config File | Purpose | Key Name | Use Case |
|-------------|---------|----------|----------|
| `~/.claude/mcp_config.json` | Custom-built MCPs | `servers` | Your own MCP servers (PRISM, Orchestration) |
| `~/.config/claude/mcp_config.json` | External MCPs | `mcpServers` | Third-party NPM/Docker MCPs (GitHub, databases) |
| `<project>/.mcp.json` | Per-project MCPs | `mcpServers` | Project-specific tools (MongoDB, Redis for this project) |

**Key Insight:** Two different key names exist (`servers` vs `mcpServers`) for historical reasons. Custom MCPs use `servers`, everything else uses `mcpServers`.

### Configuration Scopes

Claude Code has three scope levels (use **local scope** for project-specific servers):

| Scope | Command | Storage | Visibility | Best For |
|-------|---------|---------|-----------|----------|
| **local** (default) | `claude mcp add` | `~/.claude.json` | All subdirs in git repo | Project-specific servers (MongoDB for this repo) |
| project | `claude mcp add -s project` | `.mcp.json` | Requires approval settings | Team-shared, checked into git |
| user | `claude mcp add -s user` | `~/.claude.json` | All projects everywhere | Global tools (GitHub, Package Registry) |

**Recommendation:** Use **local scope** for project databases, **user scope** for global tools like GitHub.

---

## Available MCP Servers (Quick Reference)

### Productivity & Documentation

**ObsidianPilot MCP** - Direct filesystem access to Obsidian vaults
- Full-text search (SQLite FTS5), frontmatter properties, tags, regex
- 100-1000x faster than REST API
- **Setup:** `uvx obsidianpilot` with `OBSIDIAN_VAULT_PATH` env var
- **Tools:** 15+ tools, ~3k tokens

**Jira MCP** - Full Jira integration with JQL search
- Search issues, get dev info (commits, PRs)
- **Setup:** `npm install -g @aashari/mcp-server-atlassian-jira`
- **Tools:** 4 tools, ~2.5k tokens

### Databases (Configure Per-Project)

**MongoDB MCP** - Direct MongoDB access
- Find, aggregate, CRUD operations, schema inference
- **Setup:** `claude mcp add mongodb npx -y mongodb-mcp-server -e MDB_MCP_CONNECTION_STRING=...`
- **Tools:** 9 tools, ~3k tokens

**PostgreSQL MCP** - Query PostgreSQL databases
- SQL queries, schema inspection, read-only by default
- **Setup:** `docker pull mcp/postgres` + `.mcp.json` config
- **Tools:** 3 tools, ~2.5k tokens

**SQLite MCP** - Lightweight database operations
- Read/write queries, table management
- **Setup:** `npm install -g @modelcontextprotocol/server-sqlite`
- **Tools:** 6 tools, ~2.5k tokens

**Redis MCP** - Key operations, cache inspection, pub/sub
- Get/set/delete, TTL management
- **Setup:** `docker pull mcp/redis` + `.mcp.json` config
- **Tools:** 8 tools, ~2k tokens

**Neo4j MCP** - Graph database queries
- Cypher queries, graph traversal
- **Setup:** `npm install -g @modelcontextprotocol/server-neo4j`
- **Tools:** 4 tools, ~3k tokens
- **Note:** You already use Neo4j for PRISM (port 7688)

**For detailed setup instructions, see `reference.md`.**

---

## Quick Setup Guide

### Installing Global External MCPs

**Via npm:**
```bash
npm install -g @modelcontextprotocol/server-github
```

**Add to user scope:**
```bash
claude mcp add -s user github mcp-server-github \
  -e GITHUB_TOKEN=ghp_xxxxxxxxxxxxx
```

**Verify:**
```bash
claude mcp list
# Should show: github: mcp-server-github - ✓ Connected
```

### Installing Per-Project MCPs

**Use local scope (default - works in all subdirectories):**
```bash
cd /path/to/your/repo
claude mcp add mongodb npx -y mongodb-mcp-server \
  -e MDB_MCP_CONNECTION_STRING="mongodb://localhost:27017/mydb?authSource=admin"
```

**Verify in subdirectory:**
```bash
cd /path/to/your/repo/some/deep/subdirectory
claude mcp list
# Should still show mongodb server
```

### Custom MCP Development

**Location:** `~/repos/claude_mcp/<name>_mcp/`

**Add to `~/.claude/mcp_config.json`:**
```json
{
  "servers": {
    "prism": {
      "command": "python3",
      "args": ["-m", "prism_mcp.interfaces.mcp_server"],
      "env": {
        "PYTHONPATH": "~/repos/claude_mcp/prism_mcp",
        "NEO4J_URI": "bolt://localhost:7688",
        "QDRANT_HOST": "localhost",
        "QDRANT_PORT": "6334"
      }
    }
  }
}
```

**Restart Claude Code after config changes.**

---

## Using MCP Tools

### When to Use MCP vs Built-in Tools

**Use MCP when:**
- Structured API access needed (GitHub issues > parsing `gh` output)
- Complex queries (MongoDB aggregations > bash mongosh parsing)
- Frequent operations (PRISM memory > manual file management)
- Safety/validation required (Database MCPs prevent SQL injection)

**Use built-in Bash when:**
- Simple one-off commands
- Tools already available via CLI
- No structured parsing needed
- MCP overhead not justified

**Example comparison:**

| Task | Via Bash | Via MCP | Winner |
|------|----------|---------|--------|
| Create GitHub PR | `gh pr create` | `mcp__github__create_pull_request` | MCP (structured, safer) |
| List files | `ls -la` | Filesystem MCP | Bash (simpler) |
| Query MongoDB | `mongosh --eval "..."` | `mcp__mongodb__find` | MCP (structured results) |
| Search packages | `npm search foo` | `mcp__package_registry__search_npm_packages` | MCP (detailed info) |

### Integration Patterns

**Pattern 1: Database Queries**
```
Query the MongoDB rthree database for all assets where info.owner.subID = $SUB1
```
Claude will use `mcp__mongodb__find` tool automatically.

**Pattern 2: Multi-step Workflows**
```
1. Search Jira for open INT tickets assigned to me
2. For each ticket, get dev info (commits, PRs)
3. Summarize work in progress
```
Claude will chain multiple MCP tools (Jira search → Jira dev info → text summary).

**Pattern 3: Memory + Code**
```
Before implementing this feature, check PRISM for similar patterns we've used
```
Claude will query PRISM MCP, retrieve past decisions, apply patterns to current work.

**Pattern 4: Package Research**
```
Find Python packages for PDF parsing, check versions and vulnerabilities
```
Claude will use Package Registry MCP to search PyPI and get security info.

### Best Practices

**Do:**
- Use specific tool names when you know what you want
- Let Claude choose tools for complex workflows
- Check PRISM before major decisions (if PRISM MCP active)
- Use project-scoped databases (MongoDB, Redis) for per-project work

**Don't:**
- Install MCPs "just in case" (context overhead adds up)
- Use MCP when Bash is simpler
- Store production secrets in MCP config (use env vars or secrets manager)
- Mix scopes unnecessarily (use local for projects, user for globals)

---

## Troubleshooting

### Server Not Starting

**Symptoms:** MCP tools not available after restart, `claude mcp list` shows "✗ Disconnected"

**Quick Fixes:**

1. **Verify command path:**
   ```bash
   which mcp-server-github  # Should return path
   which uvx  # For ObsidianPilot
   ```

2. **Check config syntax:**
   ```bash
   python3 -m json.tool ~/.claude/mcp_config.json
   python3 -m json.tool ~/.config/claude/mcp_config.json
   ```

3. **Test server manually:**
   ```bash
   python3 -m prism_mcp.interfaces.mcp_server  # Custom MCP
   npx @modelcontextprotocol/server-github     # NPM MCP
   ```

4. **Restart Claude Code** - Config changes require full restart (exit and relaunch)

### Connection Issues

**Database connection failed:**

1. **Verify service is running:**
   ```bash
   mongosh "mongodb://localhost:27017/mydb?authSource=admin"
   redis-cli ping
   psql "postgresql://user:pass@localhost:5432/dbname"
   ```

2. **Check network access:**
   ```bash
   nc -zv localhost 27017  # MongoDB
   nc -zv localhost 5432   # PostgreSQL
   nc -zv localhost 6379   # Redis
   ```

3. **Verify credentials** - Double-check username, password, database name

**GitHub MCP not connecting:**

1. **Verify token:** `gh auth status`
2. **Check token permissions:** repo, workflow, read:org scopes
3. **Regenerate if expired** - Update config with new token

### Configuration Errors

**"Project MCP servers require approval" (working in subdirectories):**

**Fix:** Remove and re-add with local scope:
```bash
claude mcp remove -s project server-name
claude mcp add server-name command -e KEY=value  # Uses local by default
```

**"Too many tools - context budget exceeded":**

**Fix:** Increase token budget in `~/.config/claude/settings.json`:
```json
{
  "mcpSettings": {
    "maxToolsTokens": 60000
  }
}
```

**Default:** 25k tokens (8-10 servers)
**Recommended:** 60k tokens (20+ servers)

**For advanced troubleshooting, see `reference.md`.**

---

## Token Budget Management

### Planning New MCPs

**Before adding an MCP, ask:**
1. What value does this provide over existing tools?
2. Can I accomplish this with Bash?
3. How often will I use this?
4. What's the token cost? (check MCP documentation)

**Token budget guidelines:**
- Small MCPs (1-5 tools): ~1-2k tokens
- Medium MCPs (5-15 tools): ~2-5k tokens
- Large MCPs (15-50 tools): ~5-15k tokens

**Calculate headroom:**
- Default budget: 200k tokens
- ~50k for tools (25% overhead OK)
- ~150k for actual work

### Optimizing Token Usage

**Strategies:**
1. **Per-project MCPs** - Only load MongoDB/Redis when working on that project
2. **Remove unused MCPs** - Uninstall MCPs you added but never use
3. **Increase budget** - Raise `maxToolsTokens` to 60k if needed
4. **Selective installation** - Don't install "nice to have" MCPs

---

## Quick Reference Commands

### Managing MCPs

```bash
# List all configured servers
claude mcp list

# Add server (local scope - recommended for projects)
claude mcp add server-name command -e KEY=value

# Add server (user scope - global tools)
claude mcp add -s user server-name command -e KEY=value

# Remove server
claude mcp remove server-name

# Get server details
claude mcp get server-name

# Test server connection
claude mcp test server-name
```

### Configuration Files

```bash
# Custom MCPs (your servers)
cat ~/.claude/mcp_config.json

# External MCPs (third-party)
cat ~/.config/claude/mcp_config.json

# Per-project MCPs
cat .mcp.json  # in project root

# Claude Code settings
cat ~/.config/claude/settings.json
```

### Testing MCP Servers

```bash
# Test custom MCP
python3 -m prism_mcp.interfaces.mcp_server

# Test NPM MCP
npx @modelcontextprotocol/server-github

# Test ObsidianPilot
~/.local/bin/uvx obsidianpilot
```

### Debugging

```bash
# Validate JSON config
python3 -m json.tool ~/.claude/mcp_config.json

# Check service connectivity
nc -zv localhost 27017  # MongoDB
nc -zv localhost 6379   # Redis

# Test database connection
mongosh "mongodb://localhost:27017/mydb?authSource=admin"
redis-cli ping

# Check logs
tail -f ~/.config/claude/logs/claude-code-*.log
```

---

## Summary

**MCP extends Claude Code with specialized tools for databases, APIs, and services.**

**Key Principles:**
1. **Evaluate before installing** - Token overhead adds up
2. **Use local scope for projects** - Works in all subdirectories
3. **Use user scope for globals** - GitHub, Package Registry
4. **Test manually first** - Verify tools work before configuring MCP
5. **Monitor token budget** - Check `claude mcp list` to see tool counts

**When to Use This Skill:**
- Setting up new MCP servers
- Debugging MCP connection issues
- Understanding which MCP to use for a task
- Troubleshooting configuration problems
- Planning token budget for new MCPs

**For detailed server setups and advanced troubleshooting, see `reference.md`.**
