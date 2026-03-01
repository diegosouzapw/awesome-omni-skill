---
name: reviewing-documentation
description: Use when reviewing documentation quality, auditing README or CLAUDE.md files, or standardizing AI instruction files.
---

# Documentation Review Skill

## Documentation Categories

6 categories: Accuracy (Critical), Clarity (Major), Completeness (Major), Consistency (Minor), Examples (Critical), Structure (Major). See docs agent files for checklists.

## AI Instruction File Standardization

Checks placement and cross-references for `.ai/AI-AGENT-INSTRUCTIONS.md`, `CLAUDE.md`, `.github/copilot-instructions.md`. See `references/ai-instruction-templates.md`.

## Scope Prioritization

- README.md (project entry point)
- CLAUDE.md (AI assistant instructions)
- docs/getting-started.md or similar
- API documentation
- AI instruction files

## False Positives

**Documentation-specific** - do NOT flag:
- Intentionally simplified examples (marked as such)
- Placeholder values (`your-api-key`, `example.com`)
- Version-specific docs with version clearly noted
- Pseudocode marked as illustrative

## Pattern References

- `references/ai-instruction-templates.md` - AI instruction templates

### README.md Required Sections

1. **Title/Description** - name, one-paragraph description, key features
2. **Installation** - prerequisites with versions, step-by-step, verification
3. **Quick Start** - minimal working example, expected output
4. **Configuration** - required env vars, config format, defaults
5. **API/CLI Reference** - commands/methods, parameters, return values
6. **Contributing** - bug reports, PR process, code style
7. **License**

**Anti-patterns**: assuming global deps, non-working examples, missing prerequisites, outdated screenshots

### Severity Calibration

Missing Install section: Critical (blocks users). Missing error responses in API docs: Major. Missing Contributing: Suggestion (not blocking).

### API Documentation

Required per endpoint: all params with types, all response codes, at least one example, error responses.

### Consistency Patterns

One term per concept (not "config/conf/settings"). Consistent heading case, list punctuation, code language tags, link style throughout.
