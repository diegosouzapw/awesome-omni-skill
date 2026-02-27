---
name: email-extractor
description: Expert in email content extraction and analysis. **Use whenever the user mentions .eml files, email messages, says "Extract email information", "Using the email information", or requests to extract, parse, analyze, or process email files.** Handles email thread parsing, attachment extraction, and converting emails to structured markdown format for AI processing. (project, gitignored)
---

# Email Extractor Skill

You are an expert in extracting and analyzing email content from .eml files, converting them to AI-friendly markdown format with proper thread handling and attachment extraction.

## ⚠️ CRITICAL REQUIREMENT: ALWAYS USE FULL FILE PATHS

**YOU MUST ALWAYS use absolute/full paths when working with .eml files.**

✅ **CORRECT:**
```bash
~/.claude/skills/email-extractor/scripts/email-extractor extract /Users/sebastien.morand/Downloads/message.eml
~/.claude/skills/email-extractor/scripts/email-extractor extract ~/Downloads/email.eml -o ~/Documents/extracted
```

❌ **INCORRECT:**
```bash
~/.claude/skills/email-extractor/scripts/email-extractor extract message.eml
~/.claude/skills/email-extractor/scripts/email-extractor extract ./message.eml
```

**Why:** Always use full absolute paths or paths starting with `~/` for the input .eml file to ensure reliable file access.

## Core Capabilities

- Email content extraction from .eml files
- Thread-aware parsing (preserves email conversation flow)
- Attachment extraction (all file types)
- Markdown conversion with proper formatting
- Header parsing (From, To, Cc, Subject, Date)
- HTML email rendering to readable text
- Multi-part email handling
- Smart default output paths based on email subject
- Optional cleanup for temporary extractions

## Quick Start

### Basic Usage

**Binary Location:** `~/.claude/skills/email-extractor/scripts/email-extractor`

```bash
# Basic extraction (creates folder in .eml's directory)
~/.claude/skills/email-extractor/scripts/email-extractor extract /path/to/message.eml

# Extract to specific output directory
~/.claude/skills/email-extractor/scripts/email-extractor extract ~/Downloads/email.eml -o ~/Documents/extracted

# Get help on available commands
~/.claude/skills/email-extractor/scripts/email-extractor --help
~/.claude/skills/email-extractor/scripts/email-extractor extract --help
```

### Default Output Paths

**When no output directory specified:**
- Email: `/path/to/message.eml`
- Output: `/path/to/message_email/` (same directory as .eml)

**When custom output directory specified:**
- Email: `/path/to/message.eml`
- Custom output: `/target/`
- Final output: `/target/message_email/` (sanitized subject or filename appended)

**Examples:**
```bash
# Extract ~/Downloads/original_msg.eml → Output: ~/Downloads/original_msg_email/
~/.claude/skills/email-extractor/scripts/email-extractor extract ~/Downloads/original_msg.eml

# Extract to custom location → Output: ~/Documents/extracted/
~/.claude/skills/email-extractor/scripts/email-extractor extract ~/Downloads/original_msg.eml -o ~/Documents/extracted
```

### Output Structure

Every extraction creates:
```
email_name/
├── email.md              # Email content with thread structure
└── attachments/          # Folder containing all attachments (if any)
    ├── document.pdf
    ├── image.png
    └── ...
```

## Common Workflows

### 1. Extract and Analyze Email

```bash
# Extract email
~/.claude/skills/email-extractor/scripts/email-extractor extract ~/Downloads/message.eml

# Read content
cat ~/Downloads/message_email/email.md
ls ~/Downloads/message_email/attachments/
```

**Process:** Extract → Read email.md → Review attachments → Analyze content and thread structure

### 2. Extract Email from Downloads

When user says "We have an email" with a subject/title, emails are in `~/Downloads/` as `.eml` files:

```bash
# Search for .eml files
find ~/Downloads -name "*.eml" -type f

# Extract the matching email
~/.claude/skills/email-extractor/scripts/email-extractor extract ~/Downloads/found_email.eml
```

### 3. Batch Process Multiple Emails

```bash
# Process all .eml files in directory
for eml in ~/Downloads/*.eml; do
    ~/.claude/skills/email-extractor/scripts/email-extractor extract "$eml"
done
```

### 4. Extract to Custom Location

```bash
# Extract to specific output directory
~/.claude/skills/email-extractor/scripts/email-extractor extract ~/Downloads/temp.eml -o /tmp/email-analysis
```

## Email Markdown Format

The generated `email.md` file includes:

### Header Section
- **From:** Sender name and email
- **To:** Recipients
- **Cc:** Carbon copy recipients (if any)
- **Date:** Sent date/time
- **Subject:** Email subject line

### Thread Structure

Emails are organized respecting the conversation thread:

```markdown
# Email: [Subject]

## Metadata
- **From:** John Doe <john@example.com>
- **To:** Jane Smith <jane@example.com>
- **Date:** 2024-11-11 10:30:00
- **Subject:** Project Update

## Attachments
- document.pdf (attachments/document.pdf)
- image.png (attachments/image.png)

---

## Message Thread

### Latest Message (2024-11-11 10:30)
**From:** John Doe

[Message content]

---

### Previous Message (2024-11-10 15:20)
**From:** Jane Smith

[Message content]
```

## Binary Details

### How It Works

The `email-extractor` is a compiled Go binary that:
- Parses .eml files using Go's native email libraries
- Extracts email content, headers, and attachments
- Generates structured markdown output
- Handles multi-part emails and complex MIME structures
- No runtime dependencies required (static binary)

### Available Commands

```bash
~/.claude/skills/email-extractor/scripts/email-extractor extract <input_eml> [flags]
```

**Command: extract**
- `<input_eml>` (required): Full path to .eml file
- `-o, --output` (optional): Output directory path
- `-h, --help`: Show help for extract command

**Global Commands:**
- `--help`: Show all available commands and global options
- `version`: Show binary version information

## Prerequisites & Setup

### Required
- **None** - The binary is statically compiled and has no runtime dependencies

### Installation

The binary is already compiled and located in:
```
~/.claude/skills/email-extractor/scripts/email-extractor
```

To rebuild from source (if needed), see the CLAUDE.md file in this skill directory.

## Response Approach

When helping with email extraction:

1. **Understand task:** What information needed? Are attachments important? Need thread structure?
2. **Locate email:** Check mentioned locations, search ~/Downloads for .eml files
3. **Extract content:** Use appropriate method (cleanup vs. permanent)
4. **Process:** Read email.md, identify attachments, understand thread flow
5. **Provide results:** Summarize email content, list attachments, highlight key information
6. **Clean up:** Note extraction location, provide commands for further analysis

## Performance & Best Practices

**Performance:**
- Near-instant extraction (compiled binary)
- No setup time required
- No runtime dependencies
- No API calls or external services
- Minimal memory footprint

**When to use default path:** Single emails, permanent archives, files organized alongside .eml files

**When to use `-o` flag:** Multiple extractions, analysis projects, separating source and processed files, organizing output in specific directories

## Troubleshooting

### Common Issues

**"Email file not found":**
```bash
ls -lh /path/to/file.eml
find ~/Downloads -name "*.eml" -type f
```

**"Cannot decode email":**
- Email might be corrupted
- Try opening in email client first
- Check file size: `ls -lh file.eml`

**Binary permission issues:**
```bash
# Make binary executable if needed
chmod +x ~/.claude/skills/email-extractor/scripts/email-extractor

# Verify binary works
~/.claude/skills/email-extractor/scripts/email-extractor --help
```

## Integration with Topic Management

This skill integrates with the **topic-manager** skill:

**Trigger Keywords:**
- User says "Extract email information"
- User says "Using the email information"
- User says "Update topic [name] with email" or "Update topic [name] using the email"
- User mentions .eml files in context of topic updates

**Workflow when updating topics with email information:**
1. **Invoke email-extractor skill** to extract email content and attachments
2. **Upload email.md** to topic's Emails folder using `google-drive-manager`
3. **Upload attachments** to appropriate topic folders:
   - Presentations (PPT, PDF) → Prez folder
   - Audio/video → Records folder
   - Other documents → Misc or appropriate location
4. **Extract content from attachments** if needed:
   - Use `pdf-extractor` for PDFs and presentations
   - Use `speech-to-text` for audio/video files
5. **Analyze email content** to identify:
   - Meeting attendees (From, To, Cc fields)
   - Date and subject
   - Key decisions and action items
   - Risks mentioned
6. **Reference email in topic's minutes** with link to uploaded email.md
7. **Update topic Google Doc** with extracted information

**Example Flow:**
```
User: "Update topic 'Q2 Planning' using the email information"

Assistant Process:
1. Search ~/Downloads for .eml file matching "Q2 Planning"
2. Run: email-extractor extract ~/Downloads/q2_planning.eml
3. Review email.md and attachments
4. Upload email.md to topic's Emails folder
5. Process any attachments (extract PDFs, etc.)
6. Update topic with extracted information
```

## Use Cases

- Extract meeting invitations with attachments
- Parse email threads for topic updates
- Analyze email conversations for action items
- Extract presentation files from emails
- Process email archives for documentation
- Prepare email content for AI analysis
- Convert email threads to readable markdown
