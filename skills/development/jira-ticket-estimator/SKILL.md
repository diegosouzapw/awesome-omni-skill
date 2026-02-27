---
name: Jira Ticket Estimator
description: This skill should be used when estimating development time for Jira tickets. It provides both manual and AI-assisted estimates with T-shirt sizes, story points, and phase-by-phase time breakdowns based on task type classification, complexity scoring, and project architecture (monolithic/serverless/frontend/fullstack/mobile/test_automation).
---

# Jira Ticket Estimator

Estimate manual and AI-assisted development time for Jira tickets using an 8-phase manual workflow and 8-phase AI-assisted workflow that mirror real development processes.

## When to Use This Skill

Invoke when:
- Estimating a Jira ticket before work starts
- Sprint planning requires story point assignments
- Stakeholders need time estimates for features/bugs/refactors
- Comparing estimates across different project types
- Understanding time savings with AI-assisted development

## Inputs

**Required** (one of):
- Jira ticket ID/URL (e.g., PROJ-123)
- Free-form description of work
- User story with acceptance criteria

**Optional Flags**:
- `--project-type`: monolithic|serverless|frontend|fullstack|mobile|test_automation (default: monolithic)
- `--task-type-override`: Manual override for task classification
- `--team-velocity`: Velocity factor for story points (default: 1.0)
- `--has-infrastructure-changes`: Flag for infrastructure changes

## Estimation Workflow

### 1. Fetch Ticket Details

Use Atlassian MCP tools to retrieve ticket information:

```bash
mcp__atlassian_fetch_jira_ticket --ticket-id=<ticket-id>
```

Extract: title, description, acceptance criteria, issue type, labels.

If MCP unavailable, prompt user for ticket details.

### 2. Classify Task Type

Classify into one of 5 types using keywords and issue type:
- **Bug Fix**: Keywords: fix, bug, error | Issue Type: Bug
- **Refactor**: Keywords: refactor, improve, optimize, harden
- **Enhancement**: Keywords: add, extend, enhance (without "new")
- **Net-New**: Keywords: new, create, build, implement
- **Spike**: Keywords: investigate, research, spike

See `references/task-type-classification.md` for complete classification guide.

### 3. Repository Reconnaissance

Scan codebase to understand scope and **COUNT FILES**:
- Use Grep/Glob to find related files
- **CRITICAL: Count unique files across all searches for file_touch_overhead**
- **Track file count to pass to estimator - affects manual workflow time significantly**
- Estimate LOC changes
- Identify integration points
- Check test coverage in affected areas

**File Counting Strategy:**
```python
# Run multiple searches
messaging_files = Grep("messagingSync", output_mode="files_with_matches")
model_files = Grep("extendables", type="php", output_mode="files_with_matches")
test_files = Glob("tests/**/*Relevant*.php")
command_files = Glob("app/Console/Commands/**/*Relevant*.php")

# Combine and deduplicate
all_files = set()
all_files.update(messaging_files or [])
all_files.update(model_files or [])
all_files.update(test_files or [])
all_files.update(command_files or [])

file_count = len(all_files)
# This file_count MUST be passed to estimator.estimate_ticket()
```

**File Count Impact (file_touch_overhead):**
- **< 20 files**: No overhead
- **20-30 files (low complexity <3.0)**: ~100-150 min overhead (1.7-2.5h)
- **30-50 files (medium complexity 3.0-6.0)**: ~150-375 min overhead (2.5-6.3h)
- **50-60 files (high complexity >6.0)**: ~450-540 min overhead (7.5-9h, may hit cap at 5h/300min)

**Why this matters**: Manual development has mechanical overhead (opening files, reading context, making changes). AI can batch process efficiently, so overhead only applies to manual workflow.

### 4. Detect Manual Time Adjustments

Scan ticket title and description for explicit time additions:

**Supported patterns:**
- `(+4h)`, `(4h)`, `(+4 hours)` - Parentheses format (recommended)
- `+4h`, `+4 hours` - Plus prefix format
- `(+30m)`, `(30m)`, `(+30 minutes)` - Minutes notation
- `+2.5h`, `+1.5 hours` - Decimal values supported

**Examples:**
- "ensure qa automation run is not affected (+4h)"
- "manual testing required +2h"
- "coordinate with DevOps team (30m)"

**Intelligent Phase Distribution:**

The system analyzes the context around each time adjustment and intelligently distributes it to the appropriate workflow phase:

| Adjustment Context | Destination Phase | Example |
|-------------------|------------------|---------|
| QA, manual testing, smoke test, verification | **Verification** (Phase 7) | "QA self run (+2h)" |
| Unit test, integration test, test coverage | **Testing** (Phase 4) | "write unit tests (+1h)" |
| Code review, peer review, PR review | **Code Review** (Phase 5) | "security review (+45m)" |
| Self review | **Self Review** (Phase 3) | "self review (+30m)" |
| Planning, design, architecture | **Planning & Design** (Phase 1) | "architecture planning (+1h)" |
| Implementation, coding, development | **Implementation** (Phase 2) | "additional coding (+2h)" |
| Deployment, deploy | **Deployment** (Phase 6) | "deployment steps (+30m)" |
| Feedback, iteration, refinement | **Feedback & Iterations** (Phase 8) | "incorporate feedback (+1h)" |
| **Unmatched** (coordination, meetings, etc.) | **Phase 9: Overhead Activities** | "coordinate with DevOps (+1h)" |

**Phase 9 only appears if there are unmatched adjustments** that don't fit into any of the standard workflow phases.

### 5. Score Complexity Factors

Score each factor on 1-10 scale:

1. **Scope Size**: Files, LOC, API changes
2. **Technical Complexity**: Algorithms, integrations
3. **Testing Requirements**: Coverage needs, test complexity
4. **Risk & Unknowns**: New tech, ambiguity, breaking changes
5. **Dependencies**: Blockers, coordination, impact radius

See `references/complexity-scoring-guide.md` for detailed scoring guide.

### 6. Calculate Estimates Using Python Script

Execute the estimation script with gathered inputs:

```bash
cd .claude/skills/jira-ticket-estimator

python3 scripts/estimator.py <<EOF
{
  "title": "<ticket_title>",
  "description": "<ticket_description>",
  "project_type": "<monolithic|serverless|frontend|fullstack>",
  "issue_type": "<Bug|Story|Task|Spike>",
  "complexity_scores": {
    "scope_size": <1-10>,
    "technical_complexity": <1-10>,
    "testing_requirements": <1-10>,
    "risk_and_unknowns": <1-10>,
    "dependencies": <1-10>
  },
  "has_infrastructure_changes": <true|false>
}
EOF
```

Or use Python directly:

```python
from scripts.estimator import TicketEstimator
import json

estimator = TicketEstimator('heuristics.json')

# Option 1: Auto-detect manual adjustments from title/description
result = estimator.estimate_ticket(
    title="<ticket_title>",
    description="Manual testing required (+2h)",  # Auto-detected
    project_type="<project_type>",
    issue_type="<issue_type>",
    complexity_scores={
        'scope_size': <score>,
        'technical_complexity': <score>,
        'testing_requirements': <score>,
        'risk_and_unknowns': <score>,
        'dependencies': <score>
    },
    has_infrastructure_changes=<bool>,
    file_count=<counted_files>  # IMPORTANT: Pass file count from reconnaissance
)

# Option 2: Explicitly pass manual adjustments (overrides auto-detection)
manual_adjustments = [
    {"context": "Manual testing", "hours": 2.0, "phase": "verification"},
    {"context": "QA coordination", "hours": 1.5, "phase": "verification"},
    {"context": "Extra planning", "hours": 1.0, "phase": "planning_design"}
]

result = estimator.estimate_ticket(
    title="<ticket_title>",
    description="<ticket_description>",
    project_type="<project_type>",
    issue_type="<issue_type>",
    complexity_scores={...},
    has_infrastructure_changes=<bool>,
    file_count=<counted_files>,
    manual_adjustments=manual_adjustments  # Explicit adjustments (NEW!)
)

print(json.dumps(result, indent=2))
```

**Manual Adjustments - Two Methods**:

1. **Auto-detection** (default): Include patterns like `(+2h)`, `+2h`, `(2h)` in title/description
2. **Explicit parameter** (new): Pass `manual_adjustments` list with context, hours, and target phase

Valid phases for adjustments:
- `verification` - Test verification phase
- `testing` - Testing phase
- `code_review` - Code review & revisions phase
- `self_review` - Self review phase
- `planning_design` - Planning & design phase
- `implementation` - Implementation phase
- `deployment_to_test` - Deployment phase
- `feedback_iterations` - Feedback & iterations phase
- `overhead` - Unmatched overhead (creates Phase 9)

Script outputs JSON with complete estimate breakdown including both manual and AI-assisted workflows, plus detected/explicit manual time adjustments.

### 7. Parse and Format Results

Extract from JSON output:
- Project type and task type classification
- T-shirt size (XS/S/M/L/XL)
- Story points (Fibonacci)
- Raw and adjusted complexity scores
- Manual workflow phase-by-phase time breakdown
- AI-assisted workflow phase-by-phase time breakdown
- Detected manual time adjustments
- Time savings comparison
- Total time (calculated and rounded)

### 8. Present Estimate to User

#### Formatting Rules for Display

Convert `total_hours_rounded` (bucket value) to days/hours format:

**Day size**: 1 day = 8 hours (configured in heuristics.json)

**Conversion formula**:
- `days = total_hours_rounded ÷ 8` (integer division)
- `remaining_hours = total_hours_rounded % 8` (modulo)

**Display format**:
- If `total_hours_rounded < 8`: Show as "Xh" (e.g., "4h", "6h")
- If `remaining_hours = 0`: Show as "Xd" (e.g., "2d" for 16h, "4d" for 32h)
- If `remaining_hours > 0`: Show as "Xd Xh" (e.g., "2d 4h" for 20h, "3d 6h" for 30h)

**Examples**:
- 32h bucket → 32 ÷ 8 = 4d (remainder 0) → **"4d (32.84h)"**
- 30h bucket → 30 ÷ 8 = 3d remainder 6h → **"3d 6h (30.17h)"**
- 6h bucket → **"6h (5.89h)"**
- 16h bucket → 16 ÷ 8 = 2d (remainder 0) → **"2d (16.51h)"**
- 20h bucket → 20 ÷ 8 = 2d remainder 4h → **"2d 4h (19.23h)"**

Format output as markdown table:

```markdown
# Estimate: <TICKET-ID>

**Project Type**: <type>
**Task Type**: <type> (<classification rationale>)
**T-Shirt Size**: <size>
**Story Points**: <points>
**Complexity**: <raw>/10 → <adjusted>/10 (scale factor: <sf>)
**Files Affected**: <files count>
**Manual Development Total**: Xd Xh (or Xh if < 8h)
- Breakdown: X.XXh (includes overhead if any) → Xh rounded
**AI-Assisted Development Total**: Xd Xh (or Xh if < 8h)
- Breakdown: X.XXh (includes overhead if any) → Xh rounded
**Time Savings**: XX.X%

## Manual Development Time Breakdown

| Phase | Time | Details |
|-------|------|---------|
| 1. Planning & Design | X min | <description> |
| 2. Implementation | X min | <task-type base unit> × <complexity> |
| 3. Self Review | 30 min | Review own code before testing |
| 4. Testing | X min | <percentage>% of implementation |
| 5. Code Review & Revisions | X min | <description> |
| 6. Deployment to Test | X min | Deploy to test environment |
| 7. Test Verification | X min | Smoke tests, verification (scales with complexity) |
| 8. Feedback & Iterations | X min | Incorporate stakeholder feedback, iterate on changes |
| **9. Overhead Activities** | **X min** | **+Xh from title, +Xm from description** (if detected) |
| **TOTAL (calculated)** | **X.XXh** | |
| **TOTAL (rounded)** | **Xh** | Snapped to bucket |

## AI-Assisted Development Time Breakdown

| Phase | Time | Details |
|-------|------|---------|
| 1. AI Planning | X min | AI generates architecture, API contracts (X% savings) |
| 2. AI Implementation | X min | AI generates code, tests (X% savings) |
| 3. AI Review | X min | AI reviews for bugs, improvements |
| 4. Human Review & Testing | X min | Validate AI output, run tests |
| 5. Iterations & Vibe Coding | X min | Fix AI mistakes, refine prompts |
| 6. Deploy to Test | X min | Deploy to test environment |
| 7. Test Verification | X min | Smoke tests, E2E verification |
| 8. Feedback & Iterations | X min | Incorporate stakeholder feedback, iterate on AI-generated changes |
| **9. Overhead Activities** | **X min** | **+Xh from title, +Xm from description** (if detected) |
| **TOTAL (calculated)** | **X.XXh** | |
| **TOTAL (rounded)** | **Xh** | Snapped to bucket |

## Complexity Scores

| Factor | Score | Rationale |
|--------|-------|-----------|
| Scope Size | X/10 | <why> |
| Technical Complexity | X/10 | <why> |
| Testing Requirements | X/10 | <why> |
| Risk & Unknowns | X/10 | <why> |
| Dependencies | X/10 | <why> |

## Assumptions

1. <assumption 1>
2. <assumption 2>
3. <assumption 3>

## Confidence Level

**<High|Medium|Low>** - <rationale>

## Next Steps

1. Review estimate with stakeholders
2. Validate complexity scores and task type
3. Approval gate: proceed, modify scope, or reject
4. Choose development approach (manual vs AI-assisted)
5. Track actual vs estimated time for calibration
```

### 9. Optional: Update Jira

If `--update-jira` flag provided:

```bash
mcp__atlassian_update_jira_issue --ticket-id=<ticket-id> --story-points=<points>
```

## Project Types

Five project types with customized workflow phases:

### Monolithic (Laravel, Rails, Django)
- Planning: 90 min @ complexity 5
- Testing: 40% of implementation
- Deployment to Test: 25 min (50 min with infra)
- Test Verification: 20 min @ complexity 5 (scales with complexity)
- Feedback & Iterations: 30 min @ complexity 5 (scales with complexity)

### Serverless (AWS Lambda, Cloud Functions)
- Planning: 120 min @ complexity 5 (more for IaC)
- Testing: 35% of implementation
- Deployment to Test: 25 min (60 min with infra)
- Test Verification: 20 min @ complexity 5 (scales with complexity)
- Feedback & Iterations: 30 min @ complexity 5 (scales with complexity)

### Frontend (React, Vue, Angular)
- Planning: 75 min @ complexity 5
- Testing: 45% of implementation (includes E2E)
- Deployment to Test: 25 min (35 min with infra)
- Test Verification: 20 min @ complexity 5 (scales with complexity)
- Feedback & Iterations: 30 min @ complexity 5 (scales with complexity)

### Full-Stack (Backend + Frontend)
- Planning: 120 min @ complexity 5
- Testing: 50% of implementation (most comprehensive)
- Deployment to Test: 25 min (60 min with infra)
- Test Verification: 20 min @ complexity 5 (scales with complexity)
- Feedback & Iterations: 30 min @ complexity 5 (scales with complexity)

### Mobile (Android, iOS, React Native, Flutter)
- Planning: 100 min @ complexity 5 (screen flows, offline support)
- Testing: 50% of implementation (device testing critical)
- Deployment to Test: 25 min (40 min with infra) - TestFlight/Internal Testing
- Test Verification: 20 min @ complexity 5 (scales with complexity)
- Feedback & Iterations: 30 min @ complexity 5 (scales with complexity)

### Test Automation (Serenity BDD, Playwright, Cypress)
- **Custom 8-Phase Workflow**: Analysis & Test Planning → Environment Setup → Page Objects & Locators → Step Implementations → Gherkin Integration → Testing & Evidence → Documentation → Feedback & Iterations
- Phase 1 - Analysis & Test Planning: 145 min @ complexity 5
- Phase 2 - Environment Setup: 63 min @ complexity 5
- Phase 3 - Page Objects & Locators: 199 min @ complexity 5
- Phase 4 - Step Implementations: Task-type based
- Phase 5 - Gherkin Integration: 136 min @ complexity 5
- Phase 6 - Testing & Evidence: 72 min @ complexity 5
- Phase 7 - Documentation: 54 min @ complexity 5
- Phase 8 - Feedback & Iterations: 35 min @ complexity 5 (scales with complexity)
- **AI-Assisted**: 45% time savings across all phases

All phase times scale with complexity. See `references/workflow-formulas.md` for complete formulas.

## The 8-9 Phase Manual Workflow

Estimates follow real development phases:

1. **Planning & Design** - Architecture, DB schema, API contracts
2. **Implementation** - Core development work
3. **Self Review** - Review own code for bugs, edge cases, code quality
4. **Testing** - Unit, integration, E2E tests
5. **Code Review & Revisions** - Peer review, addressing feedback
6. **Deployment to Test** - Deploy to test environment (fixed time, infra-aware)
7. **Test Verification** - Smoke tests, verification (scales with complexity)
8. **Feedback & Iterations** - Incorporate stakeholder feedback, iterate on changes, address edge cases found in verification
9. **Overhead Activities** - Unmatched overhead from ticket (optional, only if detected)

**Intelligent Adjustment Distribution**:
When time adjustments like `(+4h)` are detected, they're intelligently distributed to the appropriate phase based on context:
- "QA self run (+2h)" → **Test Verification** phase
- "Code review with security team (+45m)" → **Code Review & Revisions** phase
- "Coordinate with DevOps (+1h)" → **Phase 9** (unmatched overhead)

**Phase 9 only appears if there are unmatched adjustments** that don't fit into standard workflow phases (e.g., coordination, meetings, external dependencies).

## The 8-9 Phase AI-Assisted Workflow

AI-assisted development follows these phases:

1. **AI Planning** - AI generates architecture, DB schema, API contracts (60-70% time savings)
2. **AI Implementation** - AI generates code, controllers, models, tests (60-70% time savings)
3. **AI Review** - AI reviews generated code for bugs and improvements
4. **Human Review & Testing** - Developer validates AI output and runs tests
5. **Iterations & Vibe Coding** - Fix AI mistakes, regenerate code, refine prompts
6. **Deploy to Test** - Deploy to test environment
7. **Test Verification** - Smoke tests, E2E verification, validation
8. **Feedback & Iterations** - Incorporate stakeholder feedback, iterate on AI-generated changes, refine based on verification
9. **Overhead Activities** - Unmatched overhead from ticket (optional, only if detected)

**Intelligent Adjustment Distribution**:
Same intelligent distribution logic applies to AI-assisted workflow. Adjustments are added to their appropriate phases based on context.

**Note**: Overhead activities (Phase 9) are the same for manual and AI-assisted development. Only the core workflow phases (1-8) benefit from AI acceleration.

Typical time savings: 40-50% compared to manual development.

## Configuration

All parameters stored in `heuristics.json`:

### Customizable per Project Type
- Workflow phase base times (manual and AI-assisted)
- Testing percentages
- Deploy times with/without infrastructure changes
- AI time savings percentages

### Overhead Activities (Phase 9)
- Automatic detection of explicit time additions in ticket content
- Supports patterns: `(+4h)`, `+2h`, `(30m)`, `+15 minutes`
- Supports hours and minutes notation
- Supports decimal values: `+2.5h`
- All detected adjustments are summed and added as Phase 9 to both workflows
- Appears in phase breakdown tables when detected
- Can be disabled by setting `"enabled": false` in heuristics.json

### Bucket Rounding

Final estimates are snapped to standardized time buckets using a threshold-based approach.

**Buckets (hours)**: 0, 1, 2, 3, 4, 6, 8 (1d), 12 (1.5d), 16 (2d), 20 (2.5d), 24 (3d), 28 (3.5d), 32 (4d), 36 (4.5d), 40 (5d)

**Day size**: 1d = 8h

**Rule**: Compute the precise grand total (workflow phases 1-9, which includes overhead in Phase 9 if detected), then snap to the nearest bucket using a **threshold-based approach**.
- For each bucket, calculate the threshold as **current + 75% of gap to next bucket**
- If calculated total ≤ threshold, stay at current bucket; otherwise, jump to next bucket
- If result > 5d (40h) → Split the scope (ticket too large)

**Formula**: `threshold = currentBucket + ((nextBucket - currentBucket) × 0.75)`

#### Threshold Table

| Current Bucket | Next Bucket | Gap | Threshold | Range                       |
| -------------- | ----------- | --- | --------- | --------------------------- |
| 0h             | 1h          | 1h  | 0.63h     | 0-0.63h → 0h                |
| 1h             | 2h          | 1h  | 1.63h     | 0.64-1.63h → 1h             |
| 2h             | 3h          | 1h  | 2.63h     | 1.64-2.63h → 2h             |
| 3h             | 4h          | 1h  | 3.63h     | 2.64-3.63h → 3h             |
| 4h             | 6h          | 2h  | 5h        | 3.64-5h → 4h                |
| 6h             | 8h          | 2h  | 7h        | 5.01-7h → 6h                |
| 8h             | 12h         | 4h  | 10h       | 7.01-10h → 8h               |
| 12h            | 16h         | 4h  | 14h       | 10.01-14h → 12h             |
| 16h            | 20h         | 4h  | 18h       | 14.01-18h → 16h             |
| 20h            | 24h         | 4h  | 22h       | 18.01-22h → 20h             |
| 24h            | 28h         | 4h  | 26h       | 22.01-26h → 24h             |
| 28h            | 32h         | 4h  | 30h       | 26.01-30h → 28h             |
| 32h            | 36h         | 4h  | 34h       | 30.01-34h → 32h             |
| 36h            | 40h         | 4h  | 38h       | 34.01-38h → 36h             |
| 40h            | -           | -   | -         | 38.01h+ → 40h (split scope) |

**Examples**:
- **4.07h** → threshold for 4h is 5h → 4.07 ≤ 5 → **stays at 4h** ✓
- **5.5h** → threshold for 4h is 5h → 5.5 > 5 → **jumps to 6h**
- **12.59h** → threshold for 12h is 14h → 12.59 ≤ 14 → **stays at 12h** ✓
- **14.5h** → threshold for 12h is 14h → 14.5 > 14 → **jumps to 16h**
- **16.5h** → threshold for 16h is 18h → 16.5 ≤ 18 → **stays at 16h** ✓
- **23.2h** → threshold for 20h is 22h → 23.2 > 22 → **jumps to 24h**

When presenting estimates, show both:
- Calculated total (e.g., 4.07h)
- Rounded total (e.g., 4h)
- Note: "snapped to bucket (threshold: Xh)"

### Global Settings
- Task type definitions and keywords
- Complexity factor scoring guides
- Complexity weights by task type
- T-shirt sizing ranges
- Story points mapping
- Bucket rounding thresholds (see Bucket Rounding section above)

### Calibration

To customize for your team:
1. Start with defaults (monolithic)
2. Track estimated vs actual for 10-20 tickets
3. Adjust phase base times in `heuristics.json`
4. Recalibrate quarterly

Example adjustment:
```json
{
  "monolithic": {
    "workflow_phases": {
      "planning_design": {
        "base_minutes_at_complexity_5": 120
      }
    }
  }
}
```

## Bundled Resources

- `heuristics.json` - Complete estimation configuration
- `scripts/estimator.py` - Python calculation engine
- `references/task-type-classification.md` - Detailed task type guide
- `references/complexity-scoring-guide.md` - Factor-by-factor scoring guide
- `references/workflow-formulas.md` - Complete formulas and examples

## Example Usage

**User**: "Estimate LOOM-4156 for our Laravel monolith"

**Agent**:
1. Fetch ticket via MCP
2. Classify as "Bug Fix" (keywords: "fix", "error")
3. Scan repository - find LoginController.php
4. Score complexity: Scope=2, Technical=2, Testing=4, Risk=2, Dependencies=3
5. Execute estimator script
6. Present formatted estimate:

```markdown
# Estimate: LOOM-4156

**Project Type**: Monolithic Application
**Task Type**: Bug Fix (keywords: "fix", "validation", "error")
**T-Shirt Size**: XS
**Story Points**: 1
**Complexity**: 3.0/10 → 1.26/10 (scale factor: 0.252)

## Manual Development Time Breakdown

| Phase | Time | Details |
|-------|------|---------|
| 1. Planning & Design | 23 min | Architecture review, edge cases |
| 2. Implementation | 16 min | Fix validation logic |
| 3. Self Review | 30 min | Review own code before testing |
| 4. Testing | 6 min | Unit tests for validation |
| 5. Code Review & Revisions | 11 min | Peer review |
| 6. Deployment to Test | 25 min | Deploy to test environment |
| 7. Verification | 5 min | Smoke test, verify fix |
| **Total (calculated)** | **1.93h** | |
| **Total (rounded)** | **2h** | Snapped to bucket |

## AI-Assisted Development Time Breakdown

| Phase | Time | Details |
|-------|------|---------|
| 1. AI Planning | 7 min | AI generates fix approach (70% savings) |
| 2. AI Implementation | 6 min | AI generates validation fix (65% savings) |
| 3. AI Review | 5 min | AI reviews for edge cases |
| 4. Human Review & Testing | 8 min | Validate fix, run tests |
| 5. Iterations & Vibe Coding | 3 min | Refine if needed |
| 6. Deploy to Test | 25 min | Deploy to test environment |
| 7. Test Verification | 10 min | Verify fix works |
| **Total (calculated)** | **1.15h** | |
| **Total (rounded)** | **1h** | Snapped to bucket |

## Time Savings

- **Manual Development**: 1.93h → 2h rounded
- **AI-Assisted Development**: 1.15h → 1h rounded
- **Time Savings**: 1h (50% faster)
```

7. Track in backlog for calibration

## Estimation Checklist

Before calling `estimator.estimate_ticket()`, ensure you have:

- [ ] Fetched ticket details (title, description, issue type)
- [ ] Classified task type (bug_fix, refactor, enhancement, net_new, spike)
- [ ] Scored all 5 complexity factors (1-10 scale)
- [ ] **Counted unique files to be modified** (CRITICAL for file_touch_overhead)
- [ ] Detected infrastructure changes (boolean)
- [ ] Reviewed ticket for manual time adjustments

**File Count Sources:**
- Files from Grep searches (deduplicated)
- Files from Glob patterns (deduplicated)
- Test files to be modified
- New files to be created
- Artisan commands/scripts affected
- **Always remove duplicates across searches**

**Common mistake**: Forgetting to pass `file_count` parameter results in underestimating manual workflow by 2-5 hours for large refactors!

## Notes

- **Estimates are predictive** - refine after reconnaissance
- **Both workflows calculated automatically** - choose based on your development approach
- **AI-assisted estimates assume**: Effective prompt engineering, experienced with AI tools, quality review process
- **Manual time adjustments detected automatically** - use `(+Xh)` or `+Xh` format in ticket content
- **File touch overhead is CRITICAL** - always count files during reconnaissance, adds 2.5 min/file to manual workflow
- **Track actual vs estimated** by phase and adjustments for calibration
- **Adjust `heuristics.json`** based on team historical data
- **Use `--update-jira`** to persist story points automatically
- **Open-source ready** - all team-specific configs in heuristics.json
