---
name: workflows-review
description: Perform exhaustive code reviews using multi-agent analysis, ultra-thinking, and worktrees
---

## Arguments
[PR number, GitHub URL, branch name, or latest]

# Review Command

<command_purpose> Perform exhaustive code reviews using multi-agent analysis, ultra-thinking, and Git worktrees for deep local inspection. </command_purpose>

## Introduction

<role>Senior Code Review Architect with expertise in security, performance, architecture, and quality assurance</role>

## Prerequisites

<requirements>
- Git repository with GitHub CLI (`gh`) installed and authenticated
- Clean main/master branch
- Proper permissions to create worktrees and access the repository
- For document reviews: Path to a markdown file or document
</requirements>

## Main Tasks

### 1. Determine Review Target & Setup (ALWAYS FIRST)

<review_target> #$ARGUMENTS </review_target>

<thinking>
First, I need to determine the review target type and set up the code for analysis.
</thinking>

#### Immediate Actions:

<task_list>

- [ ] Determine review type: PR number (numeric), GitHub URL, file path (.md), or empty (current branch)
- [ ] Check current git branch
- [ ] If ALREADY on the target branch (PR branch, requested branch name, or the branch already checked out for review) → proceed with analysis on current branch
- [ ] If DIFFERENT branch than the review target → offer to use worktree: "Use git-worktree skill for isolated Call `/git-worktree` skill with branch name
- [ ] Fetch PR metadata using `gh pr view --json` for title, body, files, linked issues
- [ ] Set up language-specific analysis tools
- [ ] Prepare security scanning environment
- [ ] Make sure we are on the branch we are reviewing. Use gh pr checkout to switch to the branch or manually checkout the branch.

Ensure that the code is ready for analysis (either in worktree or on current branch). ONLY then proceed to the next step.

</task_list>

#### Protected Artifacts

<protected_artifacts>
The following paths are compound-engineering pipeline artifacts and must never be flagged for deletion, removal, or gitignore by any review agent:

- `docs/plans/*.md` — Plan files created by `/workflows-plan`. These are living documents that track implementation progress (checkboxes are checked off by `/workflows-work`).
- `docs/solutions/*.md` — Solution documents created during the pipeline.

If a review agent flags any file in these directories for cleanup or removal, discard that finding during synthesis. Do not create a task for it.
</protected_artifacts>

#### Parallel subagents to review the PR:

<parallel_tasks>

Run ALL or most of these as subagents in parallel:

1. git-history-analyzer(PR content)
2. analyze-dependencies(PR content)
3. analyze-patterns(PR content)
4. analyze-architecture(PR content)
5. review-security(PR content)
6. analyze-performance(PR content)
7. review-data-integrity(PR content)
8. review-agent-native(PR content) - Verify new features are agent-accessible

</parallel_tasks>

#### Conditional skills (Run if applicable):

<conditional_skills>

These skills are run ONLY when the PR matches specific criteria. Check the PR files list to determine if they apply:

**If PR contains database migrations or data backfills:**

14. review-data-migrations(PR content) - Validates ID mappings match production, checks for swapped values, verifies rollback safety
15. verify-deployment(PR content) - Creates Go/No-Go deployment checklist with SQL verification queries

**When to run migration skills:**
- PR includes files matching `db/migrate/*.rb`
- PR modifies columns that store IDs, enums, or mappings
- PR includes data backfill scripts or rake tasks
- PR changes how data is read/written (e.g., changing from FK to string column)
- PR title/body mentions: migration, backfill, data transformation, ID mapping

**What these skills check:**
- `review-data-migrations`: Verifies hard-coded mappings match production reality (prevents swapped IDs), checks for orphaned associations, validates dual-write patterns
- `verify-deployment`: Produces executable pre/post-deploy checklists with SQL queries, rollback procedures, and monitoring plans

</conditional_skills>

### 4. Ultra-Thinking Deep Dive Phases

For each phase below, spend maximum cognitive effort. Think step by step. Consider all angles. Question assumptions. And bring all reviews in a synthesis to the user.

<deliverable>
Complete system context map with component interactions
</deliverable>

#### Phase 3: Stakeholder Perspective Analysis

<thinking_prompt>Put yourself in each stakeholder's shoes. What matters to them? What are their pain points? </thinking_prompt>

<stakeholder_perspectives>

1. **Developer Perspective** <questions>

   - How easy is this to understand and modify?
   - Are the APIs intuitive?
   - Is debugging straightforward?
   - Can I test this easily? </questions>

2. **Operations Perspective** <questions>

   - How do I deploy this safely?
   - What metrics and logs are available?
   - How do I troubleshoot issues?
   - What are the resource requirements? </questions>

3. **End User Perspective** <questions>

   - Is the feature intuitive?
   - Are error messages helpful?
   - Is performance acceptable?
   - Does it solve my problem? </questions>

4. **Security Team Perspective** <questions>

   - What's the attack surface?
   - Are there compliance requirements?
   - How is data protected?
   - What are the audit capabilities? </questions>

5. **Business Perspective** <questions>
   - What's the ROI?
   - Are there legal/compliance risks?
   - How does this affect time-to-market?
   - What's the total cost of ownership? </questions> </stakeholder_perspectives>

#### Phase 4: Scenario Exploration

<thinking_prompt> Explore edge cases and failure scenarios. What could go wrong? How does the system behave under stress? </thinking_prompt>

<scenario_checklist>

- [ ] **Happy Path**: Normal operation with valid inputs
- [ ] **Invalid Inputs**: Null, empty, malformed data
- [ ] **Boundary Conditions**: Min/max values, empty collections
- [ ] **Concurrent Access**: Race conditions, deadlocks
- [ ] **Scale Testing**: 10x, 100x, 1000x normal load
- [ ] **Network Issues**: Timeouts, partial failures
- [ ] **Resource Exhaustion**: Memory, disk, connections
- [ ] **Security Attacks**: Injection, overflow, DoS
- [ ] **Data Corruption**: Partial writes, inconsistency
- [ ] **Cascading Failures**: Downstream service issues 

</scenario_checklist>

### 6. Multi-Angle Review Perspectives

#### Technical Excellence Angle

- Code craftsmanship evaluation
- Engineering best practices
- Technical documentation quality
- Tooling and automation assessment

#### Business Value Angle

- Feature completeness validation
- Performance impact on users
- Cost-benefit analysis
- Time-to-market considerations

#### Risk Management Angle

- Security risk assessment
- Operational risk evaluation
- Compliance risk verification
- Technical debt accumulation

#### Team Dynamics Angle

- Code review etiquette
- Knowledge sharing effectiveness
- Collaboration patterns
- Mentoring opportunities

### 4. Simplification and Minimalism Review

Run the Task review-code-simplicity() to see if we can simplify the code.

### 5. Findings Synthesis and Task Creation Using Tasks Core

<critical_requirement> ALL findings MUST be stored using the `tasks-core` skill via `/tasks` operations. It auto-detects and validates the backend. Create tasks immediately after synthesis - do NOT present findings for user approval first. </critical_requirement>

#### Step 1: Synthesize All Findings

<thinking>
Consolidate all agent reports into a categorized list of findings.
Remove duplicates, prioritize by severity and impact.
</thinking>

<synthesis_tasks>

- [ ] Collect findings from all parallel agents
- [ ] Discard any findings that recommend deleting or gitignoring files in `docs/plans/` or `~/docs/solutions/` (see Protected Artifacts above)
- [ ] Categorize by type: security, performance, architecture, quality, etc.
- [ ] Assign severity levels: 🔴 CRITICAL (P1), 🟡 IMPORTANT (P2), 🔵 NICE-TO-HAVE (P3)
- [ ] Remove duplicate or overlapping findings
- [ ] Estimate effort for each finding (Small/Medium/Large)

</synthesis_tasks>

#### Step 2: Create Tasks Using `/tasks`

<critical_instruction> Use `/tasks` (backed by `tasks-core`) to create tasks for ALL findings immediately. Do NOT present findings one-by-one asking for user approval. Create all tasks in parallel, then summarize results to user. </critical_instruction>

**Implementation Options:**

**Option A: Direct Creation (Fast)**

- Create tasks directly using the selected system
- All findings in parallel for speed
- Follow the `tasks-core` contract and normalized output.

**Option B: Subagents in Parallel (Recommended for Scale)** For large PRs with 15+ findings, use subagents to create finding files in parallel:

```bash
# Launch multiple tasks in parallel
- Create tasks for first finding
- Create tasks for second finding
- Create tasks for third finding
etc. for each finding.
```

Subagents can:

- Process multiple findings simultaneously
- Write detailed task files with all sections filled
- Organize findings by severity
- Create comprehensive Proposed Solutions
- Add acceptance criteria and work logs
- Complete much faster than sequential processing

**Execution Strategy:**

1. Synthesize all findings into categories (P1/P2/P3)
2. Group findings by severity
3. Launch 3 parallel subagents (one per severity level)
4. Each subagent creates its batch of tasks using `/tasks`
5. Consolidate results and present summary

**Process (Using `/tasks`):**

1. For each finding:

   - Determine severity (P1/P2/P3)
   - Write detailed Problem Statement and Findings
   - Create 2-3 Proposed Solutions with pros/cons/effort/risk
   - Estimate effort (Small/Medium/Large)
   - Add acceptance criteria and work log

2. Use `/tasks` for structured task management:

   ```bash
   /tasks detect
   /tasks create --title "..." --priority "P1|P2|P3" --type "bug|feature|task|docs|question|epic" --description "..."
   /tasks update --id <id> --notes "Findings and technical details" --acceptance-criteria "- [ ] ..."
   ```

   `tasks-core` provides routing, naming/field mapping, and backend-specific persistence.

3. Create tasks in parallel and capture returned IDs in your summary.

4. Follow `tasks-core` for backend-specific template structure and field mapping.

**Task Structure (normalized across backends):**

Each task must include:

- **Identity/metadata**: backend task ID, status, priority, tags, dependencies
- **Problem Statement**: What's broken/missing, why it matters
- **Findings**: Discoveries from agents with evidence/location
- **Proposed Solutions**: 2-3 options, each with pros/cons/effort/risk
- **Recommended Action**: (Filled during triage, leave blank initially)
- **Technical Details**: Affected files, components, database changes
- **Acceptance Criteria**: Testable checklist items
- **Work Log**: Dated record with actions and learnings
- **Resources**: Links to PR, issues, documentation, similar patterns

Use backend-specific field mapping through `tasks-core` and its backend guides.

**Status values:**

- `pending` - New findings, needs triage/decision
- `ready` - Approved by manager, ready to work
- `complete` - Work finished

**Priority values:**

- `P1/P2/P3` at input level; `tasks-core` maps to backend-specific values.

**Tagging:** Always add `code-review` tag, plus: `security`, `performance`, `architecture`, `rails`, `quality`, etc.

#### Step 3: Summary Report

After creating all tasks, present comprehensive summary:

````markdown
## ✅ Code Review Complete

**Review Target:** PR #XXXX - [PR Title] **Branch:** [branch-name]

### Findings Summary:

- **Total Findings:** [X]
- **🔴 CRITICAL (P1):** [count] - BLOCKS MERGE
- **🟡 IMPORTANT (P2):** [count] - Should Fix
- **🔵 NICE-TO-HAVE (P3):** [count] - Enhancements

### Created Tasks (via `/tasks`):

**P1 - Critical (BLOCKS MERGE):**

- `{task-id}` - {description}

**P2 - Important:**

- `{task-id}` - {description}

**P3 - Nice-to-Have:**

- `{task-id}` - {description}

### Review Agents Used:

- review-security
- analyze-performance
- analyze-architecture
- review-agent-native
- review-code-simplicity
- [other agents]

### Next Steps:

1. **Address P1 Findings**: CRITICAL - must be fixed before merge

   - Review each P1 task in detail
   - Implement fixes or request exemption
   - Verify fixes before merging PR

2. **Triage All Tasks**:
   ```bash
   /triage  # Uses /tasks operations through tasks-core routing
   ```
````

3. **Work on Approved Tasks**:

   ```bash
   /resolve_task_parallel  # Fix all approved items efficiently
   ```

4. **Track Progress**:
   - Use `/tasks claim --id <id>` before implementation
   - Use `/tasks update --id <id> ...` for work-log notes and status details
   - Use `/tasks close --id <id> --reason "Completed"` to resolve

### Severity Breakdown:

**🔴 P1 (Critical - Blocks Merge):**

- Security vulnerabilities
- Data corruption risks
- Breaking changes
- Critical architectural issues

**🟡 P2 (Important - Should Fix):**

- Performance issues
- Significant architectural concerns
- Major code quality problems
- Reliability issues

**🔵 P3 (Nice-to-Have):**

- Minor improvements
- Code cleanup
- Optimization opportunities
- Documentation updates

```

### 7. End-to-End Testing (Optional)

<detect_project_type>

**First, detect the project type from PR files:**

| Indicator (examples in changed files) | Project Type |
|-----------|--------------|
| `package.json`, `bun.lockb`, `pnpm-lock.yaml`, `yarn.lock`, frontend paths (`app/views/*`, `src/pages/*`, `app/*`, `public/*`), UI files (`*.tsx`, `*.jsx`, `*.html.*`, `*.css`, `*.scss`) | Web |
| iOS app signals such as `*.xcodeproj`, `*.xcworkspace`, iOS simulator/build changes, UIKit/SwiftUI mobile flows | iOS |
| Native macOS desktop app signals such as AppKit/NSWindow menu/window interactions, desktop app targets, macOS UI flows | macOS Native |
| `go.mod`, `pyproject.toml`, `requirements*.txt`, `Cargo.toml`, backend paths (`api/*`, `server/*`, `services/*`, `internal/*`, `cmd/*`), files (`*.go`, `*.py`, `*.rs`, `*.rb`) | Backend/API |
| `scripts/*.sh`, `Makefile`, `.github/workflows/*`, infra/config-heavy PRs with no UI/mobile indicators | CLI/Infra |
| Signals from 2+ types above (for example Web + Backend, iOS + Backend, Web + iOS, macOS + Backend) | Hybrid |

If multiple categories are touched, treat it as `Hybrid` and offer each relevant test path.

</detect_project_type>

<offer_testing>

After presenting the Summary Report, offer appropriate testing based on project type:

**For Web Projects:**
```markdown
**"Want to run browser tests on the affected pages?"**
1. Yes - run `/test-browser`
2. No - skip
```

**For iOS Projects:**
```markdown
**"Want to run Xcode simulator tests on the app?"**
1. Yes - run `/test-xcode`
2. No - skip
```

**For macOS Native Projects:**
```markdown
**"Want to run native macOS UI tests on the desktop app?"**
1. Yes - run `/test-mac`
2. No - skip
```

**For Backend/API Projects:**
```markdown
**"Want to run backend verification for the affected services?"**
1. Yes - run repo-native targeted tests/checks for changed language areas (Python/Go/Rust/Ruby/TS) or `/prove-it`
2. No - skip
```

**For CLI/Infra Projects:**
```markdown
**"Want to run CLI/infra verification on changed commands and automation?"**
1. Yes - run command smoke tests (and `/prove-it` when useful)
2. No - skip
```

**For Hybrid Projects (choose based on touched combination):**
```markdown
**"Want to run end-to-end tests?"**
1. Web only - run `/test-browser`
2. iOS only - run `/test-xcode`
3. macOS only - run `/test-mac`
4. Backend/CLI only - run repo-native tests/checks or `/prove-it`
5. All relevant paths - run all applicable options
6. No - skip
```

</offer_testing>

#### If User Accepts Web Testing:

Spawn a subagent to run browser tests (preserves main context):

```
Task general-purpose("Run /test-browser for PR #[number]. Test all affected pages, check for console errors, handle failures by creating tasks and fixing.")
```

The subagent will:
1. Identify pages affected by the PR
2. Navigate to each page and capture snapshots (using agent-browser CLI)
3. Check for console errors
4. Test critical interactions
5. Pause for human verification on OAuth/email/payment flows
6. Create P1 tasks for any failures
7. Fix and retry until all tests pass

**Standalone:** `/test-browser [PR number]`

#### If User Accepts iOS Testing:

Spawn a subagent to run Xcode tests (preserves main context):

```
Task general-purpose("Run /test-xcode for scheme [name]. Build for simulator, install, launch, take screenshots, check for crashes.")
```

The subagent will:
1. Verify XcodeBuildMCP is installed
2. Discover project and schemes
3. Build for iOS Simulator
4. Install and launch app
5. Take screenshots of key screens
6. Capture console logs for errors
7. Pause for human verification (Sign in with Apple, push, IAP)
8. Create P1 tasks for any failures
9. Fix and retry until all tests pass

**Standalone:** `/test-xcode [scheme]`

#### If User Accepts macOS Native Testing:

Spawn a subagent to run native macOS tests (preserves main context):

```
Task general-purpose("Run /test-mac for the affected desktop app flows. Use Peekaboo to validate windows/menus/interactions, capture artifacts, and create tasks for failures.")
```

The subagent will:
1. Verify Peekaboo is installed with required permissions
2. Resolve target app/window and capture baseline UI
3. Run key desktop flows (navigation, primary action, settings, close/reopen)
4. Capture screenshots/annotated artifacts for each major step
5. Create P1 tasks for critical failures and retry when fixes are applied

**Standalone:** `/test-mac [app name|bundle id|frontmost]`

#### If User Accepts Backend/API Verification:

Spawn a subagent to run backend verification (preserves main context):

```
Task general-purpose("Run backend verification for PR #[number]. Detect changed backend languages and modules, run targeted tests/checks, inspect failures, create tasks for issues, and re-run until passing.")
```

The subagent will:
1. Identify changed backend languages and services (Python, Go, Rust, Ruby, TypeScript/Node)
2. Run targeted tests/checks for touched areas using repo-native commands
3. Use `/prove-it` when command selection is ambiguous or cross-service validation is needed
4. Capture errors/logs and create P1 tasks for critical failures
5. Fix and retry until checks pass or blockers are clearly documented

**Standalone:** `/prove-it [feature-or-change]`

#### If User Accepts CLI/Infra Verification:

Spawn a subagent to run CLI/infra verification (preserves main context):

```
Task general-purpose("Run CLI/infra verification for PR #[number]. Smoke test changed commands/scripts, validate CI/workflow-impacting changes, capture failures, and iterate to green.")
```

The subagent will:
1. Identify changed scripts, command entry points, and CI/workflow files
2. Run smoke tests for changed commands with realistic arguments
3. Validate critical automation paths impacted by the PR
4. Capture failures and create P1 tasks for critical breakages
5. Fix and retry until checks pass or remaining risk is explicit

**Standalone:** `/prove-it [feature-or-change]`

### Important: P1 Findings Block Merge

Any **🔴 P1 (CRITICAL)** findings must be addressed before merging the PR. Present these prominently and ensure they're resolved before accepting the PR.
```
