---
name: analyze-thread
description: Analyze complete email thread with conversation context, timeline, participants, and attachments tracking.
---

# Analyze Email Thread

Comprehensive analysis of an email thread including conversation flow, participant interactions, timeline, key points, and attachment evolution.

## When to Use

Use this skill when you need to:
- Understand complete conversation context
- Track discussion progression over time
- Identify key decisions and action items
- See how attachments evolved through the thread
- Get overview of multi-email conversations

**Triggers**:

English:
- "Analyze this email thread `<message-id>`"
- "Show me the conversation `<message-id>`"
- "What's the thread about `<message-id>`"
- "Thread analysis for `<message-id>`"
- "Summarize this thread `<message-id>`"

Chinese:
- "分析这个邮件线索 `<message-id>`"
- "看看这个对话 `<message-id>`"
- "整理一下这个线索 `<message-id>`"
- "线索内容总结 `<message-id>`"
- "这个讨论的脉络 `<message-id>`"

## Invocation Methods

### Method 1: Natural Language (Recommended)

```bash
User: Analyze this email thread <E42FF77C-C563-466D-8544-3E0C16EA24EC@taler.net>
AI: [Proceeds to analyze immediately]
```

### Method 2: Slash Command

```bash
User: /analyze-thread
AI: 请提供线索中任意一封邮件的 Message-ID
User: <E42FF77C-C563-466D-8544-3E0C16EA24EC@taler.net>
AI: [Proceeds to analyze]
```

> **Note**: Any message-id from the thread works. The system will find all related emails.

## Execution Steps

1. **Get message-id** (if not provided)
   ```bash
   Please provide any Message-ID from the thread

   💡 Quick method (recommended):
   In Mail.app: Select email → Press shortcut (⌘⇧C) → Message-ID copied
   See README "Setup Mail Quick Action" section if not configured
   ```

2. **Read thread using MCP**
   ```python
   thread_data = mcp__mail__read_thread(message_id)
   ```

3. **Handle truncation checks**

   **Check 1: Thread Result Oversized**
   - If result indicates size overflow (>80KB):
     - Warn user about thread size
     - Suggest reducing max_body_length
     - Ask if user wants to retry with tighter limits

   **Check 2: Individual Email Truncation**
   - After successful load, check for truncated emails
   - Classify by importance:
     - ⚠️ Critical: Truncated + has attachments
     - ⚠️ High: Lost 60%+ content
     - ℹ️ Medium: Only quote-stripped
   - Suggest reading important emails individually if needed

4. **Analyze thread structure**
   - Count total emails
   - Identify all participants
   - Extract date range
   - Track subject evolution

5. **Extract key information**
   - Main discussion topics
   - Decisions made
   - Action items with owners
   - Attachments and their evolution
   - Important dates/deadlines

6. **Present structured output**

## Output Format

```markdown
🧵 Email Thread Analysis

### Thread Overview
- Total emails: [N]
- Time span: [start date] to [end date]
- Duration: [X days/weeks/months]
- Participants: [list of unique senders]

### Conversation Timeline
[Chronological summary of key emails]

1. [Date] - [Sender] - [Brief description]
2. [Date] - [Sender] - [Brief description]
...

### 💬 Main Topics
1. [Topic 1]
   - [Brief summary]
2. [Topic 2]
   - [Brief summary]

### ✅ Decisions Made
- [Decision 1]
- [Decision 2]

### ⚠️ Action Items
- [ ] [Action item] - [Owner] - [Deadline if any]

### 📎 Attachments Summary
[Total] attachments across thread:
- [filename] - [context/importance]
- [filename] - [context/importance]

### 📅 Important Dates
- [Date]: [Event/Deadline]

### ⚠️ Truncation Notice
[If applicable, warn about truncated emails]
```

## Thread Content Truncation Handling ⚠️

**CRITICAL**: Thread reading has smart limits to prevent token overflow (1200 chars/email with quote stripping).

### Check 1: Thread Result Oversized

If thread is too large:

```markdown
⚠️ **Thread Too Large** (estimated XXX KB)

This email thread is too large to load with current settings.

**Options:**
1. 🔄 **[Recommended]** Reduce limit and retry
   - Current: 1200 chars/email
   - Suggested: 800 chars/email

2. 📧 **Read individual emails**
   - Use /analyze-email for specific messages

**Shall I automatically reduce the limit and retry?**
```

**Implementation**:
- NEVER try to process oversized results
- ALWAYS ask user first before retrying

### Check 2: Individual Email Truncation

After loading, check for truncated emails:

```python
truncated_emails = []
for email in thread['emails']:
    if email.get('truncated'):
        importance = assess_truncation_importance(email)
        if importance in ['Critical', 'High']:
            truncated_emails.append({
                'message_id': email['message_id'],
                'subject': email['subject'],
                'original_length': email.get('original_length'),
                'importance': importance
            })
```

**User Prompt** (only if critical/high truncations found):

```markdown
⚠️ **Detected Truncated Emails**

[Number] emails were truncated. Important ones:

1. [Subject] ([Importance])
   - Original: [X] chars → Current: 1200 chars
   - Has attachments: Yes/No

**Recommendation**: Read important emails individually using /analyze-email
```

## Example Scenarios

### Scenario 1: Technical Discussion Thread

User: /analyze-thread
AI: 请提供线索中任意一封邮件的 Message-ID
User: <E42FF77C-C563-466D-8544-3E0C16EA24EC@taler.net>
AI:
🧵 Email Thread Analysis

### Thread Overview
- Total emails: 37
- Time span: Jul 2, 2025 to Dec 10, 2025
- Duration: ~5 months
- Participants:
  - Marc Stibane (marc@taler.net) - GNU Taler developer
  - Yang Xu (fatbobman@gmail.com) - SwiftUI consultant

### Conversation Timeline

**Phase 1: Problem Identification (Jul 2-3)**
- Marc asks about SwiftUI Text layout sizing issue
- Yang explains SwiftUI layout behavior
- Marc provides screenshots and code examples

**Phase 2: Solution Development (Jul 3-4)**
- Yang implements TruncationDetectingText component
- Solution tested and working
- Marc extends to support two text components

**Phase 3: Business Completion (Jul 4-8)**
- MIT license agreed
- Payment: 150€ via Wise
- Invoice provided

**Phase 4: Bug Discovery (Jul 8 - Dec 10)**
- Layout flickering issue on iPhone XS (iOS 17)
- Multiple debugging attempts
- Issue still under investigation

### 💬 Main Topics
1. **SwiftUI Layout Problem**
   - Text component not returning correct size with lineLimit
   - Need for adaptive layout selection

2. **Solution Implementation**
   - TruncationDetectingText component
   - PreferenceKey-based truncation detection
   - ZStack + LayoutPriority approach

3. **Ongoing Issue**
   - Layout flickering on specific devices
   - Related to text width and timing

### ✅ Decisions Made
- Use MIT license for the solution code
- Payment amount: 150€
- Payment method: Wise transfer

### ⚠️ Action Items
- [ ] Resolve layout flickering on iPhone XS (Marc)
- [ ] Consider writing blog post about solution (Yang)

### 📎 Attachments Summary
9 attachments across thread:
- **AdaptiveAmountRow.swift** - Final solution code
- **TruncationDetectingText.swift** - Core component
- **Invoice - SwiftUI Consultation.pdf** - Paid invoice
- Various screenshots showing flickering issue

### 💡 Key Insights
- Solution works well for main use case
- Edge case issue on specific hardware/iOS version
- Successful commercial collaboration

### Scenario 2: Short Thread (No Truncation)

User: Analyze this thread <short@example.com>
AI:
🧵 Email Thread Analysis

### Thread Overview
- Total emails: 3
- Time span: Jan 10, 2025
- Duration: 1 day
- Participants: Alice, Bob

### Conversation Timeline
1. Alice asks about project timeline
2. Bob provides estimate
3. Alice confirms and schedules meeting

### ⚠️ Action Items
- [ ] Meeting on Jan 15 at 2 PM

### 📎 Attachments Summary
1 attachment: project_timeline.pdf

## Related Skills

- `/analyze-email` - For single email analysis
- `/analyze-attachments` - For deep attachment analysis

## Technical Notes

**Quote Stripping**: Thread reading automatically removes redundant quoted content while preserving context (first 10 lines of each quote block). This reduces token usage by ~80%.

**Token Safety**: Default 1200 chars/email limit ensures threads of up to ~50 emails can be analyzed safely within Claude Code's token limits.
