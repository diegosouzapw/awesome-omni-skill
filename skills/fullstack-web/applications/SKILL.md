---
name: applications
description: Manage and query job applications. Use when the user wants to add, view, update, or analyze their job applications.
---

# Applications Skill

You help users manage their job application tracker. This skill provides conversational access to the application tracker stored in `applications.jsonl`.

## Storage Location

Applications are stored in `applications.jsonl` in the project root. Each line is a JSON object representing one application.

## Data Model

```json
{
  "id": "app-xyz",
  "company": "Company Name",
  "role": "Job Title",
  "status": "applied|interviewing|rejected|offer",
  "date_applied": "2025-01-15",
  "jd_url": "https://...",
  "jd_content": "Full job description...",
  "resume_path": "outputs/company_role/resume.html",
  "company_url": "https://company.com",
  "notes": "Free-form notes",
  "created_at": "...",
  "updated_at": "..."
}
```

## Capabilities

### 1. List Applications

Read `applications.jsonl` and present applications in a clear format.

**User requests:**
- "Show me my applications"
- "List all applications"
- "What applications do I have?"

**Response format:**
```
| ID | Company | Role | Status | Date Applied |
|----|---------|------|--------|--------------|
| app-xxx | Company A | Engineer | applied | 2025-01-15 |
```

### 2. Show Application Details

Show full details of a specific application.

**User requests:**
- "Show me app-xxx"
- "Details for the Google application"
- "What's the JD for app-xxx?"

### 3. Add Application

Create a new application entry. Collect required information interactively.

**Required fields:**
- Company name
- Role/position

**Optional fields:**
- JD URL
- JD content (full text)
- Company URL
- Resume path
- Notes

**User requests:**
- "Add a new application"
- "I applied to Google for a Senior Engineer role"
- "Track my application to [Company]"

**Process:**
1. If company/role provided, use them
2. Ask for any missing required info
3. Ask for optional info or offer to skip
4. Generate ID (app-XXXXXXXX)
5. Set status to "applied"
6. Set date_applied to today
7. Append to applications.jsonl

### 4. Update Application

Update status or other fields of an existing application.

**Status values:**
- `applied` - Initial state
- `interviewing` - In interview process
- `rejected` - Application rejected
- `offer` - Received offer

**User requests:**
- "Update app-xxx to interviewing"
- "I got rejected from Google"
- "Mark the Amazon application as offer"
- "Add notes to app-xxx"

### 5. Remove Application

Remove an application from tracking.

**User requests:**
- "Remove app-xxx"
- "Delete the Google application"

**Always confirm before removing.**

### 6. Natural Language Queries

Answer questions about applications.

**User requests:**
- "How many applications do I have?"
- "Show pending applications" (status = applied)
- "Which applications are in interview stage?"
- "Applications from last week"
- "Did I apply to Google?"

## CLI Commands

Use the `bragger` CLI for quick operations:

```bash
# Interactive mode
bragger add

# Flag mode (quick add)
bragger add --company "Company" --role "Job Title"
bragger add --company "Company" --role "Title" --jd-url "https://..." --notes "Referral"

# Other commands
bragger list             # List all
bragger show <id>        # Show details
bragger update <id>      # Interactive update
bragger update <id> --status "interviewing"  # Flag mode (quick)
bragger remove <id>      # Remove with confirmation
```

### Available flags for `add` and `update`:

| Flag | Required | Description |
|------|----------|-------------|
| `--company` | Yes (with flags) | Company name |
| `--role` | Yes (with flags) | Job title |
| `--status` | No | Status: applied, interviewing, rejected, offer (default: applied) |
| `--date` | No | Date applied in YYYY-MM-DD format (default: today) |
| `--jd-url` | No | Job description URL |
| `--jd-content` | No | Job description text (inline) |
| `--jd-file` | No | Path to file containing job description |
| `--company-url` | No | Company website |
| `--resume-path` | No | Path to resume file |
| `--notes` | No | Notes about application |

**For `add`:** `--company` and `--role` are required when using flags. Without any flags, runs interactively.

**For `update`:** All flags are optional. Only provided flags will be updated. Without flags, runs interactively.

**Note:** `--jd-content` and `--jd-file` cannot be used together.

### Updating Applications

Update any field using flags:

```bash
# Update status
bragger update app-xxx --status "interviewing"

# Update multiple fields
bragger update app-xxx --status "offer" --notes "Accepted the offer!"

# Add JD content later
bragger update app-xxx --jd-file ./jd.txt
```

### Backfilling Old Applications

Use `--status` and `--date` to add applications retroactively:

```bash
bragger add --company "OldCorp" --role "Engineer" --date "2025-01-15" --status "interviewing"
```

### Adding JD Content

There are three ways to add job description content:

1. **From file** (recommended for long JDs):
   ```bash
   bragger add --company "Acme" --role "Engineer" --jd-file ./jd.txt
   ```

2. **Inline** (for short descriptions):
   ```bash
   bragger add --company "Acme" --role "Engineer" --jd-content "Looking for..."
   ```

3. **Via Claude skill** (for fetching from URL):
   If you only have a URL and want clean extracted content, use the `/applications` skill:
   > "Fetch and store the JD content for the [company] application"
   
   This leverages Claude's ability to intelligently extract relevant content from job posting pages, filtering out navigation, ads, and other noise.

## Implementation Notes

### Reading applications.jsonl

Use the Read tool to read the file, then parse each line as JSON.

### Writing applications.jsonl

When adding or updating, read all entries, modify as needed, then write back the entire file using the Write tool.

### ID Generation

Generate IDs in format: `app-` + 8 random hex characters (e.g., `app-a1b2c3d4`)

## Example Interactions

**User:** "I just applied to Stripe for a Senior Backend Engineer role"

**Response:**
1. Create new application with company="Stripe", role="Senior Backend Engineer"
2. Set status="applied", date_applied=today
3. Ask: "Would you like to add the job description or any notes?"
4. Save to applications.jsonl
5. Confirm: "Added application app-xxx for Senior Backend Engineer at Stripe"

---

**User:** "Show me my pending applications"

**Response:**
1. Read applications.jsonl
2. Filter where status="applied"
3. Display in table format
4. Include count: "You have X pending applications"

---

**User:** "I got an interview with Stripe!"

**Response:**
1. Find Stripe application
2. Update status to "interviewing"
3. Confirm: "Updated Stripe application to interviewing status"
