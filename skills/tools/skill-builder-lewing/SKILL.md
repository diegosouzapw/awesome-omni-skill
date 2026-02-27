---
name: skill-builder
description: Build, test, and deploy Copilot CLI skills. Use when creating a new skill, improving an existing skill, testing skills with subagents, deploying skills to a repo, or learning skill development patterns.
---

# Skill Builder

Guide for building Copilot CLI skills — from inception through deployment and iteration. This is a pure knowledge skill: you orchestrate everything using your existing tools (`create`, `edit`, `task`, `powershell`, `gh` CLI). No wrapper scripts needed.

## Source of Truth

If this file and any reference doc conflict, **this file wins**.

Precedence: `SKILL.md` > `references/*.md` > agent general knowledge.

## Core Principle: The Context Window is a Public Good

Skills share the context window with system prompts, conversation history, other skills, and the user's actual work. Every token in a skill is a token the agent can't use for reasoning.

**Default assumption: the agent is already very smart.** Only include context it doesn't already have. Challenge each paragraph: "Does this justify its token cost?" Prefer concise examples over verbose explanations.

## When to Use This Skill

Use this skill when:
- Asked to "build a skill", "create a copilot skill", or "make a skill"
- Improving an existing skill's structure, output, or documentation
- Testing a skill using multi-model subagent methodology
- Deploying a skill to a repository
- Learning skill development patterns and best practices

### Skill vs Other Configuration

| Need | Use | Not a skill |
|------|-----|------------|
| Reusable investigation/automation with triggers | **Skill** (.github/skills/) | |
| Repo-wide conventions for every prompt | | **copilot-instructions.md** |
| Path-specific rules ("all files in src/api/ must...") | | **.instructions.md** |
| Agent with identity, tools, autonomy | | **.agent.md** |

If unsure: try copilot-instructions.md first. Create a skill when you need trigger-based activation, references loaded on demand, or scripts.

## Process: Building a New Skill

### Step 1: Understand Requirements

**Evaluate first**: Before writing anything, test the agent on your use case *without* a skill. Note where it fails — those gaps are what your skill should fix. See [references/skill-lifecycle.md](references/skill-lifecycle.md).

Ask the user:
1. **What does the skill do?** (investigation, review, knowledge capture, automation)
2. **What triggers it?** (keywords, URLs, scenarios users describe)
3. **Which archetype?**

| Signal | Archetype |
|--------|-----------|
| Needs to run scripts, call APIs, process data | **Script-driven** |
| Teaches conventions, review patterns, domain rules | **Knowledge-driven** |
| Both | Start knowledge-driven, add scripts only for what you can't do natively |

> ❌ **NEVER** write a script that wraps tools you already have (create, edit, gh, powershell). If you're tempted, you're over-scripting.

> ❌ **NEVER** encode reasoning into scripts. Scripts collect data; agents reason over it. If a script has `if/elseif` chains producing prose recommendations, move that logic to SKILL.md guidance and have the script emit structured JSON instead. See [references/skill-patterns.md](references/skill-patterns.md#data-vs-reasoning-boundary).

### Step 2: Create Directory Structure

```powershell
# User-level skill
New-Item -ItemType Directory -Path ~/.copilot/skills/{name}/references -Force

# Or repo-level skill
New-Item -ItemType Directory -Path .github/skills/{name}/references -Force
# Add scripts/ only if script-driven
# Add assets/ only if the skill produces files (templates, schemas, etc.)
```

### Step 3: Write SKILL.md

Use the template from [references/skill-patterns.md](references/skill-patterns.md) matching the archetype. Key requirements:

- **Frontmatter**: `name` + structured `description` with `USE FOR` / `DO NOT USE FOR` / `INVOKES` routing signals (see [references/skill-patterns.md](references/skill-patterns.md#structured-description-pattern))
- **When to Use**: 5-8 concrete trigger scenarios
- **Stop signals**: Explicit bounds on when to stop (most impactful single addition — saved 10+ tool calls in Arena evals). See [references/skill-patterns.md](references/skill-patterns.md#stop-signals).
- **Inline anti-patterns**: Embed the 3-5 most critical mistakes near the steps where they'd occur

> ⚠️ **Frontmatter fields**: `name` and `description` are required. `argument-hint`, `user-invokable`, and `disable-model-invocation` are supported. Other fields (`license`, `version`) may be silently ignored or cause errors — avoid them. See [references/skill-patterns.md](references/skill-patterns.md#frontmatter-required) for details.

> ❌ **NEVER** restate MCP tool parameter schemas or chain tool calls into rigid step-by-step recipes — the agent has tool descriptions in its context. DO provide examples that add domain context the tool description lacks (branch ref patterns, field names, log locations). See [references/anti-patterns.md](references/anti-patterns.md#re-documenting-mcp-tools).

> ⚠️ **Context budget**: An orchestrating SKILL.md should be 2K-4K tokens. A knowledge-only SKILL.md applied once per task can be larger (up to 15K tokens). Move depth to `references/`.

### Step 4: Write Reference Docs

Create `references/*.md` for deep content the agent loads on demand:
- Domain concepts and terminology
- Detailed patterns with examples
- Troubleshooting guides
- Anti-pattern catalogs

### Step 5: Write Scripts (Script-driven Only)

Follow conventions from [references/skill-patterns.md](references/skill-patterns.md):
- Naming: `Get-{DomainAction}.ps1`
- Standard param block with defaults
- Write-Section helper for consistent output
- Emoji status: ✅ green / ⚠️ yellow / 🔴 red
- **Fail-closed error handling** — Unknown ≠ Healthy

> ❌ **NEVER** count API failures as success. Return "Unknown" and exclude from positive counts.

### Step 6: Test with Multi-Model Subagents

Follow [references/testing-patterns.md](references/testing-patterns.md):

1. Select top-tier model from 2-4 different families
2. Give each the same test prompt exercising the skill
3. Launch in parallel via `task` tool with `model` parameter
4. Synthesize: consensus findings = high confidence
5. Fix errors first, then warnings, then consider suggestions
6. **Retrospective**: When an agent misapplies guidance, ask the *same model* why it made that choice — its self-analysis reveals guidance gaps you can close with targeted anti-patterns (see [references/skill-lifecycle.md](references/skill-lifecycle.md))
7. **A/B test**: After fixing issues, re-run the same task to verify improvement — same model, same prompt, compare correctness/speed/tool calls (see [references/testing-patterns.md](references/testing-patterns.md))

**For new skills or major restructuring**, use the writer-critic convergence loop instead: one agent writes, a different-model agent critiques, writer applies fixes, repeat until convergence (2-3 rounds). See [references/testing-patterns.md](references/testing-patterns.md#writer-critic-convergence-loop).

### Step 7: Deploy

**Local first**: Copy to `~/.copilot/skills/{name}/` and test in a real conversation.

**Then to repo**:
```powershell
git checkout -b add-{name}-skill
# Copy files to .github/skills/{name}/
git add .github/skills/ && git commit -m "Add {name} skill"
git push origin HEAD && gh pr create --title "Add {name} skill"
```

**Optional**: Wire into `.github/copilot-instructions.md` for automatic invocation.

### Step 8: Handle Review

See [references/testing-patterns.md](references/testing-patterns.md) for the review thread workflow. Key points:
- Verify every automated reviewer claim before accepting
- Reply with evidence when pushing back
- Resolve threads via GraphQL after addressing

> ⚠️ **Automated reviewers have ~30-50% false positive rates.** Don't accept suggestions uncritically. Verify each claim.

## Quick Start Recipes

### Create an investigation skill (script-driven)
"I keep running the same API calls to diagnose X"
→ Step 1-7 above with script-driven archetype. Script handles API calls + data correlation. SKILL.md documents when to use, parameters, and how to interpret results.

### Create a knowledge/review skill (knowledge-driven)
"I keep explaining the same conventions/patterns to agents"
→ Mine real review comments or docs for patterns. Organize by category with severity (❌/⚠️/💡). Include real quotes as evidence. The SKILL.md IS the skill — no scripts.

### Test an existing skill
→ Jump to Step 6. Give subagents a realistic task and collect feedback.

### Deploy a skill to a repo
→ Jump to Step 7. Create branch, copy files, PR, handle review.

### Build a custom agent
→ See [references/agent-conventions.md](references/agent-conventions.md). Agents are flat `.agent.md` files — simpler than skills but with cross-platform nuances (tools field, subagent spawning, companion skill pattern for reference material).

## Troubleshooting

### Skill not triggering
- **Check description keywords**: The `description` in frontmatter must match what users actually say. Use trigger phrases, not technical jargon.
- **Verify location**: User skills go in `~/.copilot/skills/{name}/`. Repo skills go in `.github/skills/{name}/`.
- **SKILL.md must exist**: The file must be named exactly `SKILL.md` (case-sensitive on Linux).

### Where to put a skill

| Audience | Location | Notes |
|----------|----------|-------|
| Just you | `~/.copilot/skills/` | Available in all repos. Also: `~/.agents/skills/` |
| Your team/repo | `.github/skills/` | Available to anyone working in that repo. Also: `.agents/skills/` |
| Multiple repos | User-level, then copy to repos as needed | Or publish as a shared reference |

> 💡 Use `.copilot/` / `.github/` as primary. The `.agents/` paths provide cross-agent compatibility.

### Large knowledge domains
When a knowledge-driven skill grows beyond ~8K tokens in SKILL.md:
- Split domain rules into `references/` files by category
- Keep SKILL.md as the process/orchestration hub
- Agent loads references on demand — they don't all need to fit in context at once

### Periodic review
The skill ecosystem is evolving. Periodically re-check the official docs for new supported frontmatter fields, skill discovery improvements, and best practices:
- [GitHub: About Agent Skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills)
- [VS Code: Agent Skills](https://code.visualstudio.com/docs/copilot/customization/agent-skills)
- [GitHub: Custom Instructions](https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot)
- [agentskills.io spec](https://agentskills.io) — open standard defining skill format (name/description constraints)
- [Skills, Tools & MCP Development Guide](https://aka.ms/skills/guidance) — comprehensive patterns for skill classification, routing, evaluation

## References

- **Structural patterns & templates**: [references/skill-patterns.md](references/skill-patterns.md)
- **Development lifecycle**: [references/skill-lifecycle.md](references/skill-lifecycle.md)
- **Multi-model testing**: [references/testing-patterns.md](references/testing-patterns.md)
- **Empirical anti-patterns**: [references/anti-patterns.md](references/anti-patterns.md)
- **Agent conventions**: [references/agent-conventions.md](references/agent-conventions.md)
