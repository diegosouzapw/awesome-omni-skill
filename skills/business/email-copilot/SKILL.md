---
name: email-copilot
description: Manages inbox by learning preferences, cleaning noise, and prioritizing important work. Use when user asks about emails or inbox management.
---

## Critical Rules

1. **ALWAYS** read `.claude/skills/email-copilot/rules.md` before processing ANY emails
2. **ALWAYS** run `date` to get current time before processing (for expiry checks, age calculations)
3. **NEVER** mix email IDs between accounts - IDs are account-specific
4. **ALWAYS** use label names (e.g., "Finance/Receipts"), never internal IDs (e.g., "Label_6")
5. **CHECK** `account` field in output before operations
6. **LOG** all user-confirmed actions to `.claude/skills/email-copilot/action-log.md` — **IMMEDIATELY in the same tool call batch as execution, NOT after**

## CLI Alias

All commands use this base (define once per session):
```bash
EMAIL="uv run --project .claude/skills/email-copilot python .claude/skills/email-copilot/scripts/email_cli.py"
GMAIL="uv run --project .claude/skills/email-copilot python .claude/skills/email-copilot/gmail_client.py"
```

## Quick Reference

```bash
# Accounts
$GMAIL --check                    # Check setup status
$GMAIL --auth <name>              # Add/authenticate account
$EMAIL accounts                   # List accounts

# List & Read
$EMAIL [-a ACCOUNT] list [-n LIMIT] [-q QUERY]
$EMAIL [-a ACCOUNT] read <msg_id>

# Actions
$EMAIL [-a ACCOUNT] trash '<json_id_list>'
$EMAIL [-a ACCOUNT] untrash '<json_id_list>'
$EMAIL [-a ACCOUNT] move <label> '<json_id_list>' [-r] [-c]

# Labels
$EMAIL [-a ACCOUNT] labels list
$EMAIL [-a ACCOUNT] labels create "Name"
$EMAIL [-a ACCOUNT] labels delete "Name"
$EMAIL [-a ACCOUNT] labels rename "Old" "New"

# Send & Reply
$EMAIL [-a ACCOUNT] send --to EMAIL --subject SUBJ --body BODY [--cc CC] [--attachment FILE]
$EMAIL [-a ACCOUNT] reply <msg_id> --body BODY [--cc CC]

# Drafts
$EMAIL [-a ACCOUNT] drafts list
$EMAIL [-a ACCOUNT] drafts create --to EMAIL --subject SUBJ --body BODY
$EMAIL [-a ACCOUNT] drafts reply <msg_id> --body BODY
$EMAIL [-a ACCOUNT] drafts send <draft_id>
$EMAIL [-a ACCOUNT] drafts delete <draft_id>

# Attachments
$EMAIL [-a ACCOUNT] attachments <msg_id>
$EMAIL [-a ACCOUNT] download <msg_id> [-o DIR] [-f FILTER]
$EMAIL [-a ACCOUNT] search-download -q QUERY [-o DIR] [-n LIMIT]

# Filters & Maintenance
$EMAIL [-a ACCOUNT] filters list
$EMAIL [-a ACCOUNT] filters add [--from X] [--add-label Y] [--archive] [--mark-read]
$EMAIL [-a ACCOUNT] summary <label> [-n LIMIT]
$EMAIL [-a ACCOUNT] cleanup <label> [-d DAYS]
```

## Procedure

1. **Check Accounts**: `$EMAIL accounts`
2. **Load Rules**: Read `.claude/skills/email-copilot/rules.md` (MANDATORY)
3. **Review Action Log**: Read `.claude/skills/email-copilot/action-log.md` to identify patterns from recent decisions
4. **Process Each Account**:
   - `$EMAIL -a <account> list -n 100`
   - Apply account-appropriate rules (work: strict cleanup; personal: conservative)
5. **Classify & Execute**:
   - Auto-Trash List matches → `trash`
   - Expired notifications → `trash`
   - Receipts/Statements → `move` to Finance labels
6. **Log Actions**: When user confirms, execute operations AND update `action-log.md` **in the same tool call batch** (parallel Edit + Bash calls)
7. **Report & Evolve**:
   - Group remaining by account AND category
   - Review action log for recurring patterns → propose rule updates
   - Update `rules.md` if user agrees

## Learning & Evolution

When email doesn't match rules:
- **Cold sales** → Propose adding sender to Auto-Trash List
- **New project** → Propose adding Project Keyword
- **Recurring newsletter** → Propose Newsletter category

Always ask user before updating `rules.md`.

## Multi-Account Notes

- Work accounts: Prioritize project emails, strict cleanup
- Personal accounts: Conservative deletions, preserve receipts
- Each operation outputs `account` field to identify mailbox

## Tips

- **Reply-To**: `reply` uses Reply-To header when present (mailing lists)
- **GitHub**: Check snippet for "approved", "lgtm", bot names
- **Safety**: If unsure, list in report instead of trashing
- **Drafts Workflow**: Create draft → User reviews in Gmail → `drafts send` or manual send

## Action Log Format

Log file: `.claude/skills/email-copilot/action-log.md`

```markdown
## YYYY-MM-DD

### Trashed
- [account] sender@example.com - "Subject" (reason: auto-trash match / user confirmed)

### Moved
- [account] sender@example.com - "Subject" → Label/Name

### Kept
- [account] sender@example.com - "Subject" (user decision: reason)

### Patterns Identified
- sender@example.com appeared 3x this week → consider adding to auto-trash
```

Review log before each session to:
1. Identify senders appearing repeatedly → propose Auto-Trash rule
2. Find emails consistently moved to same label → propose filter
3. Track user overrides → adjust rule confidence

## Additional Files

- **First-time setup**: Read `.claude/skills/email-copilot/README.md`
- **Invoice recipes**: Read `.claude/skills/email-copilot/SKILL-RECIPES.md`
- **User rules**: `.claude/skills/email-copilot/rules.md`
- **Action log**: `.claude/skills/email-copilot/action-log.md`
