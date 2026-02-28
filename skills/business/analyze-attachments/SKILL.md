---
name: analyze-attachments
description: Deep attachment analysis with extraction, importance classification, and content analysis for emails and threads.
---

# Analyze Attachments

Comprehensive attachment analysis including extraction, importance classification, content analysis, and intelligent cleanup.

## When to Use

Use this skill when you need to:
- Extract and view email attachments
- Analyze document content (PDFs, contracts, invoices)
- Identify important files and deadlines
- Understand attachment context in conversation
- Classify attachments by importance

**Triggers**:

English:
- "Analyze attachments in `<message-id>`"
- "Extract attachments from `<message-id>`"
- "What files are attached `<message-id>`"
- "Show me attachments `<message-id>`"
- "Analyze the files in this email `<message-id>`"

Chinese:
- "分析附件 `<message-id>`"
- "提取邮件附件 `<message-id>`"
- "这封邮件有什么附件 `<message-id>`"
- "查看附件内容 `<message-id>`"
- "附件分析 `<message-id>`"

## Invocation Methods

### Method 1: Natural Language (Recommended)

```bash
User: Analyze attachments <E42FF77C-C563-466D-8544-3E0C16EA24EC@taler.net>
AI: [Proceeds to analyze immediately]
```

### Method 2: Slash Command

```bash
User: /analyze-attachments
AI: 请提供邮件 Message-ID
User: <E42FF77C-C563-466D-8544-3E0C16EA24EC@taler.net>
AI: [Proceeds to analyze]
```

> **Note**: Can analyze attachments from single emails or entire threads.

## Operation Modes

### 1. Quick Mode (Default) ⚡

**Token Usage: Minimal**

- List all attachments (filename, type, size)
- Classify by importance
- Identify context
- ❌ Do NOT extract content

**Use when**: Just need to know what's attached

### 2. Interactive Mode 🔄

**Token Usage: On-demand**

- Show attachment list with importance
- Ask which to extract
- Extract only selected files
- Analyze content

**Use when**: Selective analysis needed

### 3. Auto Mode 🤖

**Token Usage: Higher**

- Auto-extract high-importance files
- Full content analysis
- Extract deadlines and action items
- Auto-cleanup temporary files

**Use when**:
- Email from important senders (HMRC, IRS, legal, finance)
- Subject keywords (invoice, contract, urgent, tax)
- User explicitly requests full analysis

## Execution Steps

### Step 1: Get Message-ID

```bash
Please provide the email Message-ID

💡 Quick method (recommended):
In Mail.app: Select email → Press shortcut (⌘⇧C) → Message-ID copied
See README "Setup Mail Quick Action" section if not configured
```

### Step 2: Read Email Data

```python
email_data = mcp__mail__read_email(message_id)
```

### Step 3: Analyze Attachments

Check if email is part of thread:
```python
thread_paths = mcp__mail__get_thread_paths(message_id)
if len(thread_paths) > 1:
    # This is a thread
    # Ask user: analyze this email only or entire thread?
```

### Step 4: Classify Attachments

Importance levels:

**🚨 High Importance**
- Government notices (HMRC, IRS, tax documents)
- Legal documents (contracts, agreements, NDAs)
- Financial documents (invoices, receipts, bills)
- Certificates (licenses, permits)
- File type: `.pdf` from official sources

**⚠️ Medium Importance**
- Technical documentation
- Reports and analysis
- Spreadsheets and presentations
- File types: `.pdf`, `.docx`, `.xlsx`, `.pptx`

**💡 Low Importance**
- Images (unless specified as evidence)
- Signatures
- Plain text files
- File types: `.png`, `.jpg`, `.txt`, `.mov`

### Step 5: Choose Mode Based on Context

**Auto Mode triggers**:
- Sender contains: "hmrc", "irs", "tax", "court", "legal"
- Subject contains: "invoice", "contract", "urgent", "deadline", "tax"
- High-importance attachments detected

**Interactive Mode**:
- Mixed importance attachments
- User seems uncertain

**Quick Mode**:
- Only low-importance attachments
- User just browsing

### Step 6: Execute Based on Mode

**Quick Mode**: Just list and classify

**Interactive Mode**:
```bash
Found 3 attachments:

🚨 High Importance:
1. invoice_2025.pdf (2.3 MB)

⚠️ Medium Importance:
2. report.docx (156 KB)

💡 Low Importance:
3. screenshot.png (45 KB)

Would you like me to extract and analyze the high-importance file?
```

**Auto Mode**:
```bash
Auto-extracting high-importance attachments...
[Extract and analyze]
```

### Step 7: Extract and Analyze (if applicable)

```python
# Extract attachments
attachments = mcp__mail__extract_attachments(message_id, filenames)

# Analyze content
for file_path in extracted_files:
    content = read_file(file_path)
    analysis = analyze_content(content)
```

### Step 8: Cleanup

**CRITICAL**: Always cleanup after analysis

```python
mcp__mail__cleanup_attachments(message_ids=[message_id])
```

## Output Format

### Quick Mode Output

```markdown
📧 Email Attachments

### Email Information
- From: [sender]
- Subject: [subject]
- Date: [date]

### 📎 Attachments Found ([N] files)

🚨 High Importance ([X]):
1. [filename] ([type]) - [size]
   - Reason: [why it's important]

⚠️ Medium Importance ([Y]):
2. [filename] ([type]) - [size]
   - Context: [usage]

💡 Low Importance ([Z]):
3. [filename] ([type]) - [size]
```

### Interactive/Auto Mode Output

```markdown
📧 Attachment Analysis

### Email Information
- From: [sender]
- Subject: [subject]
- Date: [date]

### 📎 Extracted Files Analysis

🚨 High Importance Files:

**1. invoice_20250115.pdf**
- Type: PDF document
- Size: 2.3 MB
- **Content Summary**:
  - Invoice number: INV-2025-001
  - Amount: £1,500.00
  - Due date: January 30, 2025
  - Status: Pending payment
- **⚠️ Action Required**: Payment due in 15 days

⚠️ Medium Importance Files:

**2. project_report.docx**
- Type: Word document
- Size: 156 KB
- **Content Summary**:
  - Q4 performance report
  - Key metrics and KPIs
- **💡 Recommendation**: Review for quarterly planning

💡 Low Importance Files:

**3. screenshot.png**
- Type: Image
- Size: 45 KB
- Context: Visual reference

### 📅 Important Dates
- Jan 30, 2025: Invoice payment due

### ⚠️ Action Items
- [ ] Pay invoice INV-2025-001 by Jan 30

✅ Temporary files cleaned up
```

## Example Scenarios

### Scenario 1: Tax Notice (Auto Mode)

User: Analyze attachments <tax-notice@example.com>

AI detects:
- Sender: notifications@hmrc.gov.uk
- Subject: "Tax Return Due 15 January"

AI: Auto Mode triggered
📧 Attachment Analysis

### 📎 Extracted Files Analysis

🚨 High Importance Files:

**1. tax_return_2025.pdf**
- Type: PDF document
- Size: 845 KB
- **Content Summary**:
  - Tax year: 2024-2025
  - Total tax due: £3,450.00
  - Deadline: 15 January 2026
  - Payment reference: XHJ1234567890
- **⚠️ Action Required**: Submit return and pay by Jan 15

### 📅 Important Dates
- ⚠️ Jan 15, 2026: Tax return deadline (14 days from now)

### ⚠️ Action Items
- [ ] Complete tax return
- [ ] Pay £3,450.00 by Jan 15
- [ ] Use reference: XHJ1234567890

✅ Temporary files cleaned up

### Scenario 2: Mixed Attachments (Interactive Mode)

User: /analyze-attachments
AI: 请提供邮件 Message-ID
User: <project-update@example.com>
AI:
📧 Email Attachments

### 📎 Attachments Found (4 files)

🚨 High Importance (1):
1. contract_draft.pdf (1.8 MB)
   - Legal document requiring review

⚠️ Medium Importance (2):
2. budget_2025.xlsx (245 KB)
   - Financial planning spreadsheet
3. project_timeline.docx (89 KB)
   - Schedule document

💡 Low Importance (1):
4. team_photo.jpg (1.2 MB)
   - Team picture

💡 Would you like me to extract and analyze the contract draft?

User: Yes, please

AI: [Extracts and analyzes contract]

### Scenario 3: Thread with Attachments

User: Analyze attachments <thread-start@example.com>

AI: This email is part of a thread (15 emails).

Would you like to:
1. Analyze attachments from this email only
2. Analyze all attachments from the entire thread

User: Option 2

AI: [Analyzes attachments across thread, shows evolution]

📎 Thread Attachment Evolution

**Version History**:
1. contract_v1.pdf (Email 1) - Initial draft
2. contract_v2.pdf (Email 5) - Revised terms
3. contract_final.pdf (Email 12) - ⚠️ Final version

🚨 Most Recent: contract_final.pdf
- All parties have signed
- Effective date: February 1, 2025

## Important Reminders

1. **Always cleanup** temporary files after analysis
2. **Respect user token preferences** - default to quick mode
3. **Provide context** - explain why attachments are important
4. **Track thread history** - show attachment evolution
5. **Be proactive** - suggest auto mode for important emails

## Related Skills

- `/analyze-email` - For single email analysis
- `/analyze-thread` - For complete thread analysis
