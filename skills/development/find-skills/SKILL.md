---
name: find-skills
description:
  Helps users discover and install agent skills when they ask questions like
  "how do I do X", "find a skill for X", "is there a skill that can...", or
  express interest in extending capabilities. This skill should be used when the
  user is looking for functionality that might exist as an installable skill.
---

<!-- prettier-ignore-start -->
**Document Version:** 1.0
**Last Updated:** 2026-02-08
**Status:** ACTIVE
<!-- prettier-ignore-end -->

# Find Skills

This skill helps you discover and install skills and plugins from both the
skills.sh ecosystem and Claude Code plugin marketplaces.

## When to Use

- Tasks related to find-skills
- User explicitly invokes `/find-skills`

## When NOT to Use

- When the task doesn't match this skill's scope -- check related skills
- When a more specialized skill exists for the specific task

## When to Use This Skill

Use this skill when the user:

- Asks "how do I do X" where X might be a common task with an existing skill
- Says "find a skill for X" or "is there a skill for X"
- Asks "can you do X" where X is a specialized capability
- Expresses interest in extending agent capabilities
- Wants to search for tools, templates, or workflows
- Mentions they wish they had help with a specific domain (design, testing,
  deployment, etc.)
- Is starting a new project or working in an unfamiliar domain

## How to Help Users Find Skills

### Step 1: Understand What They Need

When a user asks for help with something, identify:

1. The domain (e.g., React, testing, design, deployment)
2. The specific task (e.g., writing tests, creating animations, reviewing PRs)
3. Whether this is a common enough task that a skill likely exists

### Step 2: Run the Unified Search

Run the unified capabilities search that covers both ecosystems:

```bash
node scripts/search-capabilities.js [query keywords]
```

For example:

- User asks "how do I make my React app faster?" →
  `node scripts/search-capabilities.js react performance`
- User asks "can you help me with PR reviews?" →
  `node scripts/search-capabilities.js pr review`
- User asks "I need to create a changelog" →
  `node scripts/search-capabilities.js changelog`

The search returns results grouped by:

- **INSTALLED** — local skills and installed plugins matching the query
- **AVAILABLE IN MARKETPLACES** — plugins from 6 marketplace registries
- **AVAILABLE ON SKILLS.SH** — skills from the open skills.sh ecosystem

### Step 3: Present Options to the User

When you find relevant results, present them organized by status:

1. Already installed capabilities they can use right now
2. Available marketplace plugins with install commands
3. Available skills.sh packages with install commands

### Step 4: Offer to Install

**For marketplace plugins:**

```bash
claude plugin install <name>@<marketplace>
```

**For skills.sh packages:**

```bash
npx skills add <owner/repo@skill> -g -y
```

The `-g` flag installs globally (user-level) and `-y` skips confirmation
prompts.

## Fallback: Direct skills.sh Search

If the unified search doesn't find what you need, you can also search skills.sh
directly for broader results:

```bash
npx skills find [query]
```

**Browse skills at:** https://skills.sh/

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
| AI/LLM          | llm, rag, embedding, prompt              |
| Backend         | api, microservices, database, graphql    |
| Security        | security, auth, audit, compliance        |

## Tips for Effective Searches

1. **Use specific keywords**: "react testing" is better than just "testing"
2. **Try alternative terms**: If "deploy" doesn't work, try "deployment" or
   "ci-cd"
3. **Search by domain**: Use broad domain terms to see all related capabilities
4. **Check installed first**: The search shows installed capabilities — use
   those before installing new ones

## When No Skills Are Found

If no relevant skills exist:

1. Acknowledge that no existing skill was found
2. Offer to help with the task directly using your general capabilities
3. Suggest the user could create their own skill with `npx skills init`

Example:

```
I searched for skills related to "xyz" but didn't find any matches in either
the plugin marketplaces or skills.sh.

I can still help you with this task directly! Would you like me to proceed?

If this is something you do often, you could create your own skill:
npx skills init my-xyz-skill
```

---

## Version History

| Version | Date       | Description            |
| ------- | ---------- | ---------------------- |
| 1.0     | 2026-02-25 | Initial implementation |
