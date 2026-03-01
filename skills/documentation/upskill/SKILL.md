---
name: upskill
description: Meta-skill that learns new capabilities from NPM and pip package registries. Use when user asks to integrate with a service (e.g., "I need to use Asana"), mentions an API we don't have a skill for, or explicitly invokes "/skill upskill <package>". Automatically searches NPM and pip, installs packages globally to ~/.claude/, and generates skill documentation. DEFAULT: Any time system needs to use an API, check package registries first.
---

# Upskill - Learn New Capabilities from Package Registries

## Purpose

Meta-skill that automatically learns new capabilities by searching NPM and pip registries, installing packages globally, and generating skill documentation. Enables the system to recursively teach itself new tools.

**How Skills Work:**
- Skills are **documentation/instructions** loaded into context via the Skill tool
- Main context reads the skill, then writes and executes scripts directly
- **No subagents** - execution happens in main context for simplicity and speed
- Skills are lightweight reference material (~20-30 lines), full docs separate

## When to Use

**Auto-trigger on:**
- User asks to integrate with a service: "I need to use Asana", "Connect to Slack"
- User mentions an API/system we don't have a skill for
- Explicit invocation: `/skill upskill <package-name>`
- **DEFAULT**: Any time system needs to use an API → check package registries first

## Process

### Step 1: Search Both Registries (NPM first, then pip)

```bash
# Search NPM
curl -s "https://registry.npmjs.org/-/v1/search?text=<package-name>&size=20" | jq '.objects[] | {name: .package.name, version: .package.version, description: .package.description, downloads: .package.downloadsLastWeek}'

# Search PyPI (package must exist for direct query)
curl -s "https://pypi.org/pypi/<package-name>/json" | jq '{name: .info.name, version: .info.version, description: .info.summary}'
```

**Selection criteria:**
- Official/maintained packages
- Download counts and maintenance frequency
- TypeScript definitions (NPM) or type stubs (pip)
- Recently updated

### Step 2: Fetch Package Info

**NPM packages:**
```bash
# Get full metadata
curl -s "https://registry.npmjs.org/<package-name>" | jq > npm-metadata.json

# Key fields: .versions, .dist.tarball, .readme, .types, .repository
```

**pip packages:**
```bash
# Get full metadata
curl -s "https://pypi.org/pypi/<package-name>/json" | jq > pypi-metadata.json

# Key fields: .info, .releases, .urls, .project_urls
```

### Step 3: Install Package Globally

**CRITICAL:** Install to `~/.claude/` (not project-specific, not system-wide)

**For NPM packages:**
```bash
cd ~/.claude
npm install <package-name>
# Verifies: ~/.claude/node_modules/<package-name>/
```

**For pip packages:**
```bash
pip install --target ~/.claude/lib <package-name>
# Verifies: ~/.claude/lib/python3.X/site-packages/<package-name>/
```

### Step 4: Analyze API Surface

**NPM packages (TypeScript):**
- Check for `.d.ts` files in package
- Parse exported functions, classes, interfaces
- Extract main entry points from `package.json`
- Identify authentication patterns (API keys, OAuth)

**pip packages (Python):**
- Look for `.pyi` type stub files
- Inspect module structure
- Extract docstrings and function signatures
- Identify authentication requirements

**Fallback for both:**
- Extract examples from README.md
- Look for common patterns (Client classes, resources)
- Search for "authentication", "API key" patterns

### Step 5: Generate Skill Structure

**CRITICAL:** Use correct directory structure

```bash
~/.claude/skills/<package-name>/
├── SKILL.md              # Main skill file (minimal, loaded into context)
├── full-docs.md          # Complete API reference (reference only)
└── metadata.json         # Machine-readable metadata
```

**SKILL.md template** (minimal, ~20-30 lines):
```markdown
---
name: <package-name>
description: Use when <specific use case> - supports <main features>. Package installed at ~/.claude/node_modules/<package>/ (or ~/.claude/lib/pythonX.Y/site-packages/<package>/)
---

# <Package-Name> Skill

## Overview
<Brief description of what the package does>

## Package Info
- **Registry**: NPM | pip
- **Package**: <package-name>
- **Version**: X.Y.Z
- **Runtime**: Node.js | Python
- **Location**: ~/.claude/node_modules/<package>/ | ~/.claude/lib/python3.X/site-packages/<package>/

## Installation
Already installed globally at ~/.claude/<location>
Available in all projects.

## Authentication
<If required, explain how to get and set credentials>
<If not required, state: No authentication required>

## Quick Reference
<Main functions/methods with signatures>
```

**metadata.json template:**
```json
{
  "name": "<package-name>",
  "package": "<package-name>",
  "version": "X.Y.Z",
  "runtime": "node" | "python",
  "registry": "npm" | "pypi",
  "last_checked": "2025-01-13T10:00:00Z",
  "last_updated": "2025-01-13T10:00:00Z",
  "capabilities": ["cap1", "cap2"],
  "triggers": ["keyword1", "keyword2"],
  "credential_env": null | "CLAUDE_<SERVICE>_TOKEN",
  "install_path": "~/.claude/node_modules/<package>/",
  "requires_auth": false | true
}
```

### Step 6: Update Version Cache

```bash
# Update ~/.claude/skill-cache/package-versions.json
{
  "<package-name>": {
    "installed": "X.Y.Z",
    "latest": "X.Y.Z",
    "last_checked": "2025-01-13T10:00:00Z",
    "update_available": false,
    "critical": false,
    "breaking": false
  }
}
```

### Step 7: Credential Setup (if required)

**If package requires authentication:**

1. Identify auth mechanism (API key, OAuth, token)
2. Provide URL to get credentials
3. Guide user to set environment variable:
   ```bash
   # Recommended: App-specific prefix
   export CLAUDE_<SERVICE>_TOKEN=your_token_here

   # Or add to ~/.zshrc or ~/.bashrc
   ```

4. Test credentials before confirming success

### Step 8: Test and Confirm

**Verify installation:**
```bash
# For NPM packages
node -e "const pkg = require('/Users/davidpreil/.claude/node_modules/<package>'); console.log('OK');"

# For pip packages
python3 -c "import sys; sys.path.insert(0, '/Users/davidpreil/.claude/lib'); import <package>; print('OK')"
```

**Confirm success:**
```
✓ Learned <package-name> vX.Y.Z (NPM|pip)
✓ Skill: ~/.claude/skills/<package-name>/
✓ Runtime: Node.js | Python
✓ Auth: <requirements>

Ready to use!
```

## Package Selection Strategy

When both NPM and pip have options:

**Prefer NPM if:**
- Official package with better TypeScript definitions
- More downloads/active maintenance
- Recently updated
- Better documentation

**Prefer pip if:**
- Python-only service or domain
- Official SDK maintained by service
- Better Python ecosystem integration

**Ask user if:**
- Tie in quality
- Different capabilities between versions

**Always inform:**
- If alternative exists in other registry
- Trade-offs of selected option

## Registry APIs

### NPM Registry
```bash
# Search
curl "https://registry.npmjs.org/-/v1/search?text=<query>&size=20"

# Package info
curl "https://registry.npmjs.org/<package-name>"

# Specific version
curl "https://registry.npmjs.org/<package-name>/<version>"
```

### PyPI Registry
```bash
# Package info
curl "https://pypi.org/pypi/<package-name>/json"

# Search (no simple API, use web search or alternative)
```

## Type Parsing Guide

### NPM (TypeScript)
- Parse `.d.ts` files for function signatures
- Extract exported classes and interfaces
- Look for JSDoc comments
- Fall back to README examples if no types

### pip (Python)
- Parse `.pyi` type stub files
- Extract docstrings from modules
- Inspect function signatures
- Fall back to README examples

## Error Handling

**Package not found:**
- Try alternative registry
- Suggest similar names
- Offer manual search

**Install fails:**
- Check permissions
- Verify network connection
- Show error message
- Suggest fixes

**Package lacks types:**
- Warn user but proceed
- Use README examples
- Note limitations

**Auth required but missing:**
- Prompt for credentials
- Provide setup instructions
- Don't proceed without auth

## Example Workflow

**User**: "I need to work with Notion"

**System execution:**

1. Search NPM for "notion" → Found `@notionhq/client`
   - Official package ✓
   - TypeScript definitions ✓
   - Weekly downloads: 300K ✓

2. Install: `npm install @notionhq/client` to ~/.claude/node_modules/

3. Analyze:
   - Main export: `Client`
   - Auth: Requires integration token
   - Key methods: `databases.query`, `pages.retrieve`, etc.

4. Generate skill structure:
   ```
   ~/.claude/skills/notion/
   ├── SKILL.md
   ├── full-docs.md
   └── metadata.json
   ```

5. Credential setup:
   ```
   Get integration token: https://www.notion.so/my-integrations
   Set: export CLAUDE_NOTION_TOKEN=your_token
   ```

6. Test and confirm:
   ```
   ✓ Learned @notionhq/client v2.2.3 (NPM)
   ✓ Skill: ~/.claude/skills/notion/
   ✓ Runtime: Node.js
   ✓ Auth: Set CLAUDE_NOTION_TOKEN
   ```

## How Generated Skills Are Used

When a generated skill is invoked (e.g., `/skill notion`):

1. Skill tool loads `~/.claude/skills/notion/SKILL.md` into context
2. Main context reads the skill documentation
3. Main context writes a script (e.g., `notion-query.js`)
4. Main context executes the script directly: `node notion-query.js`
5. Results returned immediately

**No subagents** - simple, fast, direct execution in main context.

## Important Notes

- Always install to `~/.claude/` (never project-specific)
- Use subdirectory structure: `skills/<package>/SKILL.md`
- Keep SKILL.md minimal (~20-30 lines) for context efficiency
- Full documentation goes in `full-docs.md`
- Test before confirming success
- Inform user of alternatives in other registry
- Document auth requirements clearly
- Update version cache for all packages
- Skills are documentation, execution is direct and simple
