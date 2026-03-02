---
name: apple-productivity
description: Access macOS productivity apps (Calendar, Contacts, Mail, Messages, Reminders, Voice Memos). Use when user asks about calendar events, contacts, emails, iMessages, reminders, or voice transcription.
allowed-tools:
  - Bash
  - Read
  - Write
  - AskUserQuestion
---

# Apple Productivity Apps

Access macOS native productivity apps via helper scripts in ~/bin/.

## Quick Start

| You want to... | Run... |
|----------------|--------|
| Today's calendar | `calendar-events` |
| Find a contact | `contacts-search "name"` |
| Recent messages | `imessage-recent` |
| Unread mail | `mail-unread` |
| Overdue reminders | `reminders-list -o` |
| Transcribe voice memo | `voice-memos process` |

All scripts support `-j` for JSON output.

## Detailed References

- **Calendar**: [reference/calendar.md](reference/calendar.md)
- **Contacts**: [reference/contacts.md](reference/contacts.md)
- **Mail**: [reference/mail.md](reference/mail.md)
- **Messages**: [reference/messages.md](reference/messages.md)
- **Reminders**: [reference/reminders.md](reference/reminders.md)
- **Voice Memos**: [reference/voice-memos.md](reference/voice-memos.md)

## Security

**Read-only (no confirmation needed)**:
- Viewing events, contacts, mail, messages, reminders

**Write operations (ALWAYS confirm first)**:
- `calendar-add` - creating events
- Sending messages/emails
- Modifying reminders

**Rule**: Use `AskUserQuestion` before ANY write operation.

## Common Patterns

### Morning context check
```bash
calendar-events && reminders-list -o && imessage-recent -n
```

### Contact lookup with birthday
```bash
contacts-search -v "Marie"
```

### Historical message search
```bash
imessage-search --pattern "resume"
```

## Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| Mail.app not running | `mail-unread` requires app | Tell user to open Mail.app |
| Query timeout | Date range too large | Use smaller range |
| No results | Empty dataset | Handle gracefully |

## Related Skills

- `apple-photos` - Photo library access
- `apple-shortcuts` - Run macOS Shortcuts
- `apple-health-fitness` - Health data (under development)
