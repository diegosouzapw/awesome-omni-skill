---
name: a11y-self-check
description: Proactively validates Claude Code's own generated HTML/JSX/TSX output for accessibility before presenting to users. Use this skill automatically when generating UI code to ensure WCAG 2.1 AA compliance.
---

# A11y Self-Check for Claude Code

This skill enables Claude Code to validate its own generated HTML/JSX/TSX output for accessibility issues before presenting code to users.

## When to Use

Proactively run self-check when you (Claude Code):

1. **Generate new HTML/JSX/TSX files** - Validate before suggesting to user
2. **Write UI components** - Check before completing the task
3. **Modify existing templates** - Verify changes don't introduce issues
4. **Create forms, modals, navigation** - High-risk accessibility areas

## Self-Check Workflow

### Step 1: Write Code to Temporary File

When generating HTML/JSX/TSX code, write it to a temporary file:

```bash
# For generated HTML
cat > /tmp/claude-generated.html << 'EOF'
[your generated HTML here]
EOF

# For JSX/TSX
cat > /tmp/claude-generated.tsx << 'EOF'
[your generated JSX/TSX here]
EOF
```

### Step 2: Run A11y Lint

Execute the lint script on the temporary file:

```bash
bash ${CLAUDE_PLUGIN_ROOT}/skills/html-lint-runner/scripts/lint-html.sh /tmp/claude-generated.html
```

### Step 3: Analyze Results

Check the JSON output for issues:

```json
{
  "summary": {
    "axe_violations": 0,
    "markuplint_problems": 0,
    "total_issues": 0
  }
}
```

- If `total_issues` is 0: Code is ready to present
- If `total_issues` > 0: Fix issues before presenting

### Step 4: Fix and Re-validate

If issues found:
1. Look up WCAG/ARIA reference: `${CLAUDE_PLUGIN_ROOT}/skills/wcag-aria-lookup/`
2. Apply fixes to your generated code
3. Re-run lint to confirm resolution
4. Present corrected code to user

## Quick Checks (No Tool Required)

Before running lint tools, perform mental checks:

### Images
- [ ] All `<img>` have `alt` attribute
- [ ] Decorative images use `alt=""`
- [ ] Complex images have `aria-describedby`

### Forms
- [ ] All inputs have associated `<label>`
- [ ] Required fields have `aria-required="true"`
- [ ] Error states use `aria-invalid` and `aria-describedby`

### Interactive Elements
- [ ] Buttons have accessible names
- [ ] Links have descriptive text (not "click here")
- [ ] Custom controls have proper ARIA roles

### Structure
- [ ] Headings follow logical order (h1 → h2 → h3)
- [ ] Landmarks are used (`<main>`, `<nav>`, `<aside>`)
- [ ] Language is set on `<html lang="...">`

### Color & Contrast
- [ ] Text contrast ≥ 4.5:1 (normal), ≥ 3:1 (large)
- [ ] Information not conveyed by color alone

## Integration with /a11y-audit

For comprehensive validation, use the `/a11y-audit` command:

```
/a11y-audit /tmp/claude-generated.html
```

This spawns the `a11y-fixer` agent for detailed analysis with WCAG references.

## Example: Self-Validated Component Generation

```
User: "Create a login form component"

Claude Code (internal process):
1. Generate LoginForm.tsx
2. Write to /tmp/claude-generated.tsx
3. Run: bash ${CLAUDE_PLUGIN_ROOT}/skills/html-lint-runner/scripts/lint-html.sh /tmp/claude-generated.tsx
4. Results show: missing label association
5. Fix: Add htmlFor and id attributes
6. Re-run lint: 0 issues
7. Present validated code to user
```

## Proactive Quality Assurance

This skill transforms Claude Code from generating "potentially accessible" code to generating "validated accessible" code by:

1. **Pre-flight checks** - Mental validation before writing
2. **Automated validation** - Tool-based verification after writing
3. **Reference lookup** - WCAG/ARIA guidance for fixes
4. **Iteration** - Fix → Validate → Confirm cycle

Users receive code that has already passed accessibility checks.
