---
name: organize-agent-docs
description: Organize project agentic documentation into universal (AGENTS.md) and agent-specific files (CLAUDE.md, GEMINI.md, etc.). Use when asked to "organize agent docs", "separate agent instructions", "restructure AGENTS.md", or when a project has agent documentation that mixes universal and tool-specific content.
allowed-tools: Read, Edit, Write, Bash
---

# Organize Agent Documentation

Reorganize project agentic documentation to separate universal project information from agent-specific tooling and workflows.

## When to Use This Skill

Use this skill when:

- A project has grown to include multiple AI agents (Claude, Gemini, Cursor, Copilot, etc.)
- AGENTS.md or CLAUDE.md contains both universal project info and agent-specific tooling
- You need to add a new agent to an existing project
- Agent documentation has become difficult to maintain due to duplication
- User asks to "organize agent docs", "separate agent instructions", or similar

## What This Skill Does

Restructures agentic documentation following a clear separation of concerns pattern:

1. **AGENTS.md** - Universal project documentation (primary source of truth)
   - Product overview and user persona
   - Recent changes and version history
   - Architecture notes and testing requirements
   - Common agent tasks and workflows
   - Universal critical rules (applies to all agents)

2. **CLAUDE.md** - Claude-specific instructions
   - MCP server configuration (XcodeBuildMCP, App Store Connect MCP, etc.)
   - Claude-specific build preferences and tooling
   - References to global `~/.claude/CLAUDE.md` patterns
   - Claude-specific critical rules

3. **GEMINI.md** - Gemini-specific instructions
   - Gemini Code Assist integration patterns
   - Standard CLI tool alternatives (for agents without MCP)
   - Gemini API integration notes (if applicable)

4. **Additional agent files** (as needed)
   - CURSOR.md, COPILOT.md, CODEX.md, etc.

## Benefits

✅ **Clear separation of concerns** - Universal vs. agent-specific content
✅ **Single source of truth** - AGENTS.md remains primary reference
✅ **Scalable** - Easy to add new agents without modifying universal docs
✅ **No duplication** - Agent files reference AGENTS.md, don't duplicate content
✅ **Maintainable** - Update universal rules once, agent-specific rules independently
✅ **Discoverable** - Clear structure helps agents find relevant instructions quickly

## Prerequisites

The project should have:

- Existing AGENTS.md or CLAUDE.md with agentic instructions
- Content that mixes universal project info with agent-specific tooling
- Multiple AI agents working on the project (or planning to add more)

## Usage

### Step 1: Analyze Current Documentation

Read the existing agentic documentation to understand what content exists:

```bash
# Read primary agent documentation
cat AGENTS.md  # or CLAUDE.md, whichever exists
cat GEMINI.md  # if exists
```

Identify content types:
- **Universal**: Product overview, architecture, testing, workflows, general rules
- **Claude-specific**: MCP servers, Claude tool preferences, Claude-only workflows
- **Gemini-specific**: Gemini APIs, Gemini-specific patterns
- **Other agents**: Cursor, Copilot, custom agents

### Step 2: Create AGENTS.md (Universal Documentation)

If AGENTS.md doesn't exist, create it. Otherwise, clean it to remove agent-specific content.

**Universal content to include:**
- Product overview and target user persona
- Recent changes and version history (applies to all agents)
- CLI documentation and automation workflows
- Architecture notes and design patterns
- Testing requirements (unit tests, integration tests, UI tests)
- Common agent tasks (adding tests, fixing bugs, updating docs)
- Universal critical rules (git workflow, version management, testing)

**Content to EXCLUDE from AGENTS.md:**
- MCP server documentation (move to CLAUDE.md)
- Claude-specific tool preferences (move to CLAUDE.md)
- Agent-specific API integrations (move to respective agent files)

**Structure:**
```markdown
# ProjectName - AI Agent Instructions

**Version**: X
**Purpose**: Guide AI agents working on ProjectName
**Audience**: Claude Code, Gemini, and other AI development assistants

## Product Overview
[Product description, target user, core value]

## Recent Changes
[Last 3-5 sprints with what changed, files modified, impact]

## Scripting and Automation
[CLI tools, AppleScript, hooks, CI/CD - universal patterns]

## User Persona Summary
[Primary user, pain points, use cases, design principles]

## Architecture Notes
[Dual interface, platform strategy, testing requirements]

## Documentation Index
[Links to user-facing docs, technical docs, architecture]

## Common Agent Tasks
[Adding tests, fixing bugs, CLI changes, documentation updates]

## Critical Rules for AI Agents
[Universal rules that apply to ALL agents]

1. NEVER commit directly to `main`
2. ALWAYS test both platforms before committing
3. NEVER manually edit version numbers
4. ALWAYS read files before editing
5. NEVER create files unless necessary
6. Follow agent-specific instructions (see CLAUDE.md, GEMINI.md)
```

### Step 3: Create CLAUDE.md (Claude-Specific)

Extract Claude-specific content and create CLAUDE.md:

**Claude-specific content:**
- MCP server configuration (XcodeBuildMCP, App Store Connect MCP, etc.)
- Build tool preferences (xcodebuild over swift build, etc.)
- Claude-specific critical rules
- References to global `~/.claude/CLAUDE.md` settings

**Structure:**
```markdown
# Claude-Specific Agent Instructions

**⚠️ Read [AGENTS.md](AGENTS.md) first** for universal project documentation.

This file contains instructions specific to Claude Code agents.

## Claude-Specific Build Preferences
[Tool preferences like xcodebuild over swift build]

## MCP Server Configuration
[XcodeBuildMCP, App Store Connect MCP, custom MCP servers]

### XcodeBuildMCP
**Available Operations**: build_sim, test_sim, etc.
**Usage Pattern**: [Examples]
**Benefits**: [Why use MCP over direct commands]

### App Store Connect MCP
**Available Operations**: list_apps, get_metrics, etc.
**Usage Pattern**: [Examples]

## Claude-Specific Critical Rules
1. ALWAYS use XcodeBuildMCP tools
2. NEVER use swift build/test
3. Leverage MCP servers for automation
4. Follow global CLAUDE.md patterns

## Global Claude Settings
Your global Claude instructions: ~/.claude/CLAUDE.md
Key patterns: [Communication, Security, CI/CD, etc.]
```

### Step 4: Create GEMINI.md (Gemini-Specific)

Create GEMINI.md for Gemini-specific patterns:

**Structure:**
```markdown
# Gemini-Specific Agent Instructions

**⚠️ Read [AGENTS.md](AGENTS.md) first** for universal project documentation.

This file contains instructions specific to Google Gemini agents.

## Gemini-Specific Configuration
[Currently configured tools and workflows]

## Gemini-Specific Critical Rules
1. Use standard CLI tools (no MCP access)
2. Follow Xcode best practices
3. Test commands: [Standard xcodebuild commands]

## Future Gemini Integrations
[Placeholder for Gemini API, Code Assist, etc.]
```

### Step 5: Update Cross-References

Update AGENTS.md to reference agent-specific docs:

**In automation section:**
```markdown
### 6. Agent-Specific Tools
**Platform**: Varies by agent
**Best For**: Agent-specific automation and integrations

See agent-specific documentation:
- **Claude Code**: [CLAUDE.md](CLAUDE.md) - XcodeBuildMCP, App Store Connect MCP
- **Gemini**: [GEMINI.md](GEMINI.md) - Standard CLI tools
```

**In critical rules:**
```markdown
9. **Follow agent-specific instructions** - See [CLAUDE.md](CLAUDE.md) or [GEMINI.md](GEMINI.md)
```

### Step 6: Verify Organization

Check that the reorganization is complete:

```bash
# Verify all files exist
ls -la AGENTS.md CLAUDE.md GEMINI.md

# Check that AGENTS.md has no agent-specific content
grep -i "mcp" AGENTS.md  # Should only appear in cross-references
grep -i "xcodebuild" AGENTS.md  # Should only appear in universal testing docs

# Check that CLAUDE.md references AGENTS.md
head -5 CLAUDE.md  # Should have "Read AGENTS.md first" warning

# Check that GEMINI.md references AGENTS.md
head -5 GEMINI.md  # Should have "Read AGENTS.md first" warning
```

### Step 7: Commit Changes

Commit the reorganized documentation:

```bash
git add AGENTS.md CLAUDE.md GEMINI.md
git commit -m "docs: Reorganize agentic instructions into agent-specific files

Separated universal project documentation from agent-specific tooling:

- AGENTS.md: Primary source of truth for universal project info
- CLAUDE.md: Claude-specific instructions (MCP servers, build preferences)
- GEMINI.md: Gemini-specific instructions (standard CLI tools)

Benefits:
- Clear separation of concerns (universal vs. agent-specific)
- Single source of truth with agent-specific extensions
- Scalable for adding new AI agents
- No duplication; agent files reference AGENTS.md"

git push origin development
```

## Content Categorization Guide

### Universal Content (AGENTS.md)

**Always universal:**
- Product overview and mission
- Target user persona and pain points
- Architecture patterns and design decisions
- Testing requirements (applies to all agents)
- Git workflow and branching strategy
- Version management and release process
- Documentation index and structure
- Common agent tasks (bug fixes, testing, docs)

**Critical rules (universal):**
- NEVER commit to main directly
- NEVER delete development branch
- ALWAYS test both platforms (if multiplatform)
- NEVER manually edit version numbers
- ALWAYS read files before editing
- NEVER create unnecessary files

### Claude-Specific Content (CLAUDE.md)

**Claude-only:**
- MCP server configuration and usage
- XcodeBuildMCP tool operations
- App Store Connect MCP operations
- Custom MCP servers specific to project
- Build tool preferences (when Claude-specific)
- References to global `~/.claude/CLAUDE.md`

**Critical rules (Claude-specific):**
- ALWAYS use XcodeBuildMCP tools
- NEVER use swift build/test (if using XcodeBuildMCP)
- Leverage MCP servers for automation
- Follow global CLAUDE.md communication patterns

### Gemini-Specific Content (GEMINI.md)

**Gemini-only:**
- Gemini API integration patterns
- Gemini Code Assist workflows
- Standard CLI alternatives to MCP tools
- Gemini-specific build commands

**Critical rules (Gemini-specific):**
- Use standard CLI tools (no MCP access)
- Follow Xcode standard workflows
- Use direct xcodebuild commands

## Examples from Real Projects

### Example 1: Produciesta (Media Production App)

**Before reorganization:**
- AGENTS.md: 643 lines mixing universal and Claude-specific content
- CLAUDE.md: 27 lines redirecting to AGENTS.md
- GEMINI.md: 27 lines redirecting to AGENTS.md

**After reorganization:**
- AGENTS.md: 637 lines of universal content (product, CLI, architecture, testing)
- CLAUDE.md: 135 lines (MCP servers, build preferences, Claude rules)
- GEMINI.md: 44 lines (standard tools, future integrations)

**Extracted from AGENTS.md to CLAUDE.md:**
- Section 6: XcodeBuildMCP Tools (Development Automation)
- Critical Rule #5: ALWAYS use XcodeBuildMCP tools
- Critical Rule #6: NEVER use swift build/test

**Replaced in AGENTS.md:**
- Section 6 now points to agent-specific docs
- Critical rules reduced from 10 to 9 universal rules
- Added new rule: "Follow agent-specific instructions"

### Example 2: Swift Library Project

**AGENTS.md (universal):**
- Library architecture and API design
- Testing requirements (SPM tests)
- Documentation generation (DocC)
- Release process (git tags, GitHub releases)

**CLAUDE.md (Claude-specific):**
- XcodeBuildMCP for swift_package_test
- App Store Connect MCP for TestFlight distribution (if applicable)
- Build preferences (xcodebuild over swift build)

**GEMINI.md (Gemini-specific):**
- Standard swift test commands
- GitHub Actions workflow patterns
- DocC generation with standard tools

## Troubleshooting

### Circular References

**Problem**: AGENTS.md and CLAUDE.md both reference each other, creating confusion.

**Solution**: Always start with AGENTS.md (universal), then extend with agent-specific files:
- AGENTS.md: Does NOT read other files first (it's the foundation)
- CLAUDE.md: MUST start with "Read AGENTS.md first"
- GEMINI.md: MUST start with "Read AGENTS.md first"

### Duplication Between Files

**Problem**: Same content appears in both AGENTS.md and CLAUDE.md.

**Solution**: Apply the "single source of truth" principle:
- If it's universal → AGENTS.md only
- If it's Claude-specific → CLAUDE.md only, reference AGENTS.md for context
- Never duplicate; always reference

### Agent Can't Find Instructions

**Problem**: Agent reads CLAUDE.md but misses universal content in AGENTS.md.

**Solution**: Make the cross-reference explicit and prominent:
- Start CLAUDE.md with: "⚠️ Read [AGENTS.md](AGENTS.md) first"
- Add a summary of what's in AGENTS.md
- Include direct links to relevant AGENTS.md sections

### Too Many Agent Files

**Problem**: Project has 5+ agent-specific files, getting hard to maintain.

**Solution**: Only create agent files when there's substantial agent-specific content:
- If agent only needs 10 lines → Add a section to AGENTS.md instead
- If agent needs 50+ lines → Create dedicated file
- Consider merging rarely-used agent files

## Best Practices

1. **Start with AGENTS.md** - Always establish universal documentation first
2. **Agent files are extensions** - They add to AGENTS.md, not replace it
3. **One source of truth** - Never duplicate content across files
4. **Explicit cross-references** - Make dependencies clear ("Read X first")
5. **Update dates** - Keep "Last Updated" timestamps current
6. **Test organization** - Verify agents can find instructions easily
7. **Review periodically** - Reorganize if universal content ends up in agent files
8. **Scale intentionally** - Only add agent files when needed, not preemptively

## Related Skills

- **skill-creator** - Guide for creating effective skills (meta-skill)
- **sprint-supervisor** - Task execution and sprint management
- **podcast-audio-plan** - Example of comprehensive skill with references

## Notes

- This skill focuses on documentation organization, not content creation
- Assumes project already has some form of agentic documentation
- Works best for projects with 2+ AI agents or planning to scale
- Can be adapted for non-AI documentation (API docs, internal wikis)
- The pattern is based on real-world experience reorganizing Produciesta docs
