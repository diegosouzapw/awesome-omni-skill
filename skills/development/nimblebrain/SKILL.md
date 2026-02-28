---
name: skill-author
description: Creates production-grade Claude Code skills from natural language descriptions. Use when building new skills, requested with "build me a skill that...", "create a skill for...", or "I need a skill to...". Generates complete skill files with proper frontmatter, context references, and quality self-assessment.
metadata:
  version: 1.0.7
  category: development
  tags:
    - skill-creation
    - claude-code
    - automation
    - ai-tools
    - meta
  triggers:
    - build me a skill
    - create a skill for
    - I need a skill to
    - make a new skill
    - write a skill that
    - generate a skill
  surfaces:
    - claude-code
  author:
    name: NimbleBrain
    url: https://www.nimblebrain.ai
---

# Skill Author

Build high-quality Claude Code skills from plain descriptions. Skills are injected into Claude's context when invoked, giving it specialized capabilities for specific tasks.

## Quick Start

When user says "build me a skill that reviews PRs for security issues":

```markdown
---
name: security-pr-reviewer
description: Reviews pull requests for security vulnerabilities and OWASP Top 10 issues. Use when reviewing PRs, checking code for security, or auditing changes before merge. Triggers include "review PR for security", "check this PR", "security audit".

metadata:
  version: 1.0.7
  version: 1.0.0
  category: security
  tags:
    - security
    - code-review
    - pr-review
---

# Security PR Reviewer

[... skill content ...]
```

## Process

### Phase 1: Understand the Request

Extract from the user's description:
- **Core capability**: What does this skill enable?
- **Target domain**: What area/task does it address?
- **Trigger phrases**: When should someone invoke this?
- **Output type**: Dialogue? Document? Validation? Code?

Ask clarifying questions if:
- The domain is ambiguous
- Multiple valid approaches exist
- User intent is unclear

### Phase 2: Determine Skill Type

| Type | Characteristics | Example |
|------|-----------------|---------|
| **Process** | Multi-phase workflow with checkpoints | build-mcp-server, deployment-validator |
| **Dialogue** | Conversational, adaptive to user | strategic-thought-partner |
| **Audit** | Verification against criteria | docs-auditor, security-reviewer |
| **Generator** | Creates artifacts from inputs | changelog-generator |
| **Stance** | Adopts a specific perspective | contrarian-thought-partner |

### Phase 3: Gather Context References

**Critical step**: Skills should reference relevant project context when applicable.

#### Context Discovery Checklist

```
[ ] Is there a CLAUDE.md in the project root?
[ ] Are there existing patterns to reference?
[ ] What files should the skill read when invoked?
[ ] What verification commands exist?
[ ] Are there templates to use?
```

#### Common Context References by Domain

| Domain | Typical References |
|--------|-------------------|
| Code Review | Style guides, linting config, test patterns |
| Documentation | Existing docs structure, templates |
| Security | Security policies, allowed dependencies |
| Testing | Test framework, coverage requirements |
| Deployment | CI/CD config, environment specs |

### Phase 4: Generate the Skill

#### Frontmatter Requirements

```yaml
---
name: kebab-case-name
description: Third person description. Use when [triggers]. Triggers include "phrase 1", "phrase 2".

metadata:
  version: 1.0.7
  version: 1.0.0
  category: development|writing|security|testing|etc
  tags:
    - relevant-tag
  triggers:
    - "trigger phrase"
  author:
    name: Author Name
    url: https://github.com/author
---
```

**Name rules:**
- Gerund form preferred: `reviewing-code`, `building-apis`
- Kebab-case only (lowercase, hyphens)
- Max 64 characters

**Description rules:**
- Third person ("Analyzes..." not "I analyze...")
- Include WHAT it does AND WHEN to use it
- List 2-3 trigger phrases
- Max 1024 characters

#### Section Structure (adapt based on type)

```markdown
# Skill Title

[1-2 sentence overview]

## Quick Start

[Immediate actionable example - code or steps. No preamble.]

## Process / Workflow / Methodology

[Step-by-step with phases or checklist]

## Examples

**Example 1: [Scenario]**
Input: [specific input]
Output: [specific output]

## Best Practices / Techniques

[Guidelines, tips, patterns]

## Anti-Patterns / Common Pitfalls

[What NOT to do]

## References

[Links to project context, templates, docs - if applicable]
```

### Phase 5: Self-Assessment

Before delivering, score the skill:

| Criterion | Points | Check |
|-----------|--------|-------|
| Format compliance | 15 | Valid frontmatter, third-person description |
| Conciseness | 15 | No over-explanation, under ~500 lines |
| Quick Start | 15 | Immediate actionable content |
| Workflow | 15 | Clear steps, checklists for complex flows |
| Examples | 20 | Concrete input/output pairs |
| Completeness | 20 | Edge cases, pitfalls, defaults provided |

**Scoring:**
- 80+: Excellent, ship it
- 60-79: Good, minor improvements possible
- 40-59: Acceptable, suggest improvements
- <40: Needs work, iterate

Report the score and any suggestions.

### Phase 6: File Placement

Skills go in `.claude/skills/<skill-name>/`:

```
.claude/skills/<skill-name>/
├── SKILL.md           # Main skill file (required)
├── templates/         # Optional: Templates used by skill
│   └── *.template
└── examples/          # Optional: Example inputs/outputs
```

## Skill Design Principles

### Be Concise
Claude is smart. Don't explain basics (what APIs are, how loops work). Get to the point.

### Be Specific
Concrete examples beat abstract descriptions. Show real input/output pairs.

### Provide Defaults
Don't offer many options. Give THE recommended approach. Users can deviate if needed.

### Include Context References
Skills that reference project docs are more useful than generic skills.

### Design for Invocation
The skill is injected into context. Write as if Claude will read this and immediately know how to act.

## Output Types by Skill Type

| Skill Type | Output Format |
|------------|---------------|
| Process | Phase-by-phase reports with checkmarks |
| Dialogue | Conversational, adaptive responses |
| Audit | Structured reports with ratings |
| Generator | Artifacts (docs, code, prompts) |
| Stance | Direct, opinionated responses |

## Example: Building a New Skill

**User:** "Build me a skill that writes changelog entries from git commits"

**Phase 1 - Understanding:**
- Core capability: Generate changelogs from commits
- Domain: Release management
- Triggers: "write changelog", "generate release notes"
- Output: Markdown changelog entries

**Phase 2 - Type:** Generator (creates artifacts)

**Phase 3 - Context:**
- Check for existing CHANGELOG.md format
- Reference git conventions
- Look for release patterns in the repo

**Phase 4 - Generate:**

```markdown
---
name: changelog-generator
description: Generates changelog entries from git commit history. Use when preparing releases, writing release notes, or documenting changes. Triggers include "write changelog", "generate release notes", "what changed since".

metadata:
  version: 1.0.7
  version: 1.0.0
  category: development
  tags:
    - changelog
    - releases
    - git
---

# Changelog Generator

Generate well-formatted changelog entries from git commits.

## Quick Start

Given: `git log v1.0.0..HEAD --oneline`

Output:
## [1.1.0] - 2026-01-15

### Added
- User authentication with OAuth support (#123)
- Dark mode toggle in settings (#125)

### Fixed
- Memory leak in WebSocket handler (#124)

### Changed
- Upgraded React to v19 (#126)

## Process

1. **Get commit range**: `git log <from>..<to> --oneline`
2. **Categorize commits** by conventional commit prefix or keywords
3. **Group by type**: Added, Changed, Fixed, Removed, Security
4. **Format entries** with PR/issue links where available
5. **Order by impact**: Breaking changes first, then features, fixes

[... continue ...]
```

**Phase 5 - Score:** 75/100
- Suggestions: Add more examples, include edge cases for merge commits

## Anti-Patterns

| Anti-Pattern | Problem | Instead |
|--------------|---------|---------|
| Over-explaining | Wastes context | Assume Claude knows basics |
| No examples | Too abstract | Show concrete input/output |
| Generic advice | Not actionable | Give specific recommendations |
| Missing context refs | Disconnected from project | Link to relevant files |
| Too many options | Decision paralysis | Provide THE approach |
| First/second person description | Format violation | Use third person |

## Skill Templates

### Process Skill Template

```markdown
---
name: process-name
description: [Third person]. Use when [triggers].

metadata:
  version: 1.0.7
  version: 1.0.0
  category: [category]
---

# [Title]

[Overview]

## Quick Start

[Immediate example]

## Process

### Phase 1: [Name]
[Steps with checklist]

### Phase 2: [Name]
[Steps with checklist]

## Verification

[How to validate results]
```

### Dialogue Skill Template

```markdown
---
name: dialogue-name
description: [Third person]. Use when [triggers].

metadata:
  version: 1.0.7
  version: 1.0.0
  category: [category]
---

# [Title]

[Role description]

## Core Principles

[Stance and approach]

## Session Flow

[How conversations progress]

## Techniques

[Specific dialogue patterns]

## Anti-Patterns

[What not to do]
```

### Audit Skill Template

```markdown
---
name: audit-name
description: [Third person]. Use when [triggers].

metadata:
  version: 1.0.7
  version: 1.0.0
  category: [category]
---

# [Title]

[What this audits and why]

## Audit Criteria

[Checklist of things to verify]

## Methodology

[Step-by-step audit process]

## Output Format

[Structured report template]

## Rating Scale

[How to score/rate findings]
```

