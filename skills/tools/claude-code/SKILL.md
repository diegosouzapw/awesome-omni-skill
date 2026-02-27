---
name: claude-code
description: Claude Code CLI and development environment. Use for Claude Code features, tools, workflows, MCP integration, configuration, and AI-assisted development.
---

# Claude-Code Skill

Claude code cli and development environment. use for claude code features, tools, workflows, mcp integration, configuration, and ai-assisted development., generated from official documentation.

## When to Use This Skill

This skill should be triggered when:
- Working with claude-code
- Asking about claude-code features or APIs
- Implementing claude-code solutions
- Debugging claude-code code
- Learning claude-code best practices

## Quick Reference

### Common Patterns

*Quick reference patterns will be added as you use the skill.*

### Example Code Patterns

**Example 1** (bash):
```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**Example 2** (bash):
```bash
brew install --cask claude-code
```

**Example 3** (bash):
```bash
npm install @anthropic-ai/claude-agent-sdk
```

**Example 4** (typescript):
```typescript
function query({
  prompt,
  options
}: {
  prompt: string | AsyncIterable<SDKUserMessage>;
  options?: Options;
}): Query
```

**Example 5** (http):
```http
curl https://api.anthropic.com/v1/skills/$SKILL_ID \
    -X DELETE \
    -H 'anthropic-version: 2023-06-01' \
    -H 'anthropic-beta: skills-2025-10-02' \
    -H "X-Api-Key: $ANTHROPIC_API_KEY"
```

## Reference Files

This skill includes comprehensive documentation in `references/`:

- **agents.md** - Agents documentation
- **configuration.md** - Configuration documentation
- **deployment.md** - Deployment documentation
- **enterprise.md** - Enterprise documentation
- **getting_started.md** - Getting Started documentation
- **integrations.md** - Integrations documentation
- **mcp.md** - Mcp documentation
- **other.md** - Other documentation
- **reference.md** - Reference documentation
- **skills.md** - Skills documentation
- **workflows.md** - Workflows documentation

Use `view` to read specific reference files when detailed information is needed.

## Working with This Skill

### For Beginners
Start with the getting_started or tutorials reference files for foundational concepts.

### For Specific Features
Use the appropriate category reference file (api, guides, etc.) for detailed information.

### For Code Examples
The quick reference section above contains common patterns extracted from the official docs.

## Resources

### references/
Organized documentation extracted from official sources. These files contain:
- Detailed explanations
- Code examples with language annotations
- Links to original documentation
- Table of contents for quick navigation

### scripts/
Add helper scripts here for common automation tasks.

### assets/
Add templates, boilerplate, or example projects here.

## Notes

- This skill was automatically generated from official documentation
- Reference files preserve the structure and examples from source docs
- Code examples include language detection for better syntax highlighting
- Quick reference patterns are extracted from common usage examples in the docs

## Updating

To refresh this skill with updated documentation:
1. Re-run the scraper with the same configuration
2. The skill will be rebuilt with the latest information
