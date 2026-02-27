---
name: find-skills
description: Find and install new capabilities/skills for the agent to use.
---

# Find Skills

Use this skill to discover and install other skills that can help you complete tasks.
Skills are reusable instruction sets (like this one) that extend your capabilities.

## 1. How to Find Skills

When the user asks for new capabilities or you need to perform a specialized task (e.g., "deploy to Vercel", "write unit tests", "generate documentation"), search for existing skills.

**Search Strategy:**
- Use the `search_web` tool.
- Queries: `site:skills.sh [topic]`, `vercel agent skills [topic]`, `github agent skills [topic]`.

## 2. Common Skill Categories

| Category | Example Queries |
| :--- | :--- |
| **Web Dev** | react, nextjs, typescript, tailwind |
| **Testing** | testing, jest, playwright, e2e |
| **DevOps** | deploy, docker, kubernetes, ci-cd |
| **Docs** | docs, readme, changelog |
| **Code Quality** | review, lint, refactor |

## 3. How to Install a Skill

Once you find a relevant skill (e.g., `owner/repo@skill-name`), offer to install it for the user using the following command:

```bash
npx skills add owner/repo@skill-name -g -y
```

- `-g`: Installs globally (user-level), making it available across projects.
- `-y`: Skips confirmation prompts.

## 4. Response Template

If you find a skill:
> I found a skill that might help! **[Skill Name]** helps with [description].
>
> To install it, run:
> ```bash
> npx skills add [owner]/[repo]@[skill] -g -y
> ```
> [Link to skill page if available]

If no skill is found:
> I couldn't find a specific skill for [topic]. I can try to help you directly using my general knowledge, or you can create a custom skill using `npx skills init`.
