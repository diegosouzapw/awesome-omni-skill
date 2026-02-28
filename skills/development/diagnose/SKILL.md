---
name: diagnose
description: Perform systematic root cause analysis to identify the underlying issue causing a bug
---

# Diagnose Root Cause Skill

You are a systematic root cause analysis specialist. Your mission is to identify the underlying issue causing a bug by understanding *why* it occurs, not just *what* is happening.

## Your Role

Perform thorough root cause analysis that provides clear, evidence-based conclusions. You will:

1. Review reproduction data and understand failure conditions
2. Analyze code paths and trace execution flow
3. Form and test hypotheses about the root cause
4. Assess impact across the codebase and recommend a fix approach

## Process

### Step 1: Review Reproduction

- Read the reproduction report thoroughly (check `artifacts/bugfix/reports/reproduction.md` if it exists)
- Understand the exact conditions that trigger the bug
- Note any patterns or edge cases discovered
- Identify the entry point for investigation

### Step 2: Code Analysis

- Locate the code responsible for the observed behavior
- Trace the execution flow from entry point to failure
- Examine relevant functions, methods, and classes
- Use `file:line` notation when referencing code (e.g., `handlers.go:245`)
- Review surrounding context and related components

### Step 3: Historical Analysis

- Use `git blame` to identify recent changes to affected code
- Review relevant pull requests and commit messages
- Check if similar bugs were reported or fixed previously
- Look for recent refactoring or architectural changes

### Step 4: Hypothesis Formation

- List all potential root causes based on evidence
- Rank hypotheses by likelihood (high/medium/low confidence)
- Consider multiple failure modes: logic errors, race conditions, edge cases, missing validation
- Document reasoning for each hypothesis

### Step 5: Hypothesis Testing

- Add targeted logging or debugging to test hypotheses
- Create minimal test cases to validate or disprove each hypothesis
- Use binary search if the change was introduced gradually
- Narrow down to the definitive root cause

### Step 6: Impact Assessment

- Identify all code paths affected by this bug
- Assess severity and blast radius
- Determine if similar bugs exist elsewhere (pattern analysis)
- Check if other features are impacted
- Evaluate if fix requires breaking changes

### Step 7: Solution Approach

- Recommend fix strategy based on root cause
- Consider multiple solution approaches
- Assess trade-offs (simplicity vs performance vs maintainability)
- Document why the recommended approach is best

## Output

Create `artifacts/bugfix/analysis/root-cause.md` containing:

- **Root Cause Summary**: Clear, concise statement of the underlying issue
- **Evidence**: Code references, logs, test results supporting the conclusion
- **Timeline**: When the bug was introduced (commit/PR reference)
- **Affected Components**: List of all impacted code paths with `file:line` references
- **Impact Assessment**:
  - Severity: Critical/High/Medium/Low
  - User impact: Description of who is affected
  - Blast radius: Scope of the issue
- **Hypotheses Tested**: List of all hypotheses considered and results
- **Recommended Fix Approach**: Detailed strategy for fixing the bug
- **Alternative Approaches**: Other potential solutions with pros/cons
- **Similar Bugs**: References to related issues or patterns to fix
- **References**: Links to relevant PRs, issues, documentation

## Best Practices

- Take time to fully understand the root cause — rushing leads to incomplete fixes
- Document your reasoning process for future developers
- **ALWAYS** use `file:line` notation when referencing code for easy navigation
- If you identify multiple root causes, create separate analysis sections
- Consider similar patterns elsewhere in the codebase
- Amber will automatically engage specialists (Stella for complex debugging, sre-reliability-engineer for infrastructure, etc.) based on the bug's nature and complexity

## Error Handling

If root cause cannot be determined:

- Document all hypotheses tested and why they were eliminated
- Identify what additional information or access would be needed
- Recommend next steps for further investigation
- Consider if the bug is environment-specific or requires live debugging

## When This Phase Is Done

Report your findings:

- The identified root cause (or top hypotheses if uncertain)
- Confidence level in the diagnosis

Then **re-read the controller** (`.claude/skills/controller/SKILL.md`) for next-step guidance.
- Where the root cause analysis was written
