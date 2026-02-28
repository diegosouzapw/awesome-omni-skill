---
name: github-issue
description: "Work on a GitHub issue end-to-end: checkout main, create branch, research best practices, plan implementation, write tests first (TDD), implement, run code review, QA validation, then create PR. Use when given a GitHub issue URL or number to implement."
disable-model-invocation: true
argument-hint: "<issue-url-or-number>"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Task
  - WebFetch
  - WebSearch
---

# GitHub Issue Workflow

Implement a GitHub issue following TDD principles with integrated code review and QA validation.

## Arguments

- `$ARGUMENTS` - GitHub issue URL (e.g., `https://github.com/owner/repo/issues/123`) or issue number (e.g., `123`)

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────────┐
│  1. Setup       → Checkout main, create feature branch          │
│  2. Research    → Fetch issue details, research best practices  │
│  3. Plan        → Create implementation plan, get user approval │
│  4. TDD Cycle   → Write failing test → Implement → Pass         │
│  5. Code Review → Run go-code-reviewer, incorporate feedback    │
│  6. QA Validate → Run qa-requirements-validator, fix gaps       │
│  7. PR Creation → Commit, push, create pull request             │
└─────────────────────────────────────────────────────────────────┘
```

## Process

### Phase 1: Environment Setup

1. **Validate working directory is clean:**
   ```bash
   git status --porcelain
   ```
   If there are uncommitted changes, warn the user and ask how to proceed.

2. **Checkout and update main branch:**
   ```bash
   git checkout main && git pull origin main
   ```

3. **Parse issue identifier from $ARGUMENTS:**
   - If URL: Extract owner, repo, and issue number
   - If number only: Use current repo context

4. **Fetch issue details:**
   ```bash
   gh issue view $ISSUE_NUMBER --json title,body,labels,assignees
   ```

5. **Create feature branch:**
   ```bash
   # Branch naming: fix/issue-{number}-{short-description} or feat/issue-{number}-{short-description}
   git checkout -b {branch-name}
   ```

### Phase 2: Research & Understanding

1. **Parse issue requirements:**
   - Extract acceptance criteria from issue body
   - Identify affected components/files
   - Note any linked issues or PRs

2. **Research best practices:**
   - Use WebSearch to find relevant patterns for the problem domain
   - Check existing codebase patterns using Glob and Grep
   - Review similar implementations in the project

3. **Explore affected code:**
   - Use the Explore agent to understand the codebase area:
   ```
   Task(
     prompt="Explore how {affected_area} works in this codebase. Find relevant files, understand the architecture, and identify where changes need to be made for: {issue_summary}",
     subagent_type="Explore"
   )
   ```

4. **Document findings:**
   - Create a mental model of what needs to change
   - Identify potential risks or edge cases

### Phase 3: Implementation Plan

1. **Create structured plan:**

   Present to user for approval:
   ```markdown
   ## Implementation Plan for Issue #{number}: {title}

   ### Requirements (from issue)
   - [ ] Requirement 1
   - [ ] Requirement 2
   ...

   ### Approach
   {High-level description of the solution}

   ### Files to Modify/Create
   | File | Action | Purpose |
   |------|--------|---------|
   | path/to/file.go | Modify | Add X functionality |
   | path/to/file_test.go | Create | Test cases for X |

   ### Test Strategy (TDD)
   1. Test case 1: {description}
   2. Test case 2: {description}
   ...

   ### Risks & Mitigations
   - Risk: {potential issue}
     Mitigation: {how to handle}
   ```

2. **Get user approval:**
   - Present the plan
   - Ask: "Does this plan look good? Should I proceed with implementation?"
   - Wait for explicit approval before continuing

### Phase 4: TDD Implementation Cycle

For each feature/requirement, follow RED-GREEN-REFACTOR:

#### 4.1 RED: Write Failing Test First

1. **Create/update test file:**
   - Write test that captures the expected behavior
   - Include edge cases identified in planning

2. **Run test to confirm it fails:**
   ```bash
   go test ./... -run TestNamePattern -v
   ```

3. **Verify failure is for the right reason:**
   - Test should fail because functionality doesn't exist yet
   - NOT because of syntax errors or wrong test setup

#### 4.2 GREEN: Implement Minimum Code to Pass

1. **Write implementation:**
   - Only enough code to make the test pass
   - Follow existing code patterns in the project
   - Refer to CLAUDE.md for project conventions

2. **Run tests:**
   ```bash
   go test ./... -v
   ```

3. **Iterate until all tests pass**

#### 4.3 REFACTOR: Improve Code Quality

1. **Clean up implementation:**
   - Remove duplication
   - Improve naming
   - Simplify logic where possible

2. **Ensure tests still pass:**
   ```bash
   go test ./... -v
   ```

### Phase 5: Code Review

1. **Run go-code-reviewer agent:**
   ```
   Task(
     prompt="Review the code changes I made for GitHub issue #{number}. Focus on:
     - KISS (Keep It Simple, Stupid) violations
     - DRY (Don't Repeat Yourself) violations
     - Go best practices and idioms
     - Error handling
     - Test coverage quality

     The changes are in these files: {list of modified files}

     Provide specific, actionable feedback.",
     subagent_type="go-code-reviewer"
   )
   ```

2. **Address feedback:**
   - For each issue identified:
     - If Critical/Important: Fix immediately
     - If Suggestion: Evaluate and fix if reasonable
   - Run tests after each fix

3. **Re-run reviewer if significant changes made:**
   - Ensure new issues weren't introduced

### Phase 6: QA Validation

1. **Run qa-requirements-validator agent:**
   ```
   Task(
     prompt="Validate my implementation against the requirements from GitHub issue #{number}.

     Requirements from issue:
     {parsed requirements from Phase 2}

     Implementation files:
     {list of modified files}

     Verify each requirement is:
     - Fully implemented
     - Properly tested
     - Handles edge cases

     Provide a detailed validation report.",
     subagent_type="qa-requirements-validator"
   )
   ```

2. **Address gaps:**
   - For any requirement NOT FULFILLED or PARTIALLY FULFILLED:
     - Go back to Phase 4 (TDD cycle) for that requirement
     - Write test for missing case
     - Implement fix
     - Re-run QA validation

3. **Continue until all requirements are FULFILLED**

### Phase 7: Commit & Create PR

1. **Final test run:**
   ```bash
   go test ./... -v
   ```

2. **Stage changes:**
   ```bash
   git add {specific files}
   ```

3. **Create commit:**
   ```bash
   git commit -m "$(cat <<'EOF'
   fix: {short description} (#{issue_number})

   {Longer description of what was done}

   - {Change 1}
   - {Change 2}

   Closes #{issue_number}

   Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
   EOF
   )"
   ```

4. **Push branch:**
   ```bash
   git push -u origin {branch-name}
   ```

5. **Create pull request:**
   ```bash
   gh pr create --title "{title}" --body "$(cat <<'EOF'
   ## Summary

   Fixes #{issue_number}

   {Brief description of changes}

   ## Changes Made

   - {Change 1}
   - {Change 2}

   ## Test Plan

   - [ ] All existing tests pass
   - [ ] New tests added for {feature}
   - [ ] Manually verified {scenario}

   ## Checklist

   - [x] Code follows project style guidelines
   - [x] Tests written and passing
   - [x] Code reviewed by go-code-reviewer
   - [x] Requirements validated by qa-requirements-validator

   ---
   Generated with Claude Code
   EOF
   )"
   ```

6. **Report PR URL to user**

## Error Handling

### Git Conflicts
If merge conflicts occur:
1. Report to user with affected files
2. Ask for guidance on resolution
3. Do NOT auto-resolve conflicts

### Test Failures
If tests fail unexpectedly:
1. Analyze failure reason
2. If test bug: Fix test
3. If implementation bug: Fix implementation
4. If unclear: Ask user for guidance

### API/CLI Errors
If `gh` commands fail:
1. Check if user is authenticated: `gh auth status`
2. Report specific error to user
3. Provide remediation steps

## Quality Gates

Before proceeding to each phase, verify:

- [ ] **Phase 1 → 2**: Branch created, issue details fetched
- [ ] **Phase 2 → 3**: Requirements understood, codebase explored
- [ ] **Phase 3 → 4**: User approved the plan
- [ ] **Phase 4 → 5**: All tests pass, implementation complete
- [ ] **Phase 5 → 6**: Code review feedback addressed
- [ ] **Phase 6 → 7**: All requirements validated as FULFILLED
- [ ] **Phase 7 complete**: PR created and URL provided

## Output

After completion, provide summary:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GITHUB ISSUE COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Issue: #{number} - {title}
Branch: {branch-name}
PR: {pr-url}

Changes:
- {file1}: {description}
- {file2}: {description}

Tests Added: {count}
Code Review: Passed
QA Validation: All requirements fulfilled

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
