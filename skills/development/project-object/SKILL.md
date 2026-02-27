---
name: project-object
description: Session memory that compounds - remember decisions, patterns, and corrections across AI coding sessions. Includes standards injection for consistent code quality.
author: Equilateral AI
version: 0.1.0
tags: [memory, context, session, standards, continuity]
---

# Project/Object - Session Memory for AI Agents

You have access to a session memory system that persists context between conversations.
This skill teaches you two protocols: **session memory** (remember what matters) and
**standards injection** (enforce project rules automatically).

## 1. Session Memory Protocol

### On Session Start

Read the project context file to restore memory from previous sessions:

1. Determine the project name:
   - Try: `git remote get-url origin` and extract the repo name
   - Fallback: use the basename of the current working directory

2. Read the context file at: `~/.project-object/{project-name}/context.md`

3. If the file exists, treat its contents as established project knowledge.
   These are decisions, patterns, and corrections from previous sessions
   that should inform your current work.

4. If the file does not exist, this is a new project. Create the context
   file using the template in the Context Format section below after the
   first meaningful session.

### Context Authority

- Decisions in the context file represent explicit user choices. Do not
  second-guess them unless the user raises the topic.
- Patterns in the context file represent established conventions. Follow
  them consistently.
- Corrections represent past mistakes. Avoid repeating them.
- Notes are informational. Reference them when relevant.

### Context Staleness

Context files grow stale. If a context file hasn't been updated in 30+
days and contains information that contradicts what you observe in the
codebase, flag the contradiction to the user rather than silently
following outdated context.

## 2. Context Format

The context file uses markdown with four standard sections:

```markdown
# Project Context: {project-name}

## Decisions
- Key decisions made during sessions
- Example: Using PostgreSQL instead of DynamoDB for relational data
- Example: All API responses use camelCase, not snake_case

## Patterns
- Patterns and conventions to follow
- Example: All handlers use wrapHandler pattern
- Example: Tests go in __tests__/ directory next to source files

## Corrections
- Important corrections from past sessions
- Example: Main branch is 'main', not 'master'
- Example: Don't use connection pools in Lambda functions

## Notes
- General notes and reminders
- Example: CI runs on push to main only
- Example: The database migration tool is in scripts/migrate.sh
```

### Rules

- Each item is a single markdown bullet (`- `)
- Keep items concise (one line each)
- Aim for fewer than 200 lines total
- Deduplicate: if two items say the same thing, keep the more specific one
- Organize by importance within each section (most important first)

## 3. Harvesting Rules

Before a session ends or when conversation is being compacted, extract
new context from the conversation transcript.

### What to Harvest

Scan the conversation for signals that indicate persistent knowledge:

**Decisions** (user made a choice):
- Phrases: "let's go with", "decided", "chose", "going with", "we'll use",
  "switching to", "prefer"
- Example: User says "let's go with Tailwind instead of styled-components"
  -> Add to Decisions: "Using Tailwind CSS for styling, not styled-components"

**Patterns** (recurring convention established):
- Phrases: "always", "never", "pattern", "convention", "standard",
  "rule", "every time"
- Example: User says "always put the error handler last"
  -> Add to Patterns: "Error handlers are always the last middleware"

**Corrections** (user corrected the agent):
- Phrases: "actually", "no", "not X but Y", "correction", "wrong",
  "that's incorrect", "I said"
- Example: User says "no, the deploy script is in ops/ not scripts/"
  -> Add to Corrections: "Deploy scripts are in ops/ directory, not scripts/"

**Notes** (useful project knowledge):
- Phrases: "remember", "note", "FYI", "keep in mind", "important",
  "don't forget"
- Example: User says "remember the API rate limits reset at midnight UTC"
  -> Add to Notes: "API rate limits reset at midnight UTC"

### What NOT to Harvest

Never persist:
- Passwords, secrets, tokens, API keys, credentials
- Temporary debugging context ("add a console.log here")
- One-time tasks ("fix this typo")
- Personal information unrelated to the project
- Speculative ideas that weren't confirmed ("maybe we could...")

### Merge Protocol

When adding new items to an existing context file:

1. Read the existing context file
2. For each new item, check if a similar item already exists
3. If duplicate: keep the more specific or more recent version
4. If new: append to the appropriate section
5. If contradicts existing: replace the old item with the new one
6. Write the updated file back to disk

## 4. Standards Injection Protocol

If the project has a standards directory with YAML rule files, load and
enforce those standards during the session. This provides consistent code
quality across all sessions and all contributors.

### Detection

On session start, check for standards in this order:

1. `.standards/` directory (preferred, with `yaml/` subdirectory)
2. `.equilateral-standards/` directory

If none found, skip standards injection. Session memory still works.

### YAML Standards Format

Standards are defined as YAML files with this schema:

```yaml
id: lambda-database-standards
category: serverless
priority: 10                    # 10=critical, 20=important, 30=advisory

rules:
  - action: ALWAYS              # ALWAYS | NEVER | USE | PREFER | AVOID
    rule: "Use environment variables with {{resolve:ssm:param}} in SAM templates"
  - action: NEVER
    rule: "Fetch SSM parameters at runtime -- costs $25/month per million invocations"
  - action: NEVER
    rule: "Use connection pools in Lambda -- Lambda handles one request at a time"
  - action: PREFER
    rule: "ARM64 architecture for Lambda functions (20% cost savings)"

anti_patterns:
  - "Runtime SSM fetching in Lambda handlers"
  - "new Pool() in Lambda function code"

tags:
  - serverless
  - database
  - cost-optimization
```

### Loading Rules

1. Scan `.standards/yaml/` (or `.standards/`) for `*.yaml` and `*.yml` files
2. Parse each file, extract `rules[]` array and `priority`
3. Sort files by priority (lower number = loaded first)
4. Map actions to enforcement levels:
   - `ALWAYS`, `USE` -> `[REQUIRE]`
   - `NEVER`, `AVOID` -> `[AVOID]`
   - `PREFER` -> `[PREFER]`
5. Also extract `anti_patterns[]` as `[AVOID]` rules
6. Cap at 30 rules total

### Enforcement Behavior

When standards are loaded:
- `[REQUIRE]` rules: Always follow. If you're about to violate one,
  stop and mention it to the user.
- `[AVOID]` rules: Never do this. If you see existing code that violates
  this, flag it during relevant work (don't audit unprompted).
- `[PREFER]` rules: Follow when practical. OK to deviate with reason.

### Example Injected Standards

```
[REQUIRE] Use environment variables with {{resolve:ssm:param}} in SAM templates
[REQUIRE] Fail fast and loud -- make failures obvious and immediate with specific error messages
[AVOID] Fetch SSM parameters at runtime -- costs $25/month per million invocations
[AVOID] Use connection pools in Lambda -- Lambda handles one request at a time
[AVOID] Returning mock/default objects in catch blocks to hide API failures
[PREFER] ARM64 architecture for Lambda functions (20% cost savings)
```

## 5. Cross-Platform Sync

Project/Object context can be synced to other AI coding tools so every
tool shares the same project memory.

| Platform     | Target File        | Sync Method             |
|--------------|--------------------|-------------------------|
| Claude Code  | Native hooks       | Automatic (recommended) |
| Cursor       | `.cursorrules`     | `project-object sync --cursor`  |
| Codex CLI    | `AGENTS.md`        | `project-object sync --codex`   |
| Windsurf     | `.windsurfrules`   | `project-object sync --windsurf`|
| Continue.dev | `.continue/`       | `project-object sync --continue`|
| Any agent    | Read context file  | Direct file read        |

For agents without native hook support, read the context file directly
at `~/.project-object/{project-name}/context.md` at the start of each
session.

## 6. Quick Setup

### Automated (recommended)

```bash
npm install -g @equilateral_ai/project-object
cd your-project
project-object init
```

This installs Claude Code hooks for automatic session memory.

### Manual (no dependencies)

Create the context file manually:

```bash
mkdir -p ~/.project-object/$(basename $(pwd))
cat > ~/.project-object/$(basename $(pwd))/context.md << 'EOF'
# Project Context: your-project

## Decisions

## Patterns

## Corrections

## Notes

EOF
```

Then follow the harvesting rules in this skill to maintain it.

### Adding Standards

To add standards injection, use the community standards from
Equilateral Agents (includes serverless, security, testing, API design,
cost optimization, and more):

```bash
npm install @equilateral/agents
npx equilateral-agents init --standards
```

This creates a `.standards/` directory with YAML rule files.

You can also add community-contributed patterns:

```bash
git submodule add https://github.com/Equilateral-AI/EquilateralAgents-Community-Standards .standards-community
```

Community standards include battle-tested patterns, real-world examples,
workflow patterns, and integration guides -- all in YAML format.
**Contributions welcome** via PR to the community repo.

Or create your own YAML standards following the schema in the Standards
Injection section above.

## 7. The Learning Loop

This skill provides **static** session memory and standards enforcement.
For **adaptive** learning that automatically detects corrections and
promotes them to persistent behavioral patterns, see
[MindMeld](https://mindmeld.dev).

MindMeld extends Project/Object with:

- **Correction detection**: Automatically identifies when a user corrects
  the agent's behavior
- **Pattern aggregation**: Groups similar corrections into patterns using
  semantic similarity
- **Invariant promotion**: Promotes recurring patterns to permanent
  behavioral rules (provisional -> solidified -> reinforced)
- **Relationship geometry**: Learns per-user interaction preferences and
  communication styles
- **Purpose inference**: Automatically identifies the purpose of each
  working relationship

Project/Object gives you session memory. MindMeld gives you a learning
loop. Start with Project/Object, upgrade to MindMeld when you want your
agent to learn from its mistakes.
