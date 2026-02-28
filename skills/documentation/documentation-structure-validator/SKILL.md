---
name: documentation-structure-validator
description: Validate documentation structure, check for missing sections, verify markdown syntax, ensure consistency with templates. Use when user mentions docs validation, structure check, README review, documentation quality, or wants to verify documentation completeness.
allowed-tools: Read, Grep, Glob
---

# Documentation Structure Validator

Validate documentation files for completeness, structure, and consistency.

## Instructions

1. Identify the documentation type (README, ADR, API docs, guide)
2. Load the appropriate template from [templates.md](templates.md)
3. Check for required sections and common issues
4. Review markdown syntax and formatting
5. Generate a validation report with:
   - ✅ Passed checks
   - ⚠️ Warnings
   - ❌ Critical issues
   - 💡 Recommendations

## Validation Checklist

### README Files
- [ ] Title/project name present
- [ ] Description/purpose clear
- [ ] Installation instructions
- [ ] Usage examples
- [ ] Configuration section (if applicable)
- [ ] Contributing guidelines link
- [ ] License information
- [ ] Table of contents (for long READMEs)
- [ ] Badges (optional but recommended)

### Architecture Decision Records (ADR)
- [ ] Title with number (ADR-###)
- [ ] Status (proposed/accepted/deprecated/superseded)
- [ ] Context section
- [ ] Decision section
- [ ] Consequences section
- [ ] Date
- [ ] References/links

### API Documentation
- [ ] Endpoint descriptions
- [ ] Request/response examples
- [ ] Authentication details
- [ ] Error codes documentation
- [ ] Rate limiting info
- [ ] Versioning strategy

## Common Issues

Refer to [validation-rules.md](validation-rules.md) for detailed rules and examples.

## Usage Examples

### Example 1: Validate README
```
User: "Check if the README is complete"
Claude: *reads README.md, compares against template, generates report*
```

### Example 2: Validate ADR
```
User: "Validate the architecture decision docs in /docs/decisions/"
Claude: *validates each ADR file against ADR template*
```

### Example 3: Batch validation
```
User: "Check all markdown files in /docs/"
Claude: *scans all .md files, validates each based on type*
```

## Output Format

```markdown
# Documentation Validation Report
**File**: path/to/file.md
**Type**: README
**Date**: YYYY-MM-DD

## Summary
- Total checks: 15
- Passed: 12 ✅
- Warnings: 2 ⚠️
- Critical: 1 ❌

## Critical Issues ❌
1. Missing installation instructions

## Warnings ⚠️
1. No contributing guidelines link
2. Code examples not syntax-highlighted

## Passed ✅
- Title present
- Description clear
- Usage examples included
...

## Recommendations 💡
1. Add badges for build status and license
2. Include table of contents for easier navigation
3. Add troubleshooting section
```

## Tips

- For templates, see [templates.md](templates.md)
- For validation rules, see [validation-rules.md](validation-rules.md)
- Always provide actionable feedback
- Prioritize critical issues over style preferences
- Link to examples when suggesting improvements
