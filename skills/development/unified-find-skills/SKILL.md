---
name: unified-find-skills
description: Helps users discover and install agent skills from skills.sh, clawhub.com, and tessl.io. Use when the user asks to find a skill for a task, extend agent capabilities, or search for tools/workflows.
---

# Find Skills

This skill helps you discover and install skills from three registries:
- **skills.sh** - The original open agent skills ecosystem
- **clawhub.com** - Vector-based skill search with simple slugs (requires `clawhub` CLI)
- **tessl.io** - Registry with versioned skills and tiles

## When to Use This Skill

Use this skill when the user:
- Asks "how do I do X" where X might be a common task with an existing skill
- Says "find a skill for X" or "is there a skill for X"
- Asks "can you do X" where X is a specialized capability
- Expresses interest in extending agent capabilities
- Wants to search for tools, templates, or workflows
- Mentions they wish they had help with a specific domain (design, testing, deployment, etc.)

## Search Available Registries

Search all **available** registries. If `clawhub` CLI is not installed, skip that registry.

### Step 1: Understand What They Need

When a user asks for help with something, identify:
1. The domain (e.g., React, testing, design, deployment)
2. The specific task (e.g., writing tests, creating animations, reviewing PRs)
3. Whether this is a common enough task that a skill likely exists

### Step 2: Search Available Registries

Check which CLIs are available and search in parallel:

```bash
# skills.sh (always available via npx)
npx skills find [query] --limit 5

# clawhub (only if installed)
if command -v clawhub &> /dev/null; then
  clawhub search "[query]" --limit 5
fi

# tessl.io (via web scraping)
curl -s "https://tessl.io/registry/discover?contentType=skills" | grep -o 'name:"[^"]*"' | head -10
```

For example:
- User asks "how do I make my React app faster?" → search available registries for "react performance"
- User asks "can you help me with PR reviews?" → search available registries for "pr review"
- User asks "I need to create a changelog" → search available registries for "changelog"

**Note on clawhub:** Requires `clawhub` CLI installed. Install with `npm install -g clawhub` if not available.

**Note on tessl.io:** The tessl registry doesn't have a simple CLI search command. You can:
- Browse at https://tessl.io/registry/discover?contentType=skills
- Extract skill names from the page using curl + grep
- Use `tessl skill search [query]` (interactive mode only)

### Step 3: Present Options to the User

When you find relevant skills, present them organized by registry with:

**For skills.sh results:**
1. The skill name and what it does
2. The install command they can run
3. A link to learn more at skills.sh

**For clawhub results:**
1. The skill slug and version
2. Description if available
3. The install command they can run

**For tessl.io results:**
1. The skill name
2. Description if available (from the registry page)
3. The install command they can run

Example response:
```
I found some skills that might help!

**From skills.sh:**
- "vercel-react-best-practices" - React and Next.js performance optimization guidelines from Vercel Engineering
  Install: npx skills add vercel-labs/agent-skills@vercel-react-best-practices
  Learn more: https://skills.sh/vercel-labs/agent-skills/vercel-react-best-practices

**From clawhub:**
- "react-expert v0.1.0" - React Expert
  Install: clawhub install react-expert

**From tessl.io:**
- "react-doctor" - Diagnose and fix React codebase health issues
  Browse: https://tessl.io/registry/discover?contentType=skills
  Install: tessl install <skill-name> (requires tessl CLI)
```

### Step 4: Offer to Install

If the user wants to proceed with a skill:

**For skills.sh skills:**
```bash
npx skills add <owner/repo@skill> -g -y
```

The `-g` flag installs globally (user-level) and `-y` skips confirmation prompts.

**For clawhub skills:**
```bash
clawhub install <slug>
```

Optionally specify version:
```bash
clawhub install <slug> --version <version>
```

**For tessl.io skills:**
```bash
tessl install <skill-name>
```

Install from GitHub:
```bash
tessl install github:user/repo
```

## Registry Comparison

| Feature        | skills.sh                          | clawhub.com                      | tessl.io                         |
| --------------- | ----------------------------------- | --------------------------------- | --------------------------------- |
| Search format  | `npx skills find <query>`          | `clawhub search "<query>"`        | Browse web or `tessl skill search` |
| Install format | `npx skills add <owner/repo@skill>`| `clawhub install <slug>`         | `tessl install <skill-name>`      |
| Versioning     | Git-based (owner/repo@skill)       | Semantic versioning (vX.Y.Z)       | Semantic versioning                 |
| Browse at      | https://skills.sh/                 | https://clawhub.ai/              | https://tessl.io/registry/discover |
| CLI required?  | No (npx)                          | Yes (`clawhub`)                   | Optional (`tessl`)                |
| Updates        | `npx skills update`                | `clawhub update <slug>` or `--all`| `tessl update`                   |

## Common Skill Categories

When searching, consider these common categories:

| Category        | Example Queries                          |
| --------------- | ---------------------------------------- |
| Web Development | react, nextjs, typescript, css, tailwind |
| Testing         | testing, jest, playwright, e2e           |
| DevOps          | deploy, docker, kubernetes, ci-cd        |
| Documentation   | docs, readme, changelog, api-docs        |
| Code Quality    | review, lint, refactor, best-practices   |
| Design          | ui, ux, design-system, accessibility     |
| Productivity    | workflow, automation, git                |

## Tips for Effective Searches

1. **Search all available registries** - Each has unique skills
2. **Use specific keywords**: "react testing" is better than just "testing"
3. **Try alternative terms**: If "deploy" doesn't work, try "deployment" or "ci-cd"
4. **Check popular sources**: Many skills.sh skills come from `vercel-labs/agent-skills` or `ComposioHQ/awesome-claude-skills`
5. **For tessl.io**: Browse the web interface since CLI search is interactive-only
6. **For clawhub**: Install CLI first with `npm install -g clawhub` if not available

## When No Skills Are Found

If no relevant skills exist in any available registry:

1. Acknowledge that no existing skill was found
2. Offer to help with the task directly using your general capabilities
3. Suggest the user could create their own skill

Example:
```
I searched all available registries for skills related to "xyz" but didn't find any matches.
I can still help you with this task directly! Would you like me to proceed?

If this is something you do often, you could create your own skill:
- With skills.sh: npx skills init my-xyz-skill
- With tessl.io: tessl skill new --name "My X Skill" --description "..."
```

## Installing Missing CLIs

If a user wants to use clawhub but doesn't have it installed:

```bash
npm install -g clawhub
```

For tessl.io:

```bash
npm install -g tessl
```
