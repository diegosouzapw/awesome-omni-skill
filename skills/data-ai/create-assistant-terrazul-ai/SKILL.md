---
name: create-assistant
description: Creates complete Terrazul AI assistant packages with proper directory structure, agents.toml manifest, Handlebars templates, and dynamic content using askUser/askAgent helpers. Use when creating reusable AI assistant configurations for Claude, Codex, or Gemini.
---

# Create Terrazul Assistant Package

This skill guides you through creating a complete Terrazul package - a reusable, distributable AI assistant configuration that works across Claude Code, Codex, and Gemini.

## When to Use This Skill

Activate this skill when the user wants to:
- Create a reusable AI assistant package
- Build a shareable configuration for Claude, Codex, or Gemini
- Set up a package with skills, agents, and MCP servers
- Create dynamic templates using `askUser` and `askAgent`
- Understand Terrazul package structure and best practices

## Related Skills

This skill works best in combination with:

| Skill | Use For |
|-------|---------|
| `create-skill` | Creating individual skills within the package |
| `add-mcp` | Finding and configuring MCP servers for the package |
| `create-agent` | Creating Claude agents for the package |
| `create-command` | Creating slash commands for Claude |

When building a full-featured assistant package, use these skills to create the individual components after setting up the package structure.

## Terrazul Package Overview

A Terrazul package is a distributable unit containing:
- **Configuration manifest** (`agents.toml`) - package metadata and exports
- **Context templates** (`.md.hbs` or `.md`) - documentation for AI tools
- **Operational files** - skills, agents, commands, MCP configs
- **Dynamic content** - interactive prompts and AI-generated content

## Package Creation Workflow

### 1. Gather Requirements

Ask the user:
- What is the purpose of this assistant package?
- Which AI tools should it support? (Claude, Codex, Gemini)
- What skills/capabilities should it provide?
- Does it need MCP server integrations?
- Should content be dynamic (using askUser/askAgent)?

### 2. Create Directory Structure

Create the package directory with this structure:

```
my-package/
├── agents.toml              # REQUIRED: Package manifest
├── README.md                # RECOMMENDED: Documentation
├── templates/               # Template files
│   ├── CLAUDE.md.hbs        # Claude context (or .md if no dynamic content)
│   ├── AGENTS.md.hbs        # Codex context
│   ├── GEMINI.md.hbs        # Gemini context
│   ├── claude/              # Claude-specific files
│   │   ├── mcp_servers.json.hbs
│   │   ├── skills/
│   │   │   └── [skill-name]/
│   │   │       └── SKILL.md
│   │   ├── agents/
│   │   │   └── [agent-name].md
│   │   └── commands/
│   │       └── [command-name].md
│   ├── codex/               # Codex-specific files
│   │   └── config.toml.hbs
│   └── gemini/              # Gemini-specific files
│       ├── settings.json.hbs
│       └── commands/
│           └── [command-name].toml
└── prompts/                 # Optional: External prompt files for askAgent
    └── analyze.txt
```

**Required vs Optional Components**:

| Component | Status | Purpose |
|-----------|--------|---------|
| `agents.toml` | **Required** | Package manifest |
| `README.md` | Recommended | Package documentation |
| `templates/` | Recommended | All template files |
| Context templates | Optional | CLAUDE.md, AGENTS.md, GEMINI.md |
| `templates/claude/skills/` | Optional | Claude skills |
| `templates/claude/agents/` | Optional | Claude agents |
| `templates/claude/commands/` | Optional | Claude commands |
| `prompts/` | Optional | External askAgent prompts |

### 3. Create agents.toml Manifest

The manifest defines package metadata and exports:

```toml
[package]
name = "@owner/package-name"           # Required for publish
version = "1.0.0"                      # Required for publish
description = "Brief package description"
repository = "https://github.com/..."
license = "MIT"
keywords = ["ai-assistant", "claude", "codex"]
authors = ["Name <email@example.com>"]

[compatibility]
claude = ">=0.2.0"
codex = "*"
gemini = "*"

[exports.claude]
template = "templates/CLAUDE.md.hbs"
skillsDir = "templates/claude/skills"
agentsDir = "templates/claude/agents"
commandsDir = "templates/claude/commands"
mcpServers = "templates/claude/mcp_servers.json.hbs"

[exports.codex]
template = "templates/AGENTS.md.hbs"
skillsDir = "templates/claude/skills"

[exports.gemini]
template = "templates/GEMINI.md.hbs"
skillsDir = "templates/claude/skills"
commandsDir = "templates/gemini/commands"
mcpServers = "templates/gemini/settings.json.hbs"
```

**Manifest Rules**:
- `name` format: `@scope/package-name` (lowercase, hyphens allowed)
- `version` must be valid semver
- All paths in `[exports]` must be relative to package root
- Tool keys: `claude`, `codex`, `gemini`, `cursor`, `copilot`

### 4. Create Context Templates

Context templates provide instructions to the AI tool. Use `.hbs` extension for dynamic content, `.md` for static.

**Static template (CLAUDE.md)**:
```markdown
# Package Name

This package provides [description].

## Available Skills

| Skill | Description |
|-------|-------------|
| `skill-name` | Brief description |

## Quick Start

[Usage instructions]
```

**Dynamic template (CLAUDE.md.hbs)**:
```handlebars
{{~ var projectInfo = askAgent("""
Analyze this repository and return:
{
  "name": "repository name",
  "description": "brief description",
  "languages": "primary languages used"
}
""", { json: true }) ~}}

# {{ vars.projectInfo.name }}

{{ vars.projectInfo.description }}

## Tech Stack
Languages: {{ vars.projectInfo.languages }}
```

### 5. Add Dynamic Content with askUser/askAgent

#### askUser - Interactive Prompts

Gather information from users during package rendering:

```handlebars
{{~ var projectPitch = askUser('Describe your project in 1-2 sentences.') ~}}
{{~ var targetAudience = askUser('Who is the primary user of this assistant?') ~}}
{{~ var constraints = askUser('List any constraints or guardrails.', {
  default: 'None',
  placeholder: 'Enter constraints or press Enter for none'
}) ~}}

# Project Overview

{{ vars.projectPitch }}

**Primary User**: {{ vars.targetAudience }}

**Constraints**: {{ vars.constraints }}
```

**askUser Options**:
- `default` - Default value if user presses Enter
- `placeholder` - Hint text shown when empty

#### askAgent - AI-Powered Analysis

Generate content using AI analysis of the codebase:

```handlebars
{{~ var analysis = askAgent("""
Analyze this repository structure and provide:
{
  "mainDirectories": "key directories and their purposes",
  "buildTools": "build tools used",
  "testFramework": "testing framework if present"
}
""", { json: true }) ~}}

## Repository Structure

{{ vars.analysis.mainDirectories }}

## Development

Build: {{ vars.analysis.buildTools }}
Tests: {{ vars.analysis.testFramework }}
```

**askAgent Options**:
- `json: true` - Expect JSON response
- `tool: 'claude'` - Specify AI tool
- `systemPrompt: 'custom prompt'` - Override system prompt
- `timeoutMs: 60000` - Custom timeout

**IMPORTANT Best Practice**: Use ONE comprehensive askAgent call with flat JSON instead of multiple sequential calls:

```handlebars
{{~ var projectData = askAgent("""
Analyze this repository and return a flat JSON object:
{
  "name": "repository name",
  "description": "brief description",
  "languages": "TypeScript, Python",
  "frameworks": "Next.js, FastAPI",
  "directories": "src: source code, tests: test files"
}
""", { json: true }) ~}}
```

This is more efficient and produces more coherent results than multiple separate calls.

### 6. Create Skills (Optional)

**TIP**: Use the `create-skill` skill for detailed guidance on creating skills. It provides templates, validation checklists, and best practices for different skill types.

Add skills in `templates/claude/skills/[skill-name]/SKILL.md`:

```markdown
---
name: skill-name
description: Third-person description of what this skill does and when to use it
---

# Skill Title

[Overview of what this skill provides]

## When to Use

Activate this skill when:
- [Trigger condition 1]
- [Trigger condition 2]

## Instructions

[Specific instructions for Claude to follow]

## Examples

[Concrete usage examples]
```

### 7. Create Agents (Optional)

Add Claude agents in `templates/claude/agents/[agent-name].md`:

```markdown
---
name: agent-name
description: What this agent does
model: sonnet
color: blue
tools:
  - Read
  - Grep
  - Glob
  - Edit
examples:
  - "Review this PR for security issues"
  - "Analyze the architecture"
---

You are a specialized agent for [purpose].

## Your Role

[Agent's responsibilities]

## Guidelines

[Specific instructions for the agent]
```

### 8. Configure MCP Servers (Optional)

**TIP**: Use the `add-mcp` skill to search mcp.so for servers, get configuration details, and properly set up MCP integrations. It handles searching, fetching server details, and generating correct configurations.

Add MCP configuration in `templates/claude/mcp_servers.json.hbs`:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "context7": {
      "command": "npx",
      "args": ["-y", "@context7/mcp-server"]
    }
  }
}
```

**CRITICAL REQUIREMENT: Include setup-mcp Command**

Every package with MCP servers **MUST** include the `setup-mcp` command. This is not optional.

When the package includes MCP server configuration:

1. **Copy the setup-mcp command** to `templates/claude/commands/setup-mcp.md`
2. **Copy the Gemini version** to `templates/gemini/commands/setup-mcp.toml`
3. **Update README.md** to include post-installation section:
   ```markdown
   ## Setup Required

   After installing, configure MCP credentials:
   ```
   /setup-mcp
   ```
   ```
4. **Update CLAUDE.md** to list the MCP servers and note that `/setup-mcp` must be run

The `setup-mcp` command guides users through setting up environment variables in their shell profile (`~/.zshenv`, `~/.bashrc`) so MCP servers can authenticate with external services.

### 9. Write README.md

Document the package for users:

```markdown
# @owner/package-name

Brief description of what this package provides.

## Features

- Feature 1
- Feature 2

## Installation

\`\`\`bash
tz add @owner/package-name
tz apply
\`\`\`

## Usage

[How to use the package]

## Configuration

[Any configuration needed]

## License

MIT
```

### 10. Validate Package

Before publishing, validate the package:

```bash
# Check structure and manifest
tz validate

# Test rendering locally
tz run --force

# Verify generated files
ls -la agent_modules/@owner/package-name/
```

## Validation Checklist

Before considering the package complete:

- [ ] `agents.toml` has valid `name` and `version`
- [ ] All template paths in `[exports]` exist
- [ ] Context templates render without errors
- [ ] Skills have valid frontmatter (name, description)
- [ ] Agents have valid frontmatter (name, description, model)
- [ ] MCP configs have valid JSON/TOML syntax
- [ ] README.md documents installation and usage
- [ ] Package works with `tz apply --force`
- [ ] **If MCP servers configured** → `setup-mcp.md` exists in `templates/claude/commands/`
- [ ] **If MCP servers configured** → `setup-mcp.toml` exists in `templates/gemini/commands/`
- [ ] **If MCP servers configured** → README mentions `/setup-mcp` in post-installation steps
- [ ] **If MCP servers configured** → CLAUDE.md documents required credentials

## Common Patterns

### Documentation-Focused Package

For packages that primarily provide context and guidelines:

```
my-docs-package/
├── agents.toml
├── README.md
└── templates/
    ├── CLAUDE.md.hbs      # Dynamic analysis of codebase
    ├── AGENTS.md           # Static codex docs
    └── claude/
        └── skills/
            └── guidelines/
                └── SKILL.md
```

### Skills-Focused Package

For packages that add capabilities:

```
my-skills-package/
├── agents.toml
├── README.md
└── templates/
    ├── CLAUDE.md
    └── claude/
        └── skills/
            ├── skill-1/
            │   ├── SKILL.md
            │   ├── reference.md
            │   └── examples.md
            └── skill-2/
                └── SKILL.md
```

### Full-Featured Package

For comprehensive packages:

```
my-full-package/
├── agents.toml
├── README.md
├── prompts/
│   └── analyze.txt
└── templates/
    ├── CLAUDE.md.hbs
    ├── AGENTS.md.hbs
    ├── GEMINI.md.hbs
    ├── claude/
    │   ├── mcp_servers.json.hbs
    │   ├── skills/
    │   ├── agents/
    │   └── commands/
    ├── codex/
    │   └── config.toml.hbs
    └── gemini/
        ├── settings.json.hbs
        └── commands/
```

## Tips for Effective Packages

1. **Start Simple**: Begin with `agents.toml` and one context template
2. **Use Dynamic Content Wisely**: `askAgent` is expensive; use sparingly
3. **Flat JSON Only**: Always request flat JSON structures from `askAgent`
4. **One Comprehensive Call**: Prefer single `askAgent` with multiple fields over multiple calls
5. **Cache-Friendly Prompts**: Design stable prompts for better caching
6. **Platform-Agnostic Skills**: Write skills that work across tools when possible
7. **Document Everything**: Good README and context files save user time
8. **Test Before Publishing**: Run `tz validate` and `tz apply --force`
9. **Use Related Skills**: Leverage `create-skill` for skill creation, `add-mcp` for MCP configuration, and `create-agent` for agent definitions

## Working with PDF Context

Assistants are often given PDFs for context (documentation, specifications, guidelines). Large PDFs can hit token limits when read directly.

**Best Practice**: Use `pdftotext` to convert PDFs to plain text:

```bash
# Convert PDF to text
pdftotext input.pdf output.txt

# Convert with layout preservation
pdftotext -layout input.pdf output.txt
```

Use the converted text however is appropriate for your use case. This avoids token limits and makes the content more accessible to the AI tool.

## Reference Materials

For detailed information:
- `reference.md` - Complete manifest and template reference
- `examples.md` - Real-world package examples
- `templates/` - Starter templates for different package types
