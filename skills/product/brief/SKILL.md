---
name: brief
description: Use when starting a new feature, project, or task - extracts requirements, constraints, non-goals, style preferences, and key concepts from fuzzy input through conversational Q&A. Output is a structured brief saved to .pipeline/brief.md. Always run this before /design.
---

# ARM — Requirements Crystallization

## Role

> **Model:** Opus (`claude-opus-4-6`). If running on Sonnet, output quality for complex reasoning tasks will be reduced.

You are Opus acting as a requirements analyst. Your job is to extract maximum signal from minimum input and produce a brief so precise that /design never needs to ask a clarifying question.

## Process

### Step 1: Detect project context

Before asking anything, silently read the following from the current working directory:
- README.md (if present)
- Any existing `.pipeline/brief.md` (prior brief to build on)
- Primary language files to identify the tech stack (package.json, go.mod, requirements.txt, *.csproj)
- Any CLAUDE.md or project-specific instructions

Record:
- **Primary language(s)** detected (TypeScript, Go, Python, C#, other)
- **LSP tools available**: check which of these are present as tools — `typescript_lsp`, `go_lsp`, `python_lsp`, `csharp_lsp`
- **Existing patterns**: note dominant architectural patterns visible in the codebase

If the project already contains code (non-empty source directories detected above), call `mcp__repomix__pack_codebase` on the current working directory with `compress: false` and `topFilesLength: 20` to get a structured file tree and top-files overview. Use this to:
- Confirm detected tech stack against the actual file structure
- Identify dominant architectural patterns (MVC, layered, feature-based, etc.)
- Inform what questions to ask in Step 2 (what already exists, what's missing, where new code would live)

If the call fails, proceed with the information gathered from reading README.md and config files.

> The outputId is not stored — this is a one-off read for brief context only.

### Step 2: Extract signal through Q&A

Ask ONE question at a time. Wait for the answer before asking the next. Prefer multiple-choice when possible.

Cover these areas in order (skip if already clear from context):

1. **Core purpose** — What does this feature/change do? What problem does it solve?
2. **Users/consumers** — Who calls this? End users, other services, CLI, tests?
3. **Hard constraints** — What MUST be true? (latency, compatibility, existing interfaces)
4. **Soft constraints** — What SHOULD be true but could flex? Flag these explicitly.
5. **Non-goals** — What are you explicitly NOT building?
6. **Success criteria** — How will you know it works? What does done look like?
7. **Style preferences** — Any naming conventions, patterns, or anti-patterns to follow?
8. **Key concepts/domain terms** — Any domain vocabulary that must be used consistently?

### Step 3: Force remaining decisions

After Q&A, present a single structured checkpoint with any remaining ambiguities as forced-choice questions. No open-ended questions in this checkpoint — every item must have options. Do not proceed until the user has answered all items.

Format:
```
CHECKPOINT — Remaining decisions:

1. [Decision]: [Option A] / [Option B] / [Option C]
2. [Decision]: [Option A] / [Option B]
```

### Step 4: Write the brief

Write `.pipeline/brief.md` with this exact structure:

```markdown
# Brief: [Feature Name]

**Date:** [YYYY-MM-DD]
**Primary Language:** [language(s)]
**LSP Available:** [list or "none"]

## Requirements
[Bulleted list of what must be built]

## Constraints
### Hard Constraints (non-negotiable)
- [constraint]

### Soft Constraints (flexible, flagged)
- [constraint] — *soft: [reason it could flex]*

## Non-Goals
[Bulleted list of explicit exclusions]

## Success Criteria
[Bulleted list — measurable, specific]

## Style & Conventions
[Naming, patterns, anti-patterns to follow]

## Key Concepts
[Domain terms and their definitions]

## Open Questions
[Any unresolved items — should be empty after checkpoint]
```

## Output

Confirm to the user: "Brief written to `.pipeline/brief.md`. Run `/design` when ready."
