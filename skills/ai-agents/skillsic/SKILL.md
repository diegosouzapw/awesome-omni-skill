---
name: skillsic
description: Discover, search, and install curated AI agent skills via skillsic.com. Use when you need to find a skill for a specific task, browse skills by category, or see AI-analyzed ratings before installing. Supports both web browsing and programmatic access via MCP.
license: MIT
compatibility: Designed for Claude Code and other skills-compatible agents. Requires internet access.
metadata:
  author: skillsic
  version: "1.0"
---

# skillsic

Discover, search, and install curated AI agent skills via skillsic.com.

## Overview

skillsic is a curated skill discovery platform for AI agents. It aggregates skills from the ecosystem, analyzes them using Claude 4.5, and provides AI-powered ratings and recommendations. The platform is hosted on the Internet Computer (ICP) for decentralized reliability.

## When to Use This Skill

Use skillsic when:
- You need to find a skill for a specific task (e.g., "I need help with Rust", "How do I deploy to Vercel?")
- You want to discover new capabilities for your agent
- You want to see AI-analyzed ratings before installing a skill
- You need to browse skills by category

## How to Search for Skills

### Via Website
Visit **https://skillsic.com** to browse, search, and explore curated skills with detailed AI analysis.

### Via MCP (Programmatic)
If you have the `@skillsic/mcp` server configured, use these tools:

1. **search_skills** - Search by keyword
   ```
   search_skills({ query: "rust programming" })
   ```

2. **get_skill** - Get detailed info about a skill
   ```
   get_skill({ id: "owner/repo" })
   ```

3. **get_top_skills** - Get highest rated skills
   ```
   get_top_skills({ limit: 10 })
   ```

4. **get_skills_by_category** - Browse by category
   ```
   get_skills_by_category({ category: "web" })
   ```

5. **get_categories** - List all categories
   ```
   get_categories()
   ```

## How to Install Skills

All skills are installed via the `skills.sh` CLI:

```bash
npx skills add <owner>/<skill-name>
```

Examples:
```bash
npx skills add vercel-labs/agent-skills
npx skills add anthropics/skills
npx skills add skillsic/skillsic
```

## Understanding Skill Ratings

skillsic analyzes each skill using Claude 4.5 and provides:

- **Overall Rating (0-5)**: Weighted average across 13 topics
- **13-Topic Breakdown**: Quality, Documentation, Maintainability, Completeness, Security, Malicious (safety), Privacy, Usability, Compatibility, Performance, Trustworthiness, Maintenance, Community
- **Flags**: Security risks, malicious patterns, privacy concerns with severity levels
- **Summary**: Brief description of what the skill does
- **Strengths/Weaknesses**: Detailed analysis
- **Use Cases**: When to use this skill
- **Dependencies**: MCP and software dependencies with their own ratings

## Available Categories

- `web` - Web development, frameworks, deployment
- `programming` - Language-specific skills (Rust, Python, etc.)
- `systems` - Systems programming, CLI tools
- `blockchain` - Web3, ICP, smart contracts
- `ai` - AI/ML tools and patterns
- `development` - General development practices
- `deployment` - CI/CD, hosting, infrastructure
- `meta` - Meta-skills and discovery tools

## Example Workflows

### Finding a Skill for a Task

User: "I need to build a Next.js app and deploy it"

1. Search for relevant skills:
   - Search "nextjs deployment" on skillsic.com
   - Or use MCP: `search_skills({ query: "nextjs deployment" })`

2. Review the results and ratings

3. Install the recommended skill:
   ```bash
   npx skills add vercel-labs/agent-skills
   ```

### Exploring Top Skills

1. Visit skillsic.com and sort by rating
2. Or use MCP: `get_top_skills({ limit: 10 })`
3. Read the AI analysis for each skill
4. Install skills that match your needs

## MCP Server Setup

To enable programmatic skill discovery, add skillsic to your agent's MCP config:

```json
{
  "mcpServers": {
    "skillsic": {
      "command": "npx",
      "args": ["@skillsic/mcp"]
    }
  }
}
```

## Links

- Website: https://skillsic.com
- MCP Package: https://www.npmjs.com/package/@skillsic/mcp

---

*skillsic is hosted on the Internet Computer (ICP) for decentralized reliability.*
