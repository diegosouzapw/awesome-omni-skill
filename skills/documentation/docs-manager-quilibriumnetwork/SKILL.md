---
name: "docs-manager"
description: "Automatically manages bugs, tasks, documentation, and reports in the .agents/ folder following project conventions. Activates when creating bug reports, documenting features, tracking tasks, generating reports, audits, researches, analyses, updating existing documentation, or organizing work in the established .agents workflow structure."
allowed-tools: ["Read", "Write", "Edit", "Glob", "Grep", "TodoWrite", "Bash"]
---

# Documentation Manager

Automatically manages bugs, tasks, documentation, and reports in the `.agents/` folder following project conventions and established workflow patterns.

## Description

This skill automates the creation, updating, and organization of documentation files in the `.agents/` folder structure following the established workflow from `agents-workflow.md`. It handles bugs, tasks, feature documentation, and reports (including audits, researches, and analyses) with appropriate templates, naming conventions, and folder organization.

**Use this skill when the user mentions:**
- Creating bug reports, tracking issues, documenting problems, debugging
- Creating tasks, planning features, tracking work, implementation planning
- Creating documentation, documenting features, explaining architecture
- Creating reports, audits, researches, analyses, investigations, assessments
- Updating or organizing existing bugs/tasks/docs/reports
- Moving completed work between folders (.done/.solved/.archived)
- Following the established .agents workflow

**Automatically triggers on phrases like:**
- "create a doc...
- "create a bug report..."
- "create a task..."
- "create a report..."
- "create an audit..."
- "audit..."
- "generate an audit..."
- "perform audit..."
- "conduct audit..."
- "security audit..."
- "code audit..."
- "document this research..."
- "analyze this..."
- "investigate..."
- "assess..."
- "document this feature..."
- "track this task..."
- "file this issue..."
- "add to .agents..."
- "move to .done..."
- "update the bug report..."
- "update the doc..."
- "update the task..."
- "update the report..."
- "update the audit..."

## Skill Coordination

This skill focuses on **document creation and content management** within the `.agents` folder structure. For folder structure changes and system integration, the `agents-folder-manager` skill handles infrastructure modifications.

**Coordination with agents-folder-manager:**
- Relies on existing folder structure for document placement
- May request folder structure changes for new content types
- Benefits from folder manager's template updates when new folders are added

## Core Capabilities

### 1. Bug Report Management
- Creates detailed bug reports in `.agents/bugs/`
- Follows established template with symptoms, root cause, solution
- Uses kebab-case naming: `feature-specific-bug-description.md`
- Includes required AI-generated warning
- Automatically moves to `.solved/` when bug is fixed

### 2. Task Creation & Management
- Creates comprehensive task files in `.agents/tasks/`
- Follows complexity-based templates (Trivial/Low/Medium/High/Critical)
- Includes proper status tracking, file references, verification steps
- Updates existing tasks with current codebase state
- Manages task lifecycle: **open** → **in-progress** → **on-hold** → **done**

### 3. Documentation Creation
- Creates feature documentation in `.agents/docs/features/`
- Follows architectural documentation patterns
- Includes integration details, technical decisions, limitations
- Cross-references related documentation and components

### 4. Report Management
- Creates reports, audits, researches, and analyses in `.agents/reports/`
- Supports various report types: security audits, feature assessments, research findings
- Follows structured reporting templates with executive summaries
- Includes methodology, findings, recommendations, and action items
- Cross-references with related tasks, bugs, and documentation

### 5. Index Automation
- **Automatically runs update-index.py** after any file operation
- **Executes yarn scan-docs** for comprehensive project documentation sync
- Maintains synchronized INDEX.md reflecting current .agents state
- Handles file moves between folders (.done, .solved, .archived)
- Updates cross-references and directory organization
- Preserves numeric ordering and proper categorization

### 6. Workflow Integration
- Respects folder structure: bugs/, tasks/, docs/, reports/ with .done/.solved/.archived subfolders
- Maintains consistent naming conventions and templates
- Preserves completed work and implementation notes
- Updates cross-references and maintains documentation index

## Instructions

When the user requests documentation management, follow this workflow:

**🔄 IMPORTANT**: Always run the index update script after any file operation to keep INDEX.md synchronized with the current state of the .agents directory.

### Step 1: Determine Document Type
Analyze the request to identify:
- **Bug Report**: Error conditions, unexpected behavior, debugging needed
- **Task**: Implementation work, feature development, specific changes needed
- **Documentation**: Architecture, feature explanation, technical guidance
- **Report**: Audits, research findings, analysis results, assessments, investigations

### Step 2: Apply Appropriate Template

**IMPORTANT**:
- Never include time estimates or schedules in tasks (e.g., "1 hour", "1 day", "30 minutes"). Tasks will primarily be completed by AI agents which work much faster than human estimates. Use complexity levels only: Trivial, Low, Medium, High, Critical.
- For complex tasks (Medium/High/Critical), Claude should first review the entire .agents folder context including INDEX.md, AGENTS.md, agents-workflow.md, existing tasks, and related documentation to understand established patterns and avoid duplicating solutions.
- Complex tasks (Medium/High/Critical complexity) should always be analyzed by the feature-analyzer agent before implementation to avoid over-engineering and ensure best practices.
- Tasks involving security considerations (authentication, encryption, user data, network communications, permissions) should be analyzed by the security-analyst agent before implementation.

**Status System** (applies to all file types):
- **`open`**: Not started yet, ready to work on (**ALWAYS the default for new tasks/bugs**)
- **`in-progress`**: **ONLY** when actively implementing a task right now, or when a task was partially implemented
- **`on-hold`**: Blocked by external factors, waiting on dependencies, or paused due to technical blockers that prevent full implementation
- **`done`**: Completed, fixed, or finalized (default for documentation and reports)
- **`archived`**: Task/bug is no longer relevant, superseded, or abandoned (only for files in `.archived/` folders)

**IMPORTANT - Task Status Rules:**
- When **creating** a new task → ALWAYS use `status: open`
- When **actively working** on implementation → change to `status: in-progress`
- When **blocked** (missing dependencies, external blockers, can't complete fully) → use `status: on-hold`
- When **completed** → use `status: done` and move to `.done/` folder
- When **archiving** (no longer relevant, superseded, abandoned) → use `status: archived` and move to `.archived/` folder
- **Never** create a new task with `status: in-progress` unless you are immediately implementing it
- **Note**: You rarely need to set `archived` status unless explicitly asked to archive a task/bug

**Complexity System** (tasks only):
- **`low`**: Simple changes, 1-2 files, clear solution
- **`medium`**: Moderate complexity, multiple files, some design decisions
- **`high`**: Complex feature, many files, significant architectural decisions
- **`very-high`**: Critical infrastructure changes, system-wide impact, major refactoring

**Priority System** (bugs only):
- **`low`**: Minor issue, workaround available, low impact
- **`medium`**: Moderate impact, affects some users, should be fixed soon
- **`high`**: Significant impact, affects many users, needs prompt attention
- **`critical`**: Severe impact, breaks core functionality, immediate fix required

**Note**: Documentation and report files should use `status: done` since they represent finalized documentation and completed analysis/audit reports.

**Folder-Based Status Rules** (CRITICAL):
When moving files between folders, the status MUST be updated to match the folder's purpose:

- **Moving to `.done/` folder** (tasks):
  - Update `status: done`
  - Update `updated` date to current date

- **Moving to `.solved/` folder** (bugs):
  - Update `status: done`
  - Update `updated` date to current date

- **Moving to `.archived/` folder** (tasks or bugs):
  - Update `status: archived`
  - Update `updated` date to current date
  - Archived items are no longer relevant, superseded, or abandoned but preserved for reference

**Default Status by Location**:
- Files in `.done/` or `.solved/` folders → `status: done`
- Files in `.archived/` folders → `status: archived`
- Files in root task folders → `status: open` (or `in-progress` if actively being implemented, or `on-hold` if blocked)
- Files in root bug folders → `status: open`
- Files in docs folders → `status: done`

**Agent Review Notation**:
- **Initially**: Create documents with only the basic AI warning: `> **⚠️ AI-Generated**: May contain errors. Verify before use.`
- **After specialized agent review**: Add the review line: `> **Reviewed by**: [agent-name] agent`
- **Common reviewing agents**:
  - `feature-analyzer` - for implementation quality, best practices, over-engineering analysis
  - `security-analyst` - for security implications, vulnerabilities, privacy concerns
  - `frontend-style-validator` - for React component styling and cross-platform compliance
  - Other specialized agents as appropriate
- **Only add review notation** after the agent has completed its analysis AND any recommended changes have been implemented

#### For Bug Reports (`.agents/bugs/`):
```markdown
---
type: bug
title: "[Clear Bug Description]"
status: open
priority: medium
ai_generated: true
reviewed_by: null
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# [Clear Bug Description]

> **⚠️ AI-Generated**: May contain errors. Verify before use.
> **Reviewed by**: [agent-name] agent *(add only after agent review and implementation - also update frontmatter)*

## Symptoms
[What goes wrong - observable behavior]

## Root Cause
[Why it happens - technical analysis]

## Solution
[How it was fixed - specific changes made]
- File changes: `src/path/to/file.ts:123`
- Key insight: [what made the difference]

## Prevention
[How to avoid in future - patterns/practices]
```

#### For Tasks (`.agents/tasks/`):
Use complexity-appropriate template:

**Low Complexity:**
```markdown
---
type: task
title: "[Action-Oriented Title]"
status: open
complexity: low
ai_generated: true
reviewed_by: null
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# [Action-Oriented Title]

> **⚠️ AI-Generated**: May contain errors. Verify before use.
> **Reviewed by**: [agent-name] agent *(add only after agent review and implementation - also update frontmatter)*

**Files**:
- `src/path/to/file.ts:123`

## What & Why
[2-3 sentences: current state → desired state → value]

## Implementation
1. **Update component** (`src/path/file.tsx:45`)
   - Specific change description
   - Reference: Follow pattern from `existing-file.tsx:123`

2. **Add type definition** (`src/types/Type.ts:67`)
   - Add interface property
   - Export type

## Verification
✅ **Feature works as expected**
   - Test: [specific action] → [expected result]

✅ **TypeScript compiles**
   - Run: `npx tsc --noEmit`

## Definition of Done
- [ ] All implementation complete
- [ ] TypeScript passes
- [ ] Manual testing successful
- [ ] No console errors
```

**High Complexity:**
```markdown
---
type: task
title: "[Complex Feature Title]"
status: open
complexity: high
ai_generated: true
reviewed_by: null
created: YYYY-MM-DD
updated: YYYY-MM-DD
related_issues: []  # Add if applicable, e.g., ["#14", "#15"]
related_docs: []    # Add if applicable
related_tasks: []   # Add if applicable
---

# [Complex Feature Title]

> **⚠️ AI-Generated**: May contain errors. Verify before use.
> **Reviewed by**: [agent-name] agent *(add only after agent review and implementation - also update frontmatter)*

**Files**: [list of all affected files with line numbers]

## What & Why
[Detailed description with technical value]

## Context
- **Existing pattern**: [reference similar implementation]
- **Constraints**: [technical limitations]
- **Dependencies**: [prerequisites]

## Prerequisites
- [ ] Review .agents documentation: INDEX.md, AGENTS.md, and agents-workflow.md for context
- [ ] Check existing tasks in .agents/tasks/ for similar patterns and solutions
- [ ] Review related documentation in .agents/docs/ for architectural context
- [ ] Feature analyzed by feature-analyzer agent for complexity and best practices
- [ ] Security analysis by security-analyst agent (if task involves auth, crypto, user data, or network operations)
- [ ] [Required setup/dependencies]
- [ ] Branch created from `develop`
- [ ] No conflicting PRs

## Implementation

### Phase 1: Core Logic
- [ ] **[Specific task]** (`file.tsx:123`)
  - Done when: [observable completion signal]
  - Verify: [specific test]
  - Reference: [existing pattern to follow]

### Phase 2: Integration (requires Phase 1)
- [ ] **[Next task]** (`file.tsx:456`)
  - Done when: [completion criteria]
  - Verify: [verification method]

## Verification
✅ **[Critical functionality]**
   - Test: [step-by-step test]
✅ **TypeScript compiles**
✅ **Mobile compatible**
✅ **Edge cases handled**

## Definition of Done
- [ ] All phases complete
- [ ] All verification tests pass
- [ ] No console errors
- [ ] Task updated with learnings
```

#### For Documentation (`.agents/docs/features/`):

**CRITICAL DISTINCTION - Documentation vs Tasks**:
- **Documentation** describes the **current state** of a feature - what it IS and how it WORKS
- **Tasks** track implementation progress - status, phases, checklists, what needs to be DONE

**Documentation files MUST NOT include**:
- ❌ "Feature Status" or "Implementation Status" sections with phases
- ❌ "Verification Checklist" or "Definition of Done" sections
- ❌ Implementation history language ("we added", "we fixed", "was implemented", "changes made")
- ❌ Phase tracking ("Phase 1 complete", "Phase 2 pending")
- ❌ Changelog-style entries or implementation notes
- ❌ Status indicators (Pending, In Progress, Complete)

**Documentation files SHOULD**:
- ✅ Describe features in present tense as they currently exist
- ✅ Explain architecture, data flow, and integration points
- ✅ Include usage examples and code snippets
- ✅ Document technical decisions and their rationale
- ✅ List known limitations and their impact
- ✅ Reference related documentation and components

```markdown
---
type: doc
title: "[Feature Name]"
status: done
ai_generated: true
reviewed_by: null
created: YYYY-MM-DD
updated: YYYY-MM-DD
related_docs: []    # Add if applicable
related_tasks: []   # Add if applicable
---

# [Feature Name]

> **⚠️ AI-Generated**: May contain errors. Verify before use.
> **Reviewed by**: [agent-name] agent *(add only after agent review and implementation - also update frontmatter)*

## Overview
[What the feature does and why it exists - written as current state, not history]

## Architecture
[Technical implementation details - describe how it works NOW]
- **Key components**: List main files/classes with line references
- **Data flow**: How information moves through system
- **Integration points**: How it connects to other features

## Usage Examples
[Code examples showing how to use the feature]

## Technical Decisions
[Rationale for key architectural choices - explain WHY, not WHEN]
- **[Decision 1]**: Rationale and trade-offs
- **[Decision 2]**: Alternative approaches considered

## Known Limitations
[Current constraints and their impact]
- [Limitation 1]: Impact and potential workarounds
- [Limitation 2]: Design trade-offs accepted

## Related Documentation
- [Cross-references to other relevant docs]
- [Links to API references]
- [Related tasks or bugs]
```

#### For Reports (`.agents/reports/`):
```markdown
---
type: report
title: "[Report Title]"
ai_generated: true
reviewed_by: null
created: YYYY-MM-DD
updated: YYYY-MM-DD
related_tasks: []  # Add if applicable
related_docs: []   # Add if applicable
---

# [Report Title]

> **⚠️ AI-Generated**: May contain errors. Verify before use.
> **Reviewed by**: [agent-name] agent *(add only after agent review and implementation - also update frontmatter)*

## Executive Summary
[Brief overview of key findings and recommendations]

## Scope & Methodology
- **Scope**: [What was analyzed/investigated]
- **Methodology**: [How the analysis was conducted]
- **Tools/Approaches**: [Specific methods or tools used]
- **Timeframe**: [When analysis was conducted]

## Findings
### [Finding Category 1]
- **Issue**: [Description of what was found]
- **Impact**: [Severity/importance level]
- **Evidence**: [Supporting details, file locations, examples]

### [Finding Category 2]
- **Issue**: [Description]
- **Impact**: [Level of concern]
- **Evidence**: [Supporting information]

## Recommendations
### High Priority
1. **[Action Item 1]**
   - **Why**: [Justification]
   - **How**: [Implementation approach]
   - **Files**: `src/path/to/file.ts:123`

2. **[Action Item 2]**
   - **Why**: [Reasoning]
   - **How**: [Steps needed]

### Medium/Low Priority
- [Less critical recommendations]

## Action Items
- [ ] **[Specific task]** - Assigned to: [who] - Due: [when]
- [ ] **[Follow-up task]** - Priority: [level]

## Related Documentation
- [Links to related tasks, bugs, documentation]
- [Cross-references to relevant code or features]

## Appendix
[Supporting data, detailed logs, additional context]

---

_Created: YYYY-MM-DD_
_Report Type: [Audit/Research/Analysis/Assessment]_
```

### Step 3: File Placement and Naming
- **General naming**: Use kebab-case: `feature-specific-descriptive-name.md`
- **Reports naming**: Use date format: `report-name_YYYY-MM-DD.md` (e.g., `auth-security-audit_2025-11-19.md`)
- Place in appropriate folder:
  - Active bugs: `.agents/bugs/`
  - Solved bugs: `.agents/bugs/.solved/`
  - Active tasks: `.agents/tasks/`
  - Completed tasks: `.agents/tasks/.done/`
  - Feature docs: `.agents/docs/features/`
  - Reports/audits: `.agents/reports/` (use date format: `report-name_YYYY-MM-DD.md`)
  - Completed reports: `.agents/reports/.done/`
  - Archived items: respective `.archived/` folders

### Step 4: Cross-Reference Management
- Add references to related documentation
- Update existing docs that reference this item
- Maintain bidirectional links where appropriate
- Reference specific file:line locations when relevant

### Step 5: Lifecycle Management

**For Task Updates (using task-update.md pattern):**
1. **Read and verify** - Check all file paths exist, note current status
2. **Update autonomously** - Fix line numbers, update code examples, clarify steps
3. **Flag (don't change)** - Scope changes, conflicts, status changes to Complete
4. **Never change** - Checked checkboxes, Implementation Notes, original "What & Why"
5. **Document changes** - Add to Updates section with timestamp

**For Status Changes:**
- Moving bugs to `.solved/` when fixed → **MUST update `status: done`**
- Moving tasks to `.done/` when complete → **MUST update `status: done`**
- Moving files to `.archived/` folders → **MUST update `status: archived`**
- Archiving outdated documentation
- Updating cross-references when files move
- **ALWAYS update the `updated` date when changing status**

### Step 6: Agent Review Integration
When creating or updating documents that would benefit from specialized review:

1. **Create initial document** with basic AI warning
2. **Identify review needs**:
   - Complex tasks (Medium/High/Critical) → `feature-analyzer` agent
   - Security-related content → `security-analyst` agent
   - Frontend components → `frontend-style-validator` agent
3. **Launch appropriate specialized agent** using Task tool
4. **Implement agent recommendations** in the document
5. **Add review notation** to document header: `> **Reviewed by**: [agent-name] agent`
6. **Update index** to reflect any changes

### Step 7: Index Management
**CRITICAL: Always update the .agents index after any file operation**

After creating, editing, moving, renaming, or deleting any bug report, task, or documentation file:

1. **Run the index update script**:
   ```bash
   # From WSL:
   python3 /mnt/d/GitHub/Quilibrium/quorum-desktop/.claude/skills/docs-manager/update-index.py

   # From native Windows (PowerShell or cmd):
   python .claude/skills/docs-manager/update-index.py
   ```

2. **Verify the INDEX.md was updated**:
   - Check that new files appear in the index
   - Verify that moved files show in correct sections
   - Confirm deleted files are removed from index

**When to run the index update script**:
- ✅ After creating any new .md file in .agents/
- ✅ After moving files between folders (.done, .solved, .archived)
- ✅ After renaming any .md file in .agents/
- ✅ After deleting any .md file in .agents/
- ✅ After editing titles (# headings) in existing files

The script automatically:
- **Runs yarn scan-docs** to sync project-wide documentation scanning
- Scans all .md files in .agents/ directory
- Extracts titles from first # heading
- Organizes by folder structure (docs → bugs → tasks → reports)
- Maintains proper subfolder groupings
- Updates "Last Updated" timestamp
- Handles numeric prefixes for ordering (01-file.md, 02-file.md)

### Step 8: Commit Guidelines
- Create descriptive commit message following project conventions
- Never mention "Claude" or "Anthropic" in commit messages
- Focus on the "why" rather than the "what"
- Use established commit message patterns from project

## Examples

### Bug Report Example
**User says**: "There's an issue with the modal stacking order"

**Skill response**: Creates `.agents/bugs/modal-zindex-stacking-issue.md` with proper template, analyzes the z-index conflict, documents the CSS fix needed, and includes prevention strategies.

### Task Creation Example
**User says**: "I need to implement user authentication"

**Skill response**: Creates `.agents/tasks/user-authentication.md` with High complexity template, breaks down into phases (setup, UI components, backend integration), includes verification steps for security testing.

### Documentation Example
**User says**: "Document the new search feature we just built"

**Skill response**: Creates `.agents/docs/features/search-feature.md` describing the search feature as it currently exists - architecture, integration with MessageDB, performance characteristics, and usage examples. Uses present tense throughout ("The search feature provides...", "Messages are indexed using...") - NOT implementation history ("We added...", "Phase 1 implemented..."). Does NOT include status sections, verification checklists, or phase tracking.

### Report Creation Example
**User says**: "Create a security audit of our authentication system"

**Skill response**: Creates `.agents/reports/auth-security-audit_2025-11-19.md` with structured audit template, analyzes authentication flows, identifies potential vulnerabilities, provides actionable recommendations with priority levels, and includes specific file references and remediation steps.

### Task Update Example
**User says**: "Update the authentication task - some file paths have changed"

**Skill response**: Reads existing task, verifies current file locations, updates outdated paths and line numbers, adds discovered edge cases, documents changes in Updates section. Then runs the index update script to reflect any title changes in INDEX.md.

### Index Update Example
**User says**: "Move the completed auth task to the .done folder"

**Skill response**:
1. Moves `.agents/tasks/user-authentication.md` to `.agents/tasks/.done/user-authentication.md`
2. Runs the comprehensive update script: `python3 update-index.py`
   - Executes `yarn scan-docs` for project-wide documentation sync
   - Updates INDEX.md with new file location
3. Verifies the task now appears under "📋 Completed Tasks" instead of "📋 Tasks" in INDEX.md
4. Reports successful completion with full documentation synchronization

### Agent Review Workflow Examples

#### Example 1: Security-Critical Task
**User says**: "Create a task for implementing end-to-end encryption"

**Initial creation**:
```markdown
# Implement End-to-End Encryption

> **⚠️ AI-Generated**: May contain errors. Verify before use.

**Status**: Pending
**Complexity**: Critical
```

**After security-analyst review and implementation**:
```markdown
# Implement End-to-End Encryption

> **⚠️ AI-Generated**: May contain errors. Verify before use.
> **Reviewed by**: security-analyst agent

**Status**: Pending
**Complexity**: Critical
```

#### Example 2: Complex Feature Analysis
**User says**: "Document the new modal system we built"

**After feature-analyzer review**:
```markdown
# Modal System Architecture

> **⚠️ AI-Generated**: May contain errors. Verify before use.
> **Reviewed by**: feature-analyzer agent

## Overview
[Documentation refined based on agent analysis...]
```

#### Example 3: Frontend Component Validation
**User says**: "Create task for new cross-platform Button component"

**After frontend-style-validator review**:
```markdown
# Cross-Platform Button Component

> **⚠️ AI-Generated**: May contain errors. Verify before use.
> **Reviewed by**: frontend-style-validator agent

**Status**: Pending
**Complexity**: Medium
```

## Workflow Integration

This skill integrates seamlessly with your existing workflow:

1. **Respects established patterns** from `agents-workflow.md`
2. **Uses existing templates** enhanced with automation
3. **Maintains folder structure** and naming conventions
4. **Preserves manual work** - never overwrites completed sections
5. **Cross-references properly** - links to related docs and code
6. **Follows commit guidelines** - proper messages, no AI mentions
7. **Updates systematically** - tracks changes and maintains history
8. **🆕 Comprehensive documentation sync** - runs both yarn scan-docs and update-index.py for complete project-wide synchronization
9. **🆕 Quality assurance through agent reviews** - integrates specialized agents for enhanced quality and best practices

The skill essentially automates what you would do manually following your `agents-workflow.md`, but with intelligent context awareness, consistent application of your established patterns, **comprehensive documentation synchronization** (both project-wide and .agents-specific), and **integrated quality reviews by specialized agents** to ensure the entire documentation ecosystem is well-maintained and synchronized.

## Utility Scripts

### YAML Frontmatter Migration (`add-yaml-frontmatter.cjs`)

Adds YAML frontmatter metadata to markdown files in `.agents/` folder.

**When to use:**
- Migrating new documentation categories
- Adding new frontmatter fields to existing files
- Backfilling metadata

**Usage:**
```bash
# Preview changes
node .claude/skills/docs-manager/add-yaml-frontmatter.cjs --dry-run

# Apply changes
node .claude/skills/docs-manager/add-yaml-frontmatter.cjs --apply
```

**What it extracts:**
- Type (from folder), Title (from H1), Status (from content/folder)
- Dates (from content/filesystem), AI-generated flag, Related issues

**Safety:** Always run `--dry-run` first. Skips files with existing frontmatter.

---

_Updated: 2026-01-09 18:48_