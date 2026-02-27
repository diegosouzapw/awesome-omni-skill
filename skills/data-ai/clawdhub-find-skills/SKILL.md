---
name: clawdhub-find-skills
description: Helps users discover and install agent skills when they ask questions like "how do I do X", "find a skill for X", "is there a skill that can...", or express interest in extending capabilities. Uses reskill as the package manager.
version: 0.4.0
author: reskill
tags:
  - discovery
  - search
  - skills
  - package-manager
---

# Find Skills (reskill)

This skill helps you discover and install skills from the reskill ecosystem.

## Security & Trust

This skill instructs the agent to run CLI commands via [reskill](https://github.com/nicepkg/reskill), an open-source, Git-based package manager for AI agent skills.

**Why a specific registry?**
reskill supports multiple registries. This skill defaults to the Rush community registry (`https://rush.zhenguanyu.com/`) — the primary public registry for the reskill ecosystem, similar to how npm defaults to `https://registry.npmjs.org`. Users can override it at any time via `--registry`, the `RESKILL_REGISTRY` environment variable, or `defaults.publishRegistry` in `skills.json`.

**CLI execution approach:**
To avoid downloading remote code on every invocation, we recommend installing reskill globally first:

```bash
npm install -g reskill
```

If a global installation is not available, `npx reskill@latest` can be used as a fallback. The agent should check for a global install before falling back to npx.

> **Key Principles:**
> 1. **Search → Present → Ask → Install** — always show results first, ask the user before installing.
> 2. **Be registry-aware** — use the configured registry, tell the user which registry you're searching.
> 3. **Prefer local CLI** — use globally installed `reskill` when available; fall back to `npx` only if needed.

## When to Use This Skill

Use this skill when the user:

- Asks "how do I do X" where X might be a common task with an existing skill
- Says "find a skill for X" or "is there a skill for X"
- Asks "can you do X" where X is a specialized capability
- Expresses interest in extending agent capabilities
- Wants to create, write, or publish a skill to a registry
- Wants to search for tools, templates, or workflows
- Mentions they wish they had help with a specific domain (design, testing, deployment, etc.)

### Quick Recommendations

For these well-known intents, **skip the search** and directly recommend the corresponding skill:

| User Intent                                      | Recommended Skill          |
| ------------------------------------------------ | -------------------------- |
| Create, write, or publish a skill to the registry | `@kanyun/rush-reskill-usage` |

If the skill is not already installed, offer to install it. Then proceed with Step 4 (Determine Target Agents) as usual.

## What is reskill?

reskill is a Git-based package manager for AI agent skills. It provides declarative configuration, version locking, and seamless synchronization for managing skills across projects and teams.

**Key commands for skill discovery:**

- `reskill find <query>` — Search for skills by keyword
- `reskill find <query> --json` — Search with machine-readable JSON output
- `reskill install <ref>` — Install a skill
- `reskill list` — List installed skills
- `reskill info <skill>` — Show skill details

## How to Help Users Find Skills

### Step 0: Resolve CLI and Registry

**CLI resolution:** Check if `reskill` is installed globally. If available, use `reskill` directly. Otherwise fall back to `npx reskill@latest`.

```bash
which reskill
```

**Registry resolution** (highest to lowest priority):

1. `--registry <url>` CLI option
2. `RESKILL_REGISTRY` environment variable
3. `defaults.publishRegistry` in `skills.json`
4. Default: `https://rush.zhenguanyu.com/`

If none of the first three are set, pass `--registry https://rush.zhenguanyu.com` explicitly. Tell the user which registry you're searching.

### Step 1: Understand What They Need

When a user asks for help with something, identify:

1. The domain (e.g., React, testing, design, deployment)
2. The specific task (e.g., writing tests, creating animations, reviewing PRs)
3. Whether this is a common enough task that a skill likely exists

### Step 2: Search for Skills (Progressive Strategy)

Use `--json` for structured results. Examples below use `reskill` (substitute `npx reskill@latest` if not globally installed):

```bash
reskill find "<query>" --json --registry https://rush.zhenguanyu.com
```

The JSON output has this structure:

```json
{
  "total": 2,
  "items": [
    {
      "name": "@scope/skill-name",
      "description": "What this skill does",
      "latest_version": "1.0.0",
      "keywords": ["keyword1", "keyword2"],
      "publisher": { "handle": "author" },
      "updated_at": "2025-01-01T00:00:00Z"
    }
  ]
}
```

**IMPORTANT: Use progressive search to maximize results.** The registry may not support multi-word fuzzy matching, so follow this strategy:

#### Round 1: Try the natural query first

```bash
reskill find "frontend design" --json --registry https://rush.zhenguanyu.com
```

If `total > 0`, proceed to Step 3 (present results).

#### Round 2: Try hyphenated version

Skill names often use hyphens. If Round 1 returns 0 results, try connecting keywords with a hyphen:

```bash
reskill find "frontend-design" --json --registry https://rush.zhenguanyu.com
```

#### Round 3: Broaden to the most relevant single keyword

If still 0 results, pick the **most specific keyword** from the user's query and search with that alone:

```bash
reskill find "frontend" --json --registry https://rush.zhenguanyu.com
```

Choose the keyword that best narrows the domain (e.g., prefer "frontend" over "design" because "design" is too broad).

#### Round 4 (optional): Try alternative/synonym keywords

If still no results, try synonyms or related terms:

- "frontend" → "ui", "web", "react"
- "deploy" → "deployment", "ci-cd", "devops"
- "test" → "testing", "jest", "playwright"

#### Agent-side filtering

When broader searches return multiple results, **read each item's `description` field** and filter by relevance to the user's original request. Only present skills whose description genuinely matches what the user needs. Do not present all results blindly.

**Example flow** — user asks "help me with frontend design":

```
1. find "frontend design"    → 0 results
2. find "frontend-design"    → 0 results
3. find "frontend"           → 3 results
4. Read descriptions → filter → 1 result is relevant to UI design
5. Present that 1 result to user
```

**Search query examples:**

| User says                            | Round 1                  | Round 2 (hyphenated)     | Round 3 (single keyword) |
| ------------------------------------ | ------------------------ | ------------------------ | ------------------------ |
| "How do I make my React app faster?" | `"react performance"`    | `"react-performance"`    | `"react"`                |
| "Can you help me with PR reviews?"   | `"pr review"`            | `"pr-review"`            | `"review"`               |
| "I need to create a changelog"       | `"changelog"`            | —                        | —                        |
| "Help me write better TypeScript"    | `"typescript practices"` | `"typescript-practices"` | `"typescript"`           |

Stop as soon as you get relevant results — no need to run all rounds.

### Step 3: Present Results and Ask Before Installing

When you find relevant skills, present them clearly:

1. The skill name and description
2. The version and author
3. Which registry the result came from (public or private)
4. The install command

Then ask the user which one(s) they want to install.

Example response:

```
I found a skill that might help! (from registry: rush.zhenguanyu.com)

**@scope/react-best-practices** (v1.2.0)
React and performance optimization guidelines.

To install:
  reskill install @scope/react-best-practices -y --registry https://rush.zhenguanyu.com

Would you like me to install it?
```

If multiple results are found, present the top 2-3 most relevant ones and let the user choose. Once the user confirms (e.g., "install it", "yes", "install 1 and 3"), proceed to install all confirmed skills — no need to ask again for each one.

### Step 4: Determine Target Agents

Before installing, resolve which agent(s) to install to. Follow this priority:

#### Priority 1: User explicitly specified `--agent`

If the user said something like "install to cursor" or "install to claude-code", pass `-a <agent>` directly — skip all detection.

#### Priority 2: Read `skills.json` → `defaults.targetAgents`

Look for `skills.json` in the current directory and up to 3 parent directories. If found, check for `defaults.targetAgents`:

```json
{
  "defaults": {
    "targetAgents": ["cursor", "claude-code"]
  }
}
```

If `targetAgents` is configured, use those agents directly with `-a`:

```bash
reskill install <name> -y -a cursor claude-code --registry https://rush.zhenguanyu.com
```

#### Priority 3: Detect agent directories

If no `skills.json` is found (or it has no `targetAgents`), scan the current directory and up to 3 parent directories for known agent directories:

| Directory           | Agent        |
| ------------------- | ------------ |
| `.cursor/`          | cursor       |
| `.claude/`          | claude-code  |
| `.codex/`           | codex        |
| `.windsurf/`        | windsurf     |
| `.github/skills/`   | copilot      |
| `.opencode/`        | opencode     |

> **Note:** For GitHub Copilot, check `.github/skills/` (not just `.github/`), since `.github/` alone usually contains workflows/issue templates and does not imply Copilot usage.

If one or more agent directories are detected, **tell the user what was found and confirm before installing**:

```
Detected agent directory: .cursor/
Will install to Cursor (.cursor/skills/). Proceed? (or specify a different agent)
```

If the user confirms, install with `-a`:

```bash
reskill install <name> -y -a cursor --registry https://rush.zhenguanyu.com
```

If multiple agent directories are detected, list all of them and let the user choose which ones to install to.

#### Priority 4: Ask the user

If no agent information is available from any of the above, ask the user which agent(s) to install to:

```
No agent configuration found. Which agent(s) would you like to install this skill to?

Supported agents: cursor, claude-code, codex, windsurf, copilot, opencode
```

Then install with the user's chosen agent(s).

### Step 5: Install the Skill

```bash
# Install to specific agent(s)
reskill install <name> -y -a <agents...> --registry https://rush.zhenguanyu.com

# Install globally (user-level, available in all projects)
reskill install <name> -y -g --registry https://rush.zhenguanyu.com
```

The `-y` flag skips CLI confirmation prompts.

After installation, let the user know the skill is ready and briefly describe what new capabilities it provides.

## Common Skill Categories

When constructing search queries, consider these categories:

| Category        | Example Queries                                    |
| --------------- | -------------------------------------------------- |
| Web Development | react, nextjs, typescript, css, tailwind, vue      |
| Testing         | testing, jest, playwright, e2e, unit-test          |
| DevOps          | deploy, docker, kubernetes, ci-cd, github-actions  |
| Documentation   | docs, readme, changelog, api-docs                  |
| Code Quality    | review, lint, refactor, best-practices, clean-code |
| Design          | ui, ux, design-system, accessibility, figma        |
| Productivity    | workflow, automation, git, project-management      |
| Data            | database, sql, data-analysis, visualization        |
| Skill Dev       | reskill, publish, create-skill, skill-authoring    |

## Tips for Effective Searches

1. **Follow the progressive strategy**: multi-word → hyphenated → single keyword → synonyms
2. **Pick the most specific keyword** when narrowing down: prefer "frontend" over "design", prefer "playwright" over "testing"
3. **Try alternative terms**: "deploy" → "deployment", "ci-cd", "devops"
4. **Always read descriptions**: when a broad search returns many results, use descriptions to filter relevant ones
5. **Skill names use hyphens**: remember to try hyphenated versions like "code-review", "best-practices"

## When No Skills Are Found

If no relevant skills exist **after exhausting all search rounds** (multi-word → hyphenated → single keyword → synonyms):

1. Acknowledge that no existing skill was found and briefly mention what you searched for
2. Offer to help with the task directly using your general capabilities
3. Suggest the user could create their own skill

Example:

```
I searched the registry with several queries ("frontend design", "frontend-design", "frontend")
but didn't find a matching skill.

I can still help you with this task directly! Would you like me to proceed?

If this is something you do often, you could also create your own skill and share it:
  mkdir my-skill && echo "---\nname: my-skill\n---\n# My Skill" > my-skill/SKILL.md
```

## Checking Installed Skills

Before searching for new skills, you can check what's already installed:

```bash
# List all installed skills
reskill list

# Get details about a specific skill
reskill info <skill-name>
```

This avoids suggesting skills the user already has.
