---
name: task-add
description: This skill should be used when the user asks to "add a task", "create task", "new task", or "task to do X". Adds timestamped tasks to daily note log with optional due dates.
auto_trigger: true
trigger_keywords:
  - "add a task"
  - "add task"
  - "create task"
  - "create a task"
  - "task to do"
  - "new task"
  - "task for"
  - "task due"
  - "task by"
allowed-tools: Read, Edit
---

# Task Add

Automatically add a timestamped task entry to today's daily note log when user wants to create a task. Due date is optional.

## Trigger Detection

This skill activates when the user's prompt contains task creation intent:
- "add a task to review PR"
- "add a task to do X by DATE"
- "create task to X due DATE"
- "new task: X"
- "task to do X for DATE"

Due date is optional. If no due date is mentioned, omit the 📅 portion from the task.

## Examples

User prompts that trigger this skill:
- "add a task to review PR" (no due date - task added without 📅)
- "add a task to review PR by tomorrow" (with due date)
- "create a task to call dentist by 2025-12-31"
- "task to do laundry by Friday"
- "new task: finish report due Jan 5"

User prompts that DO NOT trigger this skill:
- "create a task" (no description)
- "what tasks do I have" (query, not creation)

## Process

1. **Check for due date (optional)**
   - Check if prompt contains date reference (by, due, for + date)
   - If no date found, proceed without due date

2. **Get current time**
   - Parse from injected UserPromptSubmit hook context: "Time: HH:MM"
   - **IMPORTANT**: Do NOT use Bash or PowerShell commands.

3. **Extract task description**
   - Remove trigger words ("add a task to", "task to do", etc.)
   - Extract description before the due date reference
   - Preserve any [[wikilinks]] in the description

4. **Extract and parse due date**
   - Look for date reference after "by", "due", "for"
   - Parse natural language dates:
     - "tomorrow" → next day (YYYY-MM-DD)
     - "Friday" → next Friday (YYYY-MM-DD)
     - "2025-12-31" → ISO format (YYYY-MM-DD)
     - "Jan 5" / "January 5" → YYYY-01-05
   - Format: YYYY-MM-DD (ISO 8601)

5. **Construct daily note path**
   - Path: `{paths.daily_notes}/YYYY-MM-DD.md` (from config)

6. **Read the daily note**
   - Find the Log section (from config `sections.log`)

7. **Add the task entry**
   - Format: `- [ ] HH:MM [description] 📅 YYYY-MM-DD` (omit 📅 portion if no due date)
   - Use the calendar emoji: 📅 (U+1F4C5) when due date is present
   - Add after the last log entry in the section
   - Use Edit tool to insert the new entry

8. **Confirm**
   - Display the added task with timestamp, description, and due date

## Task Entry Format

```markdown
## Log

- [ ] 20:50 Review PR #123 📅 2025-12-31
- [ ] 14:22 Call dentist 📅 2026-01-05
- [ ] 09:15 Finish quarterly report
```

### Format Components

1. **Checkbox**: `- [ ]` (unchecked task)
2. **Timestamp**: `HH:MM` (when task was created)
3. **Description**: Free text with optional [[wikilinks]]
4. **Due date emoji**: `📅` (calendar emoji, U+1F4C5) - optional
5. **Due date**: `YYYY-MM-DD` (ISO 8601 format) - optional

## Date Parsing Rules

### Supported Date Formats

**Relative dates** (calculate from injected context date):
- "tomorrow" → +1 day
- "today" → same day
- "Monday", "Tuesday", etc. → next occurrence of day
- "next Monday" → following Monday
- "in 3 days" → +3 days

**Absolute dates**:
- "2025-12-31" → use as-is (ISO format)
- "12/31/2025" → convert to 2025-12-31
- "Dec 31" → current year + 12-31
- "January 5" → current year + 01-05

### Edge Cases

- If date is in the past, use it anyway (user may want to track overdue tasks)
- If date is ambiguous ("Friday" on a Friday), assume next Friday
- If date cannot be parsed, ask user: "Please specify the due date as YYYY-MM-DD"

## Critical Rules

- **Due date optional**: Include 📅 portion only if due date is mentioned
- **Calendar emoji**: Use 📅 (U+1F4C5) when due date present
- **ISO date format**: Always YYYY-MM-DD for due dates
- **Timestamp required**: Always include HH:MM format for task creation time
- **Preserve structure**: Don't modify other sections
- **Task format**: Use `- [ ] HH:MM description [📅 YYYY-MM-DD]`
- **Wikilinks**: Preserve any [[wikilinks]] in the user's description

## Six Gates Compliance

| Gate | Implementation |
|------|----------------|
| 1. Read Before Write | Reads daily note before adding task (step 6) |
| 2. File Exists | Validates daily note exists; creates if missing |
| 3. Chain Validation | N/A (single operation) |
| 4. Mutation Confirmation | Task shown to user before write |
| 5. Health Check | Uses vault config infrastructure |
| 6. Post Validation | Confirms task added with timestamp and due date |

## Configuration

This skill uses these config values:
- `paths.daily_notes`: Folder containing daily notes (default: "daily-notes")
- `sections.log`: Log section header (default: "## Log")
