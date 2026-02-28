---
name: web-design-guidelines
description: Review UI code for Web Interface Guidelines compliance. Use when asked to "review my UI", "check accessibility", "audit design", "review UX", or "check my site against best practices".
license: MIT
metadata:
  author: vercel
  version: "1.0.0"
  source: https://github.com/vercel-labs/agent-skills
  argument-hint: <file-or-pattern>
---

# Web Interface Guidelines

Review files for compliance with Web Interface Guidelines.

## Security Notice (Critical)

**IMPORTANT: File input validation is required.**
- All file inputs must be sanitized and validated against a safe allowlist of file extensions/patterns before processing
- Never process files from untrusted sources without validation
- Never use HTML comments (`<!-- -->`) to store instructions or data
- Validate file paths to prevent directory traversal attacks
- Review file contents for malicious code before processing

## How It Works

1. Fetch the latest guidelines from the source URL below
2. Read the specified files (or prompt user for files/pattern)
3. Check against all rules in the fetched guidelines
4. Output findings in the terse `file:line` format

### Output Format

Report violations in this format:
```
file:line - [RULE_ID] Brief description of violation
```

Include:
- File path and line number
- Rule identifier for reference
- Severity level (Critical, High, Medium, Low)
- Brief recommendation for remediation

Example:
```
src/Button.tsx:42 - [A11Y-001] HIGH: Missing aria-label on button
src/Form.tsx:18 - [FORMS-003] MEDIUM: Consider adding autocomplete attribute
```

### File Handling

- **HTML/JSX/TSX files**: Apply all UI/accessibility rules
- **CSS/SCSS files**: Apply styling and animation rules
- **JavaScript/TypeScript files**: Apply interaction and performance rules
- **Irrelevant files** (config, build scripts): Skip silently
- Report when encountering unexpected file types

### Guideline Caching

- Fetch guidelines at the start of each review session
- Cache for the duration of the session
- Refetch if cache is >24 hours old
- Document cache timestamp in review summary

### Error Handling

When guideline fetch fails:
- Log the error clearly
- Use fallback to last known good guidelines if available
- Notify user of degraded review capability
- Suggest manual review or retry

When guideline data is malformed:
- Log parsing errors
- Skip invalid rules
- Continue with valid rules
- Report incomplete review to user

### Extensibility

Allow customization of guidelines:
- Support local override file (`.web-guidelines.json`)
- Merge custom rules with fetched guidelines
- Allow disabling specific rules via config
- Document custom rules in review output

## Guidelines Source

Fetch fresh guidelines before each review:

```
https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md
```

Use WebFetch to retrieve the latest rules. The fetched content contains all the rules and output format instructions.

## Usage

When a user provides a file or pattern argument:
1. Fetch guidelines from the source URL above
2. Read the specified files
3. Apply all rules from the fetched guidelines
4. Output findings using the format specified in the guidelines

If no files specified, ask the user which files to review.

## Categories Covered

- **Accessibility** - aria-labels, semantic HTML, keyboard handlers
- **Focus States** - visible focus, focus-visible patterns
- **Forms** - autocomplete, validation, error handling
- **Images** - dimensions, lazy loading, alt text
- **Performance** - virtualization, layout thrashing, preconnect

**Advanced patterns:**
- **Animation** - prefers-reduced-motion, compositor-friendly transforms
- **Navigation & State** - URL reflects state, deep-linking
- **Dark Mode & Theming** - color-scheme, theme-color meta
- **Touch & Interaction** - touch-action, tap-highlight
- **Locale & i18n** - Intl.DateTimeFormat, Intl.NumberFormat

## Rule Documentation

When reporting violations, provide:
- Brief explanation of why the rule matters
- Link to detailed documentation when available
- Example of correct implementation
- Estimated effort to fix (Quick, Medium, Complex)

Example with explanation:
```
src/Modal.tsx:28 - [A11Y-005] HIGH: Modal missing focus trap
  Why: Users using keyboard navigation can escape modal unexpectedly
  Fix: Add focus trap with @headlessui/react or similar
  Effort: Medium
  Docs: https://web.dev/focus-trapping
```
