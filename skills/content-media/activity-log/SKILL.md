---
name: activity-log
description: Display the daily activity log showing all content analyzed today. Use when the user asks what they've watched, read, or analyzed today, mentions activity log, daily log, or wants to see their consumption history.
---

# Activity Log Display

Display the activity log for today or a specific date.

## When to Use

Activate this skill when the user:
- Asks "what did I watch/read/analyze today?"
- Mentions "activity log", "daily log", "consumption log"
- Wants to see their learning history
- Asks about their progress or what they've consumed

## Instructions

1. **Determine the date**:
   - Default: today's date in YYYY-MM-DD format
   - If user specifies a date, use that date instead
2. **Check if log file exists** at `logs/YYYY-MM-DD.md`
3. If file doesn't exist:
   - Say: "No activity logged for [date] yet."
   - Optionally mention: "Use /yt, /read, /arxiv, or /analyze to process content."
   - Stop here
4. If file exists:
   - **Read the log file**
   - **Display the contents** to the user in a readable format

## Log File Format

The log file typically looks like:
```markdown
# Activity Log: YYYY-MM-DD

## Videos Watched
- [Video Title](../reports/youtube/filename.md) - 14:32
- [Another Video](../reports/youtube/filename2.md) - 16:45

## Articles Read
- [Article Title](../reports/articles/filename.md) - 10:15

## Papers Reviewed
- [Paper Title](../reports/papers/filename.md) - 11:30
```

## Optional Arguments

If the user provides a date (e.g., "show me yesterday's log" or "what did I read on December 20th"), show that day's log instead of today's.

## Error Handling

- If specified date's log doesn't exist: "No activity logged for [date]."
- If logs folder doesn't exist: "No activity logs found. Start analyzing content to create logs."

## Related

- Slash command equivalent: `/log`
- Log location: `logs/YYYY-MM-DD.md`
- Created automatically when analyzing content
