---
name: rule-auditor
description: Validates code against currently loaded rules and reports compliance violations. Supports auto-fixing violations with confirmation, dry-run mode, and automatic backups. Use after implementing features, during code review, or to ensure coding standards are followed. Provides actionable feedback with line-by-line issues and suggested fixes.
context:fork: true
model: haiku
allowed-tools: read, grep, glob, search, codebase_search, edit, write
version: 3.1
executable: scripts/audit.mjs
best_practices:
  - Run audits early in development cycle
  - Focus on high-severity violations first
  - Provide specific, actionable fixes
  - Group violations by rule category
  - Include code examples in suggestions
  - Use --fix-dry-run to preview changes before applying
  - Review backup files (.bak) after auto-fixing
error_handling: graceful
streaming: supported
output_formats: [markdown, json, inline_comments]
---

## Executable Script

The rule-auditor skill now includes an executable script for CLI usage and programmatic validation.

### Installation

No installation required - the script uses Node.js built-in modules only.

### CLI Usage

```bash
# Audit a directory
node .claude/skills/rule-auditor/scripts/audit.mjs src/components/

# Audit specific file with JSON output
node .claude/skills/rule-auditor/scripts/audit.mjs src/App.tsx --format json

# Preview fixes (dry run)
node .claude/skills/rule-auditor/scripts/audit.mjs src/ --fix-dry-run

# Apply fixes (creates .bak backups)
node .claude/skills/rule-auditor/scripts/audit.mjs src/ --fix

# Audit with specific rules only
node .claude/skills/rule-auditor/scripts/audit.mjs src/ --rules nextjs,typescript

# Strict mode (fail on any violation)
node .claude/skills/rule-auditor/scripts/audit.mjs src/ --strict
```

### Output Schema

All output conforms to `.claude/schemas/skill-rule-auditor-output.schema.json` and includes:

- `skill_name`: Always "rule-auditor"
- `files_audited`: Array of audited files with line counts
- `rules_applied`: Rules used during audit with violation counts
- `compliance_score`: 0-100 score based on violations
- `violations_found`: Detailed violations with locations
- `fixes_applied`: Applied fixes (when using --fix or --fix-dry-run)
- `rule_index_consulted`: Boolean confirming rule index was loaded
- `technologies_detected`: Technologies detected in codebase
- `audit_summary`: Summary statistics
- `timestamp`: ISO 8601 timestamp

### Testing

Run the test suite to validate the audit script:

```bash
node .claude/skills/rule-auditor/scripts/test-audit.mjs
```

Tests cover:

- Basic audit functionality
- Technology detection
- Dry-run fix mode
- Fix mode with backups
- Compliance score calculation
- Exit codes
- Output schema validation

---

<identity>
Rule Auditor - Automatically validates code against your project's coding standards and rules.
</identity>

<capabilities>
- After implementing a new feature or component
- During code review to check standards compliance
- Before committing to ensure quality gates pass
- When onboarding to understand project conventions
- To generate compliance reports for teams
</capabilities>

<instructions>
<execution_process>

### Step 1: Load Rule Index

Load the rule index to discover all available rules dynamically:

- @.claude/context/rule-index.json

The index contains metadata for all 1,081+ rules in `.claude/rules-master/` and `.claude/rules-library/` (formerly archive).

### Step 2: Filter Relevant Rules

Query the index's `technology_map` based on target files:

1. **Detect technologies** from target files:
   - File extension (`.tsx` → TypeScript, React)
   - Import statements (`next` → Next.js, `react` → React)
   - Directory structure (`app/` → Next.js App Router)

2. **Query technology_map**:

   ```javascript
   // Pseudocode
   const detectedTech = ['nextjs', 'react', 'typescript'];
   const relevantRules = [];

   detectedTech.forEach(tech => {
     const rules = index.technology_map[tech] || [];
     relevantRules.push(...rules);
   });
   ```

3. **Load only relevant rule files** (progressive disclosure):
   - Master rules take priority (from `.claude/rules-master/`)
   - Library rules supplement (from `.claude/rules-library/`, formerly archive)
   - Load 5-10 most relevant rules, not all 1,081

### Step 3: Scan Target Files

Identify the files to audit based on the task:

```bash
# For a specific file
audit: src/components/UserAuth.tsx

# For a directory
audit: src/components/

# For recent changes
git diff --name-only HEAD~1
```

### Step 4: Extract Validation Patterns from Rules

Parse rule files to extract formalized validation patterns:

1. **Look for `<validation>` block in rule file** (deterministic approach):

   ```markdown
   <validation>
   forbidden_patterns:
     - pattern: "useEffect\\(.*fetch"
       message: "Do not use useEffect for data fetching; use Server Components."
       severity: "error"
     - pattern: "console\\.log"
       message: "Remove console.log statements before commit."
       severity: "warning"
   </validation>
   ```

2. **OR look for `validation` section in rule frontmatter** (legacy support):

   ```yaml
   validation:
     forbidden_patterns:
       - pattern: "useEffect\\(.*fetch"
         message: 'Do not use useEffect for data fetching; use Server Components.'
         severity: 'error'
   ```

3. **Extract forbidden_patterns**:
   - **First priority**: Load from `<validation>` block (deterministic, preferred)
   - **Second priority**: Load from `validation.forbidden_patterns` in frontmatter
   - Each pattern includes:
     - `pattern`: Regex pattern to match
     - `message`: Human-readable violation message
     - `severity`: "error" or "warning"

4. **Fallback to pattern extraction** (if no validation block/section):
   - Parse rule text for common patterns
   - Convert natural language rules to regex where possible
   - Use grep/pattern matching for structure checks
   - **Note**: This is slower and less consistent than using `<validation>` blocks

**Pattern Categories to Check:**

| Category       | Example Rule                            | Check Method               |
| -------------- | --------------------------------------- | -------------------------- |
| Naming         | "Use camelCase for functions"           | Regex scan                 |
| Structure      | "Place components in `components/` dir" | Path check                 |
| Imports        | "Use ES modules, not CommonJS"          | Pattern match              |
| Types          | "Avoid `any`, prefer `unknown`"         | AST-level grep             |
| Performance    | "Use Server Components by default"      | Directive scan             |
| Security       | "Never hardcode secrets"                | Pattern detection          |
| **Formalized** | `validation.forbidden_patterns`         | **Regex/grep (preferred)** |

**Fix Field in Validation Patterns**:

Each pattern can optionally include a `fix` field for auto-fixing:

```yaml
- pattern: "console\\.log\\((.*)\\)"
  message: 'Remove console.log statements'
  severity: 'warning'
  fix: '' # Empty string = delete the entire match

- pattern: "const (\\w+): any"
  message: "Avoid using 'any' type"
  severity: 'error'
  fix: 'const $1: unknown' # $1 references first capture group

- pattern: "var (\\w+) ="
  message: "Use 'const' or 'let' instead of 'var'"
  severity: 'warning'
  fix: 'const $1 =' # Replace var with const
```

**Fix Replacement Syntax**:

- `""` (empty string): Delete the entire matched pattern
- `"// Removed: $0"`: Replace with comment (where $0 is the full match)
- `"const $1"`: Use capture groups ($1, $2, etc.) from the pattern
- Fixed string: Replace match with literal text

### Step 5: Run Validation Checks

For each forbidden pattern:

1. **Run grep/regex search** on target files:
   - Use pattern from `validation.forbidden_patterns`
   - Search across all target files
   - Report matches with line numbers

2. **Report matches**:
   - Include line number and file path
   - Include message from pattern definition
   - Include severity (error/warning)
   - Show code snippet with match highlighted

3. **Aggregate results by severity**:
   - Group errors separately from warnings
   - Count violations per pattern
   - Sort by severity and frequency

### Step 6: Generate Compliance Report

Output a structured report with violations, warnings, and passed rules.

### Step 7: Quick-Fix Mode (Optional)

When `--fix`, `--fix-auto`, or `--fix-dry-run` flags are provided, automatically apply fixes for violations that have a `fix` field defined.

**Fix Mode Behavior**:

1. **--fix-dry-run** (Safe Preview):
   - Scan for violations with fix definitions
   - Show diff preview of changes without modifying files
   - Display before/after for each fix
   - No file modifications occur
   - Useful for reviewing impact before applying

2. **--fix** (Interactive Mode):
   - Create `.bak` backup file before any modification
   - Show diff preview for each fixable violation
   - Prompt user for confirmation: "Apply this fix? (y/n/all/skip)"
     - `y`: Apply this fix and continue
     - `n`: Skip this fix and continue
     - `all`: Apply all remaining fixes without prompting
     - `skip`: Skip all remaining fixes
   - Apply confirmed fixes using Edit tool
   - Report summary of applied vs skipped fixes

3. **--fix-auto** (Batch Mode):
   - Create `.bak` backup file before any modification
   - Apply all fixable violations automatically
   - No user prompts (fully automated)
   - Generate detailed log of all changes
   - Useful for CI/CD pipelines or bulk cleanup

**Backup Behavior**:

- Backup file: `<original-file>.bak`
- Created before first modification to each file
- Contains original file contents
- Allows manual rollback if needed
- Example: `src/App.tsx` → `src/App.tsx.bak`

**Diff Preview Format**:

```diff
File: src/components/UserAuth.tsx
Line 23:
- console.log('User authenticated:', user);
+ // Removed: console.log('User authenticated:', user);

Line 45:
- const user: any = await getUser();
+ const user: unknown = await getUser();
```

**Fix Application Process**:

1. Read target file
2. Create backup if not in dry-run mode
3. Apply regex replacement using `fix` pattern
4. Handle capture group substitutions ($1, $2, etc.)
5. Write modified content back to file
6. Log change to fix report

**Limitations**:

- Only fixes violations with `fix` field defined in rule
- Complex refactorings require manual intervention
- Multiline fixes may need manual review
- Some patterns may require AST-level transformation
  </execution_process>

<audit_patterns>
**Next.js / React Audit**:

- CHECK: 'use client' directive present when using useState, useEffect, useContext
- CHECK: Server Components for data fetching (no useEffect for fetch)
- CHECK: Image optimization using next/image

**TypeScript Audit**:

- CHECK: Type safety - No `any` types, interfaces for object shapes
- CHECK: Naming conventions - PascalCase for Components, camelCase for functions

**Python/FastAPI Audit**:

- CHECK: Async patterns - async def for I/O operations
- CHECK: Type hints - All function parameters typed
  </audit_patterns>

<integration>
**Pre-Commit Hook**: Run rule audit on modified files before commit
**CI/CD Integration**: Generate JSON reports for automated quality gates
</integration>

<best_practices>

1. **Run Early**: Audit during development, not just before commit
2. **Fix as You Go**: Address violations immediately while context is fresh
3. **Customize Rules**: Adjust rule severity in manifest.yaml for your team
4. **Track Trends**: Monitor violation counts over time to measure improvement
5. **Educate Team**: Use audit reports in code reviews to teach standards
6. **Preview First**: Always run `--fix-dry-run` before `--fix-auto` to review changes
7. **Test After Fixing**: Run tests after auto-fixing to ensure no functionality broke
8. **Keep Backups**: Don't delete .bak files until changes are verified
9. **Review Diffs**: Use `git diff` to review all changes before committing
10. **Start Small**: Test auto-fix on single files before running on entire codebase
    </best_practices>

## Quick-Fix Mode

The rule-auditor now supports automatic fixing of violations through three modes: preview, interactive, and batch.

### Fix Modes

| Mode            | Flag            | Behavior                             | Use Case                      |
| --------------- | --------------- | ------------------------------------ | ----------------------------- |
| **Preview**     | `--fix-dry-run` | Show changes without modifying files | Review impact before applying |
| **Interactive** | `--fix`         | Prompt for confirmation on each fix  | Careful, selective fixing     |
| **Batch**       | `--fix-auto`    | Apply all fixes automatically        | Bulk cleanup, CI/CD           |

### Adding Fix Definitions to Rules

To make violations auto-fixable, add a `fix` field to validation patterns in your rule files:

**In `<validation>` block**:

```markdown
<validation>
forbidden_patterns:
  - pattern: "console\\.log\\((.*)\\)"
    message: "Remove console.log statements before commit"
    severity: "warning"
    fix: ""  # Empty = delete entire match

- pattern: "const (\\w+): any"
  message: "Avoid using 'any' type"
  severity: "error"
  fix: "const $1: unknown" # Replace with unknown

- pattern: "var (\\w+) ="
  message: "Use 'const' or 'let' instead of 'var'"
  severity: "warning"
  fix: "const $1 =" # Replace var with const
  </validation>
```

**Or in frontmatter**:

```yaml
---
validation:
  forbidden_patterns:
    - pattern: "console\\.log\\((.*)\\)"
      message: 'Remove console.log statements'
      severity: 'warning'
      fix: ''
---
```

### Fix Field Syntax

The `fix` field supports several replacement patterns:

1. **Delete Match**: Empty string removes the matched pattern

   ```yaml
   fix: ''
   ```

2. **Capture Group Substitution**: Use $1, $2, etc. to reference regex groups

   ```yaml
   pattern: "const (\\w+): any"
   fix: 'const $1: unknown'
   ```

3. **Fixed Replacement**: Replace with literal text

   ```yaml
   pattern: "var (\\w+) ="
   fix: 'const $1 ='
   ```

4. **Comment Out**: Replace with comment preserving original code
   ```yaml
   pattern: 'debugger;'
   fix: '// debugger;'
   ```

### Safety Features

1. **Automatic Backups**: Creates `.bak` file before any modification
2. **Diff Preview**: Shows before/after for all fixes
3. **Dry-Run Mode**: Test fixes without modifying files
4. **Interactive Confirmation**: Review each fix before applying
5. **Rollback Support**: Restore from .bak files if needed

### Example Workflow

```bash
# 1. Preview what would be fixed
/audit src/ --fix-dry-run

# 2. Apply fixes with confirmation
/audit src/components/UserAuth.tsx --fix

# 3. Review changes
git diff

# 4. Run tests
npm test

# 5. If issues occur, rollback
mv src/components/UserAuth.tsx.bak src/components/UserAuth.tsx

# 6. If all good, commit
git add src/
git commit -m "fix: auto-fix rule violations"

# 7. Clean up backups
rm **/*.bak
```

### Creating Effective Fix Patterns

**Good Fix Patterns** (Safe, Deterministic):

- Simple find-replace operations
- Type annotations (any → unknown)
- Variable declarations (var → const)
- Import statement updates
- Comment/uncomment code
- Formatting fixes

**Poor Fix Patterns** (Manual Review Required):

- Complex refactorings
- Logic changes
- Multiline transformations
- Context-dependent fixes
- AST-level transformations

**Best Practices for Fix Definitions**:

1. Keep fixes simple and deterministic
2. Test fix patterns on sample code before deploying
3. Use capture groups to preserve variable names
4. Prefer commenting over deletion for debugging code
5. Document why a fix is safe in the rule message
6. Consider edge cases where fix might break code

### Limitations

- Only fixes violations with `fix` field defined
- Regex-based replacements only (no AST manipulation)
- Complex refactorings require manual intervention
- Multiline patterns may need special handling
- Some fixes may require post-fix manual adjustments

For complex transformations beyond regex replacement, consider using the `refactoring-specialist` agent.

</instructions>

<examples>
<formatting_example>
**Markdown Report Format**:

```markdown
## Rule Audit Report

**Target**: src/components/UserAuth.tsx
**Rules Applied**: nextjs.mdc, typescript.mdc, react.mdc
**Scan Date**: {{timestamp}}

### Summary

- **Pass**: 12 rules
- **Warn**: 3 rules
- **Fail**: 2 rules

### Violations

#### FAIL: Use Server Components by default

- **File**: src/components/UserAuth.tsx:1
- **Issue**: Missing 'use client' directive but uses useState
- **Rule**: nextjs.mdc > Components
- **Fix**: Add 'use client' at file top, or refactor to Server Component

#### FAIL: Avoid using `any`

- **File**: src/components/UserAuth.tsx:45
- **Issue**: `const user: any = await getUser()`
- **Rule**: typescript.mdc > Type System
- **Fix**: Define proper User interface

#### WARN: Minimize use of 'useEffect'

- **File**: src/components/UserAuth.tsx:23
- **Issue**: useEffect for data fetching
- **Rule**: nextjs.mdc > Performance
- **Suggestion**: Consider Server Component with async/await

### Passed Rules

- ✅ Use TypeScript strict mode
- ✅ Use lowercase with dashes for directories
- ✅ Implement proper error boundaries
  ... (12 more)
```

</formatting_example>

<code_example>
**JSON Output Format (for CI/CD)**:

```json
{
  "target": "src/components/",
  "timestamp": "2025-11-29T10:00:00Z",
  "rules_applied": ["nextjs.mdc", "typescript.mdc"],
  "summary": {
    "pass": 12,
    "warn": 3,
    "fail": 2
  },
  "violations": [
    {
      "severity": "fail",
      "rule": "typescript.mdc",
      "pattern": "Avoid using any",
      "file": "src/components/UserAuth.tsx",
      "line": 45,
      "code": "const user: any = await getUser()",
      "fix": "Define User interface"
    }
  ]
}
```

</code_example>

<code_example>
**Inline Comments Format**:

```typescript
// RULE_VIOLATION: typescript.mdc > Avoid using `any`
// FIX: Define User interface with proper types
const user: any = await getUser(); // ❌ FAIL
```

</code_example>

<code_example>
**Pre-Commit Hook**:

```bash
#!/bin/bash
# .claude/hooks/pre-commit-audit.sh
# Hook: PreToolUse (for Edit, Write tools)

# Run rule audit on modified files
modified=$(git diff --cached --name-only)
for file in $modified; do
  audit_result=$(claude skill rule-auditor --target "$file" --format json)
  if echo "$audit_result" | jq -e '.summary.fail > 0' > /dev/null; then
    echo "Rule violations found in $file"
    exit 1
  fi
done
```

</code_example>

<code_example>
**CI/CD Integration (GitHub Actions)**:

```yaml
# GitHub Actions example
- name: Rule Audit
  run: |
    claude skill rule-auditor --target src/ --format json > audit-report.json
    if [ $(jq '.summary.fail' audit-report.json) -gt 0 ]; then
      echo "::error::Rule violations detected"
      exit 1
    fi
```

</code_example>

<usage_example>
**Quick Commands**:

```
# Audit current file
/audit this file

# Audit a specific directory
/audit src/components/

# Audit with specific rules only
/audit src/ --rules nextjs,typescript

# Generate CI-friendly output
/audit src/ --format json --strict

# Show only failures
/audit src/ --severity fail

# Preview fixes without applying (dry-run)
/audit src/components/ --fix-dry-run

# Apply fixes with confirmation
/audit src/components/UserAuth.tsx --fix

# Apply all fixes automatically (batch mode)
/audit src/ --fix-auto

# Dry-run with JSON output for CI/CD
/audit src/ --fix-dry-run --format json
```

</usage_example>

<usage_example>
**Quick-Fix Mode Examples**:

**Example 1: Dry-Run Preview**

Command:

```bash
/audit src/components/UserAuth.tsx --fix-dry-run
```

Output:

```diff
## Fix Preview (Dry-Run Mode)

### Fixable Violations: 3

#### Fix 1/3
File: src/components/UserAuth.tsx:23
Rule: Remove console.log statements
Severity: warning

- console.log('User authenticated:', user);
+ // Removed: console.log('User authenticated:', user);

#### Fix 2/3
File: src/components/UserAuth.tsx:45
Rule: Avoid using 'any' type
Severity: error

- const user: any = await getUser();
+ const user: unknown = await getUser();

#### Fix 3/3
File: src/components/UserAuth.tsx:67
Rule: Use 'const' or 'let' instead of 'var'
Severity: warning

- var isAuthenticated = true;
+ const isAuthenticated = true;

---
Summary: 3 fixable violations found
No files modified (dry-run mode)
```

**Example 2: Interactive Fix Mode**

Command:

```bash
/audit src/components/UserAuth.tsx --fix
```

Interactive Session:

```
Creating backup: src/components/UserAuth.tsx.bak

Fix 1/3:
File: src/components/UserAuth.tsx:23
Rule: Remove console.log statements

- console.log('User authenticated:', user);
+ // Removed: console.log('User authenticated:', user);

Apply this fix? (y/n/all/skip): y
✓ Applied fix 1/3

Fix 2/3:
File: src/components/UserAuth.tsx:45
Rule: Avoid using 'any' type

- const user: any = await getUser();
+ const user: unknown = await getUser();

Apply this fix? (y/n/all/skip): all
✓ Applied fix 2/3
✓ Applied fix 3/3

---
Fix Summary:
- Total violations: 3
- Fixes applied: 3
- Fixes skipped: 0
- Backup created: src/components/UserAuth.tsx.bak

Run git diff to review changes.
```

**Example 3: Batch Auto-Fix**

Command:

```bash
/audit src/ --fix-auto
```

Output:

```
## Auto-Fix Report

Creating backups for modified files...
✓ src/components/UserAuth.tsx.bak
✓ src/components/Profile.tsx.bak
✓ src/utils/helpers.ts.bak

Applying fixes...

src/components/UserAuth.tsx:
  ✓ Line 23: Removed console.log statement
  ✓ Line 45: Changed 'any' to 'unknown'
  ✓ Line 67: Changed 'var' to 'const'

src/components/Profile.tsx:
  ✓ Line 12: Removed console.log statement
  ✓ Line 89: Changed 'var' to 'const'

src/utils/helpers.ts:
  ✓ Line 34: Changed 'any' to 'unknown'
  ✓ Line 78: Changed 'any' to 'unknown'

---
Fix Summary:
- Files modified: 3
- Total fixes applied: 7
- Backups created: 3

Next steps:
1. Run 'git diff' to review all changes
2. Run tests to ensure fixes didn't break functionality
3. If issues occur, restore from .bak files
```

**Example 4: Before/After Comparison**

**Before** (src/components/UserAuth.tsx):

```typescript
import { useState } from 'react';

export default function UserAuth() {
  var isAuthenticated = false;
  const user: any = await getUser();

  console.log('User authenticated:', user);

  return <div>Welcome {user.name}</div>;
}
```

**After** (with `--fix-auto`):

```typescript
import { useState } from 'react';

export default function UserAuth() {
  const isAuthenticated = false;
  const user: unknown = await getUser();

  // Removed: console.log('User authenticated:', user);

  return <div>Welcome {user.name}</div>;
}
```

**Backup** (src/components/UserAuth.tsx.bak):

```typescript
// Original file preserved - can restore with:
// mv src/components/UserAuth.tsx.bak src/components/UserAuth.tsx
```

</usage_example>

<usage_example>
**CI/CD Integration with Auto-Fix**:

**.github/workflows/code-quality.yml**:

```yaml
name: Code Quality with Auto-Fix

on: [pull_request]

jobs:
  audit-and-fix:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      # Step 1: Audit and preview fixes
      - name: Audit Code (Dry-Run)
        run: |
          claude skill rule-auditor --target src/ --fix-dry-run --format json > fix-preview.json

      # Step 2: Apply fixes automatically
      - name: Auto-Fix Violations
        run: |
          claude skill rule-auditor --target src/ --fix-auto --format json > fix-report.json

      # Step 3: Commit fixes back to PR
      - name: Commit Fixes
        run: |
          git config user.name "Rule Auditor Bot"
          git config user.email "bot@example.com"
          git add -A
          git commit -m "chore: auto-fix rule violations" || echo "No fixes applied"
          git push

      # Step 4: Post summary as PR comment
      - name: Post Fix Summary
        uses: actions/github-script@v6
        with:
          script: |
            const report = require('./fix-report.json');
            const comment = `## Auto-Fix Summary
            - Files modified: ${report.files_modified}
            - Total fixes: ${report.total_fixes}
            - See commit for details`;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: comment
            });
```

**Pre-Commit Hook with Auto-Fix**:

**.claude/hooks/pre-commit-autofix.sh**:

```bash
#!/bin/bash
# Auto-fix violations before commit

echo "Running rule audit with auto-fix..."

# Get staged files
staged_files=$(git diff --cached --name-only --diff-filter=ACM | grep -E '\.(ts|tsx|js|jsx)$')

if [ -z "$staged_files" ]; then
  echo "No files to audit"
  exit 0
fi

# Run audit with auto-fix on staged files
for file in $staged_files; do
  claude skill rule-auditor --target "$file" --fix-auto

  # Re-stage the fixed file
  git add "$file"
done

echo "✓ Auto-fix complete. Review changes with 'git diff --cached'"
```

</usage_example>
</examples>
