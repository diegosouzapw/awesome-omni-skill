---
name: process-needs-action
description: This skill should be used when you need to process pending action items from a designated folder and generate detailed implementation plans based on organizational rules. Ideal for task management workflows that require systematic processing of pending items, plan creation, and dashboard updates. Use when the user asks to process action items, review pending tasks, or generate plans from a backlog.
---

# Process Needs Action

Process pending action items from a designated folder and create detailed implementation plans based on organizational rules.

## Required Clarifications

Before processing, gather these essential details:

1. **What is your vault/workspace directory path?** (e.g., `AI_Employee_Vault/`, `workspace/`, etc.)
2. **What should I filter by?** (Process all pending items, or filter by priority: high/medium/low?)

## Optional Clarifications

3. **Should I preview plans before creating them?** (Show plan content for approval before saving)
4. **Should I auto-update file statuses?** (Change from `pending` to `in_progress` automatically, or ask first)

Note: Avoid asking all questions at once. Ask required questions first, then optional ones if needed.

## Before Implementation

Gather context to ensure successful processing:

| Source | Gather |
|--------|--------|
| **User Input** | Vault directory path, processing preferences, priority filter |
| **Codebase** | Verify paths exist, validate directory structure |
| **Conversation** | Any specific processing rules or preferences discussed |
| **Handbook/Rules** | Read organizational processing rules if available |

## Instructions

### 1. Validate Structure

Check that required paths exist:
- Verify `<vault_dir>/Needs_Action/` directory exists
- Check for `<vault_dir>/Company_Handbook.md` (optional but recommended)
- Ensure `<vault_dir>/Plans/` directory exists (create if missing)
- Verify `<vault_dir>/Dashboard.md` exists (create if missing)

### 2. Read Processing Rules

- If `<vault_dir>/Company_Handbook.md` exists, read it to understand processing rules
- Note priority keywords (e.g., URGENT, CLIENT, INVOICE, DEADLINE)
- Note file handling guidelines
- If handbook doesn't exist, use default processing rules

### 3. Scan Needs_Action Folder

- List all markdown files in `<vault_dir>/Needs_Action/`
- Filter for files with `status: pending` in frontmatter
- Apply priority filter if specified by user
- Sort by priority (high → medium → low)

### 4. Process Each Action Item

For each pending file:

**a. Read and Parse**
- Read the full file content
- Extract frontmatter metadata (type, priority, source_file, detected)
- Identify the source file type and context

**b. Analyze Content**
- Based on file type (PDF, TXT, image, etc.), determine appropriate action
- Check for priority keywords (URGENT, CLIENT, INVOICE, DEADLINE)
- Cross-reference with Company Handbook rules
- Consider Business_Goals.md priorities

**c. Create Detailed Plan**
- Create a new file in `<vault_dir>/Plans/`
- Filename: `PLAN_[YYYYMMDD_HHMMSS]_[source_filename].md`
- Use plan template from `references/plan-template.md`
- Include:
  - Clear objective
  - Step-by-step actions with checkboxes
  - Required information or approvals
  - Expected outcome
  - Links to related files
- If preview mode enabled, show plan to user for approval before saving

**d. Update Action File**
- Change status from `pending` to `in_progress` (or ask user if auto-update disabled)
- Add processing timestamp
- Add link to created plan

### 5. Update Dashboard
- Update `<vault_dir>/Dashboard.md`
- List all pending actions with priorities
- Show count of items processed
- Update "Last Check" timestamp
- If Dashboard.md doesn't exist, create it with basic structure

### 6. Log Activity (Optional)
- If `<vault_dir>/Logs/` directory exists, append to today's log file
- Filename: `<vault_dir>/Logs/YYYY-MM-DD.json`
- Record: files processed, plans created, any errors

## Error Handling

Handle these scenarios gracefully:

| Scenario | Action |
|----------|--------|
| **No pending files found** | Inform user "No pending action items found" and exit gracefully |
| **Vault directory doesn't exist** | Ask user to confirm path or provide correct vault directory |
| **Company Handbook missing** | Proceed with default processing rules, inform user |
| **Malformed frontmatter** | Log error, skip file, continue with next file |
| **File permission errors** | Log error with filename, skip file, continue processing |
| **Plans directory doesn't exist** | Create `<vault_dir>/Plans/` directory automatically |
| **Dashboard.md missing** | Create basic dashboard with template structure |

## What NOT to Do

Avoid these anti-patterns:

- **Don't modify source files directly** - Only update frontmatter status, never change content
- **Don't skip updating the dashboard** - Always update dashboard to reflect current state
- **Don't create plans without context** - Read source file completely before creating plan
- **Don't process without checking status** - Only process files with `status: pending`
- **Don't hardcode paths** - Use user-provided vault directory path
- **Don't fail silently** - Log all errors and inform user of issues

## Plan Template

See `references/plan-template.md` for the complete plan template structure.

## Success Criteria

- [ ] All pending action files matching filter criteria are reviewed
- [ ] Detailed plan created for each action item
- [ ] Action files updated to `in_progress` (if auto-update enabled)
- [ ] Dashboard updated with current status and counts
- [ ] All errors logged and reported to user
- [ ] User informed of completion with summary

## Example Usage

**User request:** "Process all high-priority action items"

**Workflow:**
1. Ask required clarifications (vault path, priority filter)
2. Validate directory structure at `<vault_dir>/`
3. Read `<vault_dir>/Company_Handbook.md` if available
4. Scan `<vault_dir>/Needs_Action/` for files with `status: pending` and `priority: high`
5. For each high-priority file:
   - Read and analyze content
   - Create detailed plan in `<vault_dir>/Plans/`
   - Update action file status
6. Update `<vault_dir>/Dashboard.md` with processing results
7. Report summary: "Processed 3 high-priority items, created 3 plans"
