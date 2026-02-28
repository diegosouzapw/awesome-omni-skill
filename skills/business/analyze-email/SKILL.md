---
name: analyze-email
description: Quick single email analysis - read metadata, content summary, and list attachments without extracting them.
---

# Analyze Email

Quick analysis of a single email. Read metadata, summarize content, and list attachments without extracting them.

## When to Use

Use this skill when you need to:
- Quickly understand what an email is about
- Get email metadata (sender, subject, date)
- See what attachments are included
- Decide if deeper analysis is needed

**Triggers**:

English:
- "Analyze this email `<message-id>`"
- "What's in this email `<message-id>`"
- "Read this email `<message-id>`"
- "Show me this email `<message-id>`"

Chinese:
- "分析这封邮件 `<message-id>`"
- "这封邮件说什么 `<message-id>`"
- "看看这封邮件 `<message-id>`"
- "查看邮件内容 `<message-id>`"

## Invocation Methods

### Method 1: Natural Language (Recommended)

```bash
User: Analyze this email <E42FF77C-C563-466D-8544-3E0C16EA24EC@taler.net>
AI: [Proceeds to analyze immediately]
```

### Method 2: Slash Command

```bash
User: /analyze-email
AI: 请提供要分析的邮件 Message-ID
User: <E42FF77C-C563-466D-8544-3E0C16EA24EC@taler.net>
AI: [Proceeds to analyze]
```

> **Note**: When invoked via slash command without parameters, ALWAYS ask for message-id first.

## Execution Steps

1. **Get message-id** (if not provided)
   ```bash
   Please provide the email Message-ID to analyze

   💡 Quick method (recommended):
   In Mail.app: Select email → Press shortcut (⌘⇧C) → Message-ID copied
   See README "Setup Mail Quick Action" section if not configured
   ```

2. **Read email using MCP**
   ```python
   email_data = mcp__mail__read_email(message_id)
   ```

3. **Extract key information**
   - Sender (from)
   - Recipients (to, cc)
   - Subject
   - Date
   - Body text summary (first 500 chars)
   - Attachments list (filename, type, size)

4. **Present structured output**

## Output Format

```markdown
📧 Email Analysis

### Basic Information
- From: [sender email and name]
- To: [recipients]
- CC: [cc recipients if any]
- Subject: [subject line]
- Date: [sent date]

### Content Summary
[First 500 characters of email body, or full text if short]

### 📎 Attachments
[Count] attachment(s):
1. [filename] ([type]) - [size]
2. ...

### 💡 Quick Assessment
- Is this a thread?: Yes/No
- Action required?: Yes/No
- Importance level: High/Medium/Low
```

## Important Notes

1. **Quick Mode Only**
   - Do NOT extract attachment content
   - Do NOT read email threads (use /analyze-thread instead)
   - Just provide overview

2. **Thread Detection**
   - Check if email is part of a thread
   - If yes, suggest user to use /analyze-thread for full context

3. **Next Steps**
   - If user wants attachment details, suggest /analyze-attachments
   - If user wants thread context, suggest /analyze-thread
   - If user asks for attachment extraction, suggest /analyze-attachments

## Example Scenarios

### Scenario 1: Quick Email Check

User: /analyze-email
AI: 请提供要分析的邮件 Message-ID
User: <123@example.com>
AI:
📧 Email Analysis

### Basic Information
- From: John Smith <john@example.com>
- To: you@company.com
- Subject: Q4 Report Ready
- Date: Dec 15, 2025 at 10:30 AM

### Content Summary
Hi,

The Q4 financial report is ready for review. Key highlights:
- Revenue increased by 15% compared to Q3
- New customer acquisition: 250 companies
- Product launch scheduled for January

Please review the attached PDF and let me know if you have any questions.

Best regards,
John

### 📎 Attachments
1 attachment:
1. Q4_Report_2025.pdf (application/pdf) - 2.3 MB

### 💡 Quick Assessment
- Is this a thread?: No
- Action required?: Yes - Review report
- Importance level: High

### Scenario 2: Thread Detected

User: What's in this email <thread-start@example.com>
AI:
📧 Email Analysis

[Email details...]

### 💡 Quick Assessment
- Is this a thread?: **Yes** (15 emails in thread)
- This email appears to be part of a longer conversation.
- Would you like me to analyze the full thread using /analyze-thread?

## Related Skills

- `/analyze-thread` - For complete email thread analysis
- `/analyze-attachments` - For deep attachment analysis and extraction
