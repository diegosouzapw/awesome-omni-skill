---
name: GroundEffect
description: Use this skill when the user asks about email, calendar, or Gmail/Google Calendar management via GroundEffect CLI. Triggers include "search my email", "list recent emails", "check my calendar", "what's on my calendar", "show my meetings", "calendar tomorrow", "calendar next week", "create a calendar event", "manage groundeffect accounts", "sync status", "start the daemon", "groundeffect", "groundeffect command", "draft email", "save draft", "save as draft", "create draft", "send email".
version: 1.0.0
---

# GroundEffect CLI

GroundEffect provides local-first email and calendar management with semantic search capabilities. All data is synced locally and searchable offline.

## Quick Reference

### Email Commands
```bash
groundeffect email search "query"              # Search emails with natural language
groundeffect email list                        # List recent emails
groundeffect email show <id>                   # Show single email
groundeffect email thread <thread_id>          # Show email thread
groundeffect email send --to X --subject "X" --body "X"  # Send email
groundeffect email send --to X --subject "X" --body "X" --html  # Send HTML email
groundeffect email send --to X --subject "X" --body "X" --save-as-draft  # Save as draft
groundeffect email attachment <email_id> <filename>      # Get attachment
groundeffect email folders                     # List IMAP folders
```

### Draft Commands
```bash
groundeffect email draft create --from X --to X --subject "X" --body "X"  # Create draft
groundeffect email draft create --from X --to X --subject "X" --body "X" --reply-to <email_id>  # Reply draft
groundeffect email draft list --from X                   # List drafts
groundeffect email draft show --from X --draft-id <id>   # Show draft
groundeffect email draft update --from X --draft-id <id> --body "X"  # Update draft
groundeffect email draft send --from X --draft-id <id>   # Send draft
groundeffect email draft delete --from X --draft-id <id> # Delete draft
```

### Calendar Commands
```bash
groundeffect calendar events                   # List events by date range (no query needed)
groundeffect calendar events --from 2026-01-07 --to 2026-01-08 --human  # Tomorrow's events
groundeffect calendar search "query"           # Search events with semantic search
groundeffect calendar list                     # List calendars
groundeffect calendar show <event_id>          # Show event details
groundeffect calendar create --summary "X" --start "ISO" --end "ISO"  # Create event
```

**Calendar Events vs Calendar Search**:
- Use `calendar events` when the user asks "what's on my calendar tomorrow/next week" (date-based, no query)
- Use `calendar search` when the user asks "find meetings about project X" (semantic search)

### Account Commands
```bash
groundeffect account list                      # List all accounts
groundeffect account show <email|alias>        # Show account details
groundeffect account add                       # Add new Google account
groundeffect account delete <email|alias>      # Remove account
groundeffect account configure <email|alias>   # Update settings
```

### Sync Commands
```bash
groundeffect sync status                       # Check sync status
groundeffect sync reset <email|alias>          # Reset synced data
groundeffect sync extend <email|alias>         # Sync older emails
groundeffect sync download-attachments <email|alias>  # Download pending attachments
```

### Daemon Commands
```bash
groundeffect daemon install                    # Install launchd daemon (auto-start at login)
groundeffect daemon uninstall                  # Remove launchd daemon
groundeffect daemon status                     # Check if daemon running
groundeffect daemon restart                    # Restart daemon
```

### Config Commands
```bash
groundeffect config settings                   # View/modify daemon settings
groundeffect config add-permissions            # Add Claude Code permissions
groundeffect config remove-permissions         # Remove Claude Code permissions
```

## Important Guidelines

**DRAFTS**: When the user asks to "save as draft", "draft an email", or "save this draft", ALWAYS use the `groundeffect email draft create` command to save it to Gmail drafts. Do NOT save email drafts to markdown files - use Gmail's native draft system so the user can access, edit, and send drafts from any device.

**SENDING**: When composing an email, use `groundeffect email send --save-as-draft` to save to Gmail drafts, or add `--confirm` to send immediately. Without these flags, the command returns a preview.

## Usage Notes

- **CLI binary**: `groundeffect`
- **Output format**: JSON by default, use `--human` for readable output
- **Date format**: Use YYYY-MM-DD for date parameters
- **Account references**: Use email address or alias interchangeably
- **Help**: Add `--help` to any command for detailed options

## Detailed Documentation

For complete command documentation with all flags and examples, read the appropriate reference file:

- **Email**: `references/email-commands.md` - search, list, show, thread, send (HTML support), attachment, folders, drafts
- **Calendar**: `references/calendar-commands.md` - search, list, show, create
- **Accounts**: `references/account-commands.md` - list, show, add, delete, configure
- **Sync**: `references/sync-commands.md` - status, reset, extend, download-attachments
- **Daemon**: `references/daemon-commands.md` - install, uninstall, status, restart
- **Config**: `references/config-commands.md` - settings, add-permissions, remove-permissions

## Common Workflows

See `examples/common-workflows.md` for step-by-step usage patterns.
