---
name: feature-dev-workflow
description: Complete end-to-end feature development workflow from issue tracking through PR delivery. Use for implementing features, building new functionality, and adding capabilities. Includes requirements discovery, architecture planning, implementation, testing, code review, design audit, and comprehensive validation.
license: MIT
metadata:
  author: powerful-systems
  version: "1.0.0"
---

# feature-dev-workflow

Execute complete feature development from issue through PR. Optimized for Claude 4.5's long-horizon reasoning and state tracking.

## Core Principles

1. **Incremental Progress**: Steady advances on few things at a time
2. **Persistent Simplification**: Challenge complexity at every phase
3. **State Tracking**: Document decisions across phases
4. **Self-Validation**: Check work before proceeding
5. **Fail Fast**: Stop and ask when blocked

## Issue Tracking

This skill supports **both Linear and GitHub Issues**:

| Format | System | Example |
|--------|--------|---------|
| `ABC-123` | Linear | Linear ticket ID |
| `#123` or `GH-123` | GitHub | GitHub issue number |
| URL | Either | Full issue URL |

**Auto-detection**: The skill will detect which system to use based on the issue format provided.

## Invocation

Receive issue ID (Linear: "ABC-123", GitHub: "#123" or "GH-123"). Follow workflow sequentially, maintain detailed TODO list.

## General Rules

**Self-checks**: If any self-check fails, take corrective action before proceeding.

**Stopping conditions**: Stop and use AskUserQuestion if:
- Blocked on same issue 3+ times
- Browser automation fails 3+ times
- Approach seems fundamentally wrong
- Scope creep detected
- Uncertain about product decisions
- Need to discuss tradeoffs
- Self-check reveals unresolved issues

---

## Phase 1: Requirements Discovery

**Goal**: Transform rough issue into well-defined spec.

**Process**:
1. **Fetch issue** (detect system from format):
   - **Linear** (`ABC-123`): Use Linear MCP to fetch ticket
   - **GitHub** (`#123`, `GH-123`): Use `gh issue view <number> --json title,body,labels,assignees`
2. **Explore codebase**: Related patterns, similar implementations, data models
3. **Develop 2-3 competing hypotheses**: Document alternatives with pros/cons, confidence levels
4. **AskUserQuestion**: Unclear requirements, product decisions, tradeoffs, scope
5. **Push back**: Challenge complexity, suggest scope cuts
6. **Update issue with refined spec**:
   - **Linear**: Use Linear MCP `update_issue` to update description
   - **GitHub**: Use `gh issue edit <number> --body "..."` to update description

**Self-Check**:
- [ ] User value clear?
- [ ] Simplest approach identified?
- [ ] Ambiguities resolved?
- [ ] Pushed back on complexity?

**Output**: Refined ticket:

```markdown
# [Concise Feature Title]

## Outcome
[User-facing value we're achieving]

## Architecture
[High-level technical approach using existing patterns]
- Data model: [changes needed]
- API/Backend: [approach]
- Frontend: [structure and integration points]
- Reuse: [existing code being leveraged]

## Verification Steps
- [ ] Testable success criterion 1
- [ ] Testable success criterion 2
- [ ] Edge case N

## Decisions Made
- Decision: [rationale]
- Tradeoff: [choice and why]
```

**State**: Approaches evaluated, decisions made, scope defined, user answers

**Checkpoint**: Update ticket. Summarize decisions and await approval.

---

## Phase 2: Implementation Planning

**Goal**: Deep understanding of implementation.

**Process**:
1. **Explore codebase**: Implementation details, types/interfaces, patterns, data flow
2. **Plan data layer**: Database changes? API endpoints? Data models?
3. **Plan frontend**: Component hierarchy? UI placement? State management?
4. **Parallel work**: Can backend/frontend work simultaneously? Clear boundaries?

**Self-Check**:
- [ ] Files to create/modify identified?
- [ ] Dependencies mapped?
- [ ] Parallelization safe?
- [ ] Approach simplifiable?

**Stopping**: If stuck 3+ times, ask user.

**State**: Files to modify, dependencies, parallel plan, open questions

**Checkpoint**: Document approach. Proceed to Phase 3.

---

## Phase 3: Simplification Review

**Goal**: Reduce complexity before coding.

**Process**:
1. **Challenge design**: Better architecture? Code reuse? Component hierarchy? Remove features?
2. **Question abstractions**: Need this layer? Flatter approach? Simpler way?
3. **No backwards compatibility** unless stated

**Self-Check**:
- [ ] Unnecessary abstractions removed?
- [ ] Reusing existing code?
- [ ] Simplest design that works?

**Decision Point**: If simplification reveals issues, use AskUserQuestion.

**State**: Simplifications made, abstractions removed, reuse identified

**Checkpoint**: Summarize approach and await approval.

---

## Phase 4: Implementation

**Goal**: Build feature incrementally.

**Process**:
1. **Create feature branch**:
   - **Linear**: Fetch ticket via MCP to get `branchName` field (e.g., 'username/abc-123-feature-name')
   - **GitHub**: Generate branch name from issue (e.g., 'feat/gh-123-issue-title-slug')
   - Create and checkout feature branch: `git checkout -b [branchName]`
   - Verify branch created: `git branch --show-current`

2. **Parallel execution** (if applicable):
   - Define task boundaries clearly
   - One agent per layer (backend/frontend/infrastructure)
   - Document coordination upfront
   - Monitor conflicts

3. **Follow project patterns**:
   - Check project CLAUDE.md, README.md, or docs/ for coding standards
   - Use existing patterns for consistency
   - Leverage relevant Claude skills for your tech stack

4. **Implement systematically**:
   - Backend/data layer first
   - API endpoints second
   - Frontend components third
   - Monitor dev server/hot reload

**Self-Check** (after each layer):
- [ ] Follows project patterns?
- [ ] Validated properly (types, schemas)?
- [ ] Simpler than plan?
- [ ] Dev environment working?

**Stopping**: If stuck 3+ attempts, ask user.

**State**: Files modified, patterns used, deviations, blockers

**Checkpoint**: Implementation complete. Proceed to Phase 5.

---

## Phase 5: Initial Testing

**Goal**: Working state via end-to-end testing.

**Process**:
1. Start development server (if not already running)
2. **Browser testing**: Use browser automation to test end-to-end if applicable
3. **Test loop**: Happy path → fix bugs → re-test
4. **Check logs**: Console errors, server logs, network requests

**Stopping**: If testing fails repeatedly, try alternative approach or ask user.

**Self-Check**:
- [ ] Happy path works?
- [ ] Tested in appropriate environment?
- [ ] Errors checked and resolved?

**State**: Tests performed, bugs fixed, approach used

**Checkpoint**: Feature works. Proceed to Phase 6.

---

## Phase 6: Code Simplification

**Goal**: Reduce implementation complexity.

**Process**:
1. Review with fresh perspective
2. **Run code-simplifier skill** if available
3. **Check for common issues**:
   - Unnecessary abstractions
   - Dead code
   - Over-engineered solutions
   - Premature optimization
4. **Challenge assumptions**: Drastically simpler? Tradeoffs? Better approaches?
5. **AskUserQuestion** if: Simplification opportunities with tradeoffs, alternative approaches, assumptions to validate

**Self-Check**:
- [ ] Complexity removal attempted?
- [ ] Code simplifier run (if available)?
- [ ] Obvious simplifications addressed?
- [ ] Tradeoffs to discuss?

**State**: Simplifications applied, tradeoffs considered, questions

**Checkpoint**: Code simplified. Proceed to Phase 7.

---

## Phase 7: Design Review

**Goal**: Optimal user interface, design system aligned.

**Process**:
1. **Visual review**: Review UI components/pages in browser
2. **Run frontend-design skill** if available
3. **Run web-design-guidelines skill** for accessibility/UX audit if available
4. **Check design system**:
   - Consistent with existing UI?
   - Following component library patterns?
   - Proper spacing, typography, colors?
   - Accessible (ARIA labels, keyboard nav, contrast)?
5. **Run code-simplifier** on UI components

**Self-Check**:
- [ ] Consistent with existing UI?
- [ ] Design skills run (if available)?
- [ ] Visual hierarchy clear?
- [ ] Components not over-nested?
- [ ] Accessible to all users?
- [ ] Design choices to discuss?

**State**: Design changes, skills run, decisions

**Checkpoint**: Design reviewed. Proceed to Phase 8.

---

## Phase 8: Comprehensive Testing

**Goal**: End-to-end validation vs ticket.

**Process**:
1. Test all verification steps from ticket
2. Test edge cases
3. Test error states
4. Verify outcomes achieved
5. **Run formatter**: Run project's code formatter (e.g., `npm run format`, `bun run format`)
6. **Run linter**: Run project's linter (e.g., `npm run lint`, `bun run lint`)
7. **Run build**: Ensure production build succeeds (e.g., `npm run build`, `bun run build`)
8. **Run tests**: Run automated test suite if it exists (e.g., `npm test`, `bun test`)

**Self-Check**:
- [ ] All verification steps pass?
- [ ] Edge cases handled?
- [ ] Error states work?
- [ ] Outcome achieved?
- [ ] Formatter run successfully?
- [ ] Linter passes?
- [ ] Build passes with no errors?
- [ ] Tests pass (if applicable)?

**Stopping**: If testing reveals fundamental issues or build fails, discuss scope/approach changes.

**State**: Verification steps completed, edge cases tested, issues fixed, code formatted, build passes

**Checkpoint**: All tests pass, code formatted, build succeeds. Proceed to Phase 9.

---

## Phase 9: Pull Request

**Goal**: Document work clearly.

**Process**:
1. Open PR on GitHub
2. Write clear title:
   - **Linear**: `feat: Description (ABC-123)` or `fix: Description (ABC-123)`
   - **GitHub**: `feat: Description (#123)` or `fix: Description (#123)`
3. Write description:

```markdown
## Summary
[What this PR does]

## Changes
- Key change 1
- Key change 2

## Architecture Decisions
- Decision: [rationale]

## Testing Performed
- [ ] Manual test 1
- [ ] Manual test 2
- [ ] Edge cases verified

## Screenshots
[Include screenshots of any new UI]

Closes ABC-123 (Linear) or Closes #123 (GitHub)
```

4. **Link issue**:
   - **GitHub issues**: Use `Closes #123` in PR body (auto-closes on merge)
   - **Linear**: Use `Closes ABC-123` (Linear integration picks this up)

**State**: PR URL, screenshots captured

**Checkpoint**: PR open. Proceed to Phase 10.

---

## Phase 10: Cleanup & Documentation

**Goal**: Prepare for future sessions.

**Process**:
1. Clean up intermediate files
2. **Update README** only if architecturally significant
3. **Update docs/** only if needed (concise, complex changes only)
4. **Update CLAUDE.md** only if needed (ONE line or link to detailed doc)
5. Mark TODOs complete

**Self-Check**:
- [ ] No intermediate files?
- [ ] Documentation appropriate (not over-documented)?
- [ ] TODO list cleared?

**Final Checkpoint**: Summarize delivery.

---

## State Tracking Throughout

Track in each phase:
- Current phase and goal
- Decisions made and rationale
- Open questions and blockers
- Files modified
- What's working vs. not

Update TODO list continuously.

## Critical Reminders

- Maintain detailed TODO list
- Use Explore agent for codebase understanding
- Self-check at each phase end
- **Available skills** (use if helpful):
  - code-simplifier (Phases 6, 7)
  - frontend-design (Phase 7)
  - web-design-guidelines (Phase 7)
  - Project-specific skills (check your project's CLAUDE.md)
- Don't kill dev server unnecessarily (hot-reload is your friend)
- SIMPLIFY at every opportunity
- No backwards compatibility unless stated
- Coordinate subagents with clear boundaries
- Track state for long-horizon reasoning
- Major checkpoints: After Phase 1, 3, 10

## GitHub Issue Commands Quick Reference

When using GitHub issues instead of Linear:

```bash
# View issue
gh issue view <number> --json title,body,labels,assignees,state

# Update issue body
gh issue edit <number> --body "new description"

# Add labels
gh issue edit <number> --add-label "in-progress"

# Add comment
gh issue comment <number> --body "Status update..."

# Close issue
gh issue close <number>

# List issues
gh issue list --label "bug" --state open

# Create issue
gh issue create --title "..." --body "..." --label "feature"
```

## Customization

To adapt this skill to your project:

1. **Update the skill description** in YAML frontmatter with your project name
2. **Create a references/ folder** with your project-specific patterns (optional):
   - Tech stack details
   - Coding standards
   - Architecture patterns
   - Common pitfalls
3. **Modify Phase 4** to reference your project's patterns file
4. **Add project-specific skills** to the Critical Reminders section

Example reference structure:

```
your-skill-name/
├── SKILL.md                    # This file (customized)
└── references/                 # Optional
    └── project-patterns.md     # Your project's patterns
```
