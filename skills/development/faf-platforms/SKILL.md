---
name: faf-platforms
description: Understand where FAF works - Claude Code (CLI + Skills), claude-faf-mcp (Desktop + MCP), vs claude.ai (web - file upload only). Explains faf-cli vs MCP differences, when to use each platform. Use when user asks "does FAF work with", "CLI vs MCP", "Claude Desktop vs claude.ai", or platform compatibility questions.
allowed-tools: Read, WebFetch
---

# FAF Platforms - Where FAF Works

## Purpose

Clarify where FAF works and how to use it across different Claude platforms: Claude Code (CLI), Claude Desktop (MCP), and claude.ai (web).

**The Goal:** Help users choose the right platform and tool for their workflow.

## When to Use

This skill activates when the user:
- Asks "Does FAF work with Claude Desktop?"
- Says "CLI vs MCP - which one?"
- Asks "Can I use FAF on claude.ai?"
- Says "What's the difference between faf-cli and claude-faf-mcp?"
- Needs platform compatibility guidance

**Trigger Words:** platform, Claude Desktop, claude.ai, CLI, MCP, compatibility, where works, which tool

## Platform Overview

### 1. Claude Code (Terminal + Skills)

**What it is:**
- Command-line interface for Claude
- Terminal-based coding assistant
- This skill is running IN Claude Code right now

**FAF integration:**
- ✅ **faf-cli** - Run `faf` commands directly
- ✅ **FAF Skills** - These skills activate automatically
- ✅ **Direct file access** - Read/write project.faf

**Best for:**
- Developers working in terminal
- Quick FAF operations (`faf init`, `faf score`)
- Using Skills for guided workflows
- Local development environments

**How to use FAF:**
```bash
# Install faf-cli
npm install -g faf-cli

# Install Skills (optional but recommended)
# Copy skills to ~/.config/claude-code/skills/

# Use naturally
"What is FAF?" → faf-teacher activates
"Set up project context" → faf-init activates
```

---

### 2. Claude Desktop (MCP Integration)

**What it is:**
- Desktop application for Claude
- Native Mac/Windows app
- Uses Model Context Protocol (MCP)

**FAF integration:**
- ✅ **claude-faf-mcp** - Official MCP server (Anthropic-approved)
- ✅ **33 MCP tools** - All faf-cli commands as tools
- ✅ **Persistent context** - project.faf always available

**Best for:**
- GUI preference over terminal
- Multi-project workflows
- Using other MCP servers alongside FAF
- Desktop-first development

**How to use FAF:**
```bash
# Install MCP server
npm install -g claude-faf-mcp

# Configure Claude Desktop
# Edit: ~/Library/Application Support/Claude/claude_desktop_config.json

{
  "mcpServers": {
    "faf": {
      "command": "npx",
      "args": ["-y", "claude-faf-mcp"]
    }
  }
}

# Restart Claude Desktop
# FAF tools appear automatically
```

**MCP Tools available:**
- faf_init, faf_score, faf_enhance, faf_validate
- faf_bi_sync, faf_migrate
- All 33 commands from faf-cli

---

### 3. claude.ai (Web Interface)

**What it is:**
- Web-based Claude interface
- Browser access (no installation)
- Simple file upload

**FAF integration:**
- ⚠️ **Limited** - File upload only
- ❌ No faf-cli commands
- ❌ No MCP integration
- ❌ No Skills

**Best for:**
- Quick demonstrations
- Sharing project.faf with others
- No installation scenarios
- One-off consultations

**How to use FAF:**
```
1. Create project.faf locally (use faf-cli elsewhere)
2. Open claude.ai
3. Upload project.faf file
4. Claude reads it as regular YAML
5. Limited - no scoring, no enhancement, no sync
```

**Limitations:**
- Cannot run `faf score` or other commands
- Cannot enhance or validate automatically
- No bidirectional sync
- Manual file upload each session

**Recommendation:** Use Claude Code or Claude Desktop for full FAF experience.

---

## Platform Comparison

| Feature | Claude Code (CLI) | Claude Desktop (MCP) | claude.ai (Web) |
|---------|-------------------|----------------------|-----------------|
| **faf-cli commands** | ✅ All 41 | Via MCP tools | ❌ None |
| **FAF Skills** | ✅ Yes | ❌ No | ❌ No |
| **MCP Tools** | ❌ No | ✅ 33 tools | ❌ No |
| **File access** | ✅ Direct | ✅ Direct | ⚠️ Upload only |
| **Auto-activation** | ✅ Skills | ✅ Tools | ❌ Manual |
| **Best for** | Terminal workflow | Desktop app | Quick demos |
| **Installation** | npm install | npm + config | None |

## faf-cli vs claude-faf-mcp

### faf-cli (Command-Line Interface)

**What it is:**
- Standalone CLI tool
- Works WITHOUT Claude
- 41 commands total
- npm or Homebrew

**When to use:**
```bash
# Standalone operations
faf init          # Create project.faf
faf score         # Check AI-readiness
faf enhance       # Improve content
faf bi-sync       # Sync to CLAUDE.md
faf validate      # Check format

# Works independently
# No Claude required
# Fast and direct
```

**Advantages:**
- Works anywhere (with or without Claude)
- Fastest execution
- Complete command set
- Scriptable

**Use cases:**
- CI/CD pipelines
- Git pre-commit hooks
- Batch processing
- Automation scripts

---

### claude-faf-mcp (MCP Server)

**What it is:**
- MCP server for Claude Desktop
- Requires Claude Desktop app
- Bridges faf-cli to MCP protocol
- 33 tools exposed

**When to use:**
```
# Inside Claude Desktop
Ask Claude:
"Initialize project context" → calls faf_init
"Check my score" → calls faf_score
"Enhance my FAF" → calls faf_enhance

# MCP tools activate automatically
# Claude decides when to use them
# Integrated workflow
```

**Advantages:**
- GUI workflow
- Automatic tool selection
- Works with other MCP servers
- Integrated with Claude Desktop

**Use cases:**
- Desktop-first development
- Multi-tool MCP setups
- GUI preference
- Conversational workflow

---

### Which Should I Use?

**Use faf-cli if:**
- ✅ You work in terminal
- ✅ You want direct command access
- ✅ You need automation/scripting
- ✅ You use Claude Code (this)
- ✅ You want fastest execution

**Use claude-faf-mcp if:**
- ✅ You use Claude Desktop app
- ✅ You prefer GUI over CLI
- ✅ You use other MCP servers
- ✅ You want conversational workflow
- ✅ You want Claude to decide which tool

**Use BOTH if:**
- ✅ Terminal + Desktop workflows
- ✅ Maximum flexibility
- ✅ They work together seamlessly

---

## Installation Quick Guide

### For Claude Code Users

```bash
# Install CLI (required)
npm install -g faf-cli

# Install Skills (recommended)
git clone https://github.com/Wolfe-Jam/faf-agent-toolkit.git
cd faf-agent-toolkit
./install.sh

# Skills activate automatically in Claude Code
```

### For Claude Desktop Users

```bash
# Install MCP server
npm install -g claude-faf-mcp

# Configure
# Edit: ~/Library/Application Support/Claude/claude_desktop_config.json
# Add faf MCP server configuration

# Restart Claude Desktop
# Tools appear automatically
```

### For claude.ai Users

```bash
# Create project.faf elsewhere (Claude Code or Desktop)
faf init

# Upload to claude.ai manually
# Limited functionality
# Consider upgrading to Claude Code or Desktop for full experience
```

---

## Common Questions

**Q: Can I use FAF without Claude?**
A: Yes! faf-cli works standalone. Create, score, enhance project.faf files anywhere.

**Q: Do Skills work in Claude Desktop?**
A: No. Skills are Claude Code only. Claude Desktop uses MCP tools instead.

**Q: Can I use MCP in Claude Code?**
A: Not directly. Claude Code uses Skills. Claude Desktop uses MCP.

**Q: Which is better - CLI or MCP?**
A: Neither. Different workflows. CLI for terminal, MCP for Desktop app.

**Q: Does claude.ai support MCP?**
A: No. claude.ai is web-only. No MCP, no CLI, just file upload.

---

## Platform Compatibility Matrix

**Supported Claude Platforms:**
- ✅ Claude Code (CLI + Skills)
- ✅ Claude Desktop (MCP)
- ⚠️ claude.ai (file upload only)

**Supported AI Tools (via faf-cli):**
- ✅ Claude (all platforms)
- ✅ Cursor
- ✅ Gemini CLI
- ✅ OpenAI Codex CLI (NOT ChatGPT)
- ✅ Warp
- ✅ Windsurf
- ✅ ANY AI that reads YAML

**Why universal?**
project.faf is just a YAML file. Any AI can read YAML. No special integration required.

---

## Next Steps

**From here:**

**If on Claude Code:**
- You're in the right place
- FAF Skills active
- Use `faf` commands directly

**If want Claude Desktop:**
- Install claude-faf-mcp
- Configure MCP
- Restart Desktop app

**If on claude.ai:**
- Consider Claude Code or Desktop for full experience
- Or use faf-cli separately and upload results

---

**Generated by FAF Skill: faf-platforms v1.0.0**
**"CLI for speed. MCP for Desktop. Skills for Claude Code. Universal by design."**
