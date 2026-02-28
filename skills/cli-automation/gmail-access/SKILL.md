---
name: gmail-access
description: Search, read, draft, and reply to Gmail emails using the gmail.cs CLI tool. Use when the user asks about emails, wants to search Gmail, create email drafts, reply to messages, or download attachments.
---

# Gmail Access

Access Gmail accounts securely via the `gmail.cs` CLI tool located in this skill directory. Credentials are stored in 1Password and never exposed.

## Setup

### 1. Create OAuth Credentials

You need to create your own Google OAuth credentials to use this skill.

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project (or select an existing one)
3. Enable the Gmail API for your project
4. Go to "Credentials" > "Create Credentials" > "OAuth client ID"
5. Choose "Desktop app" as the application type
6. Copy the Client ID and Client Secret
7. Replace the placeholder values in both `gmail.cs` and `get-gmail-token.cs`:
   ```csharp
   private const string ClientId = "YOUR_CLIENT_ID.apps.googleusercontent.com";
   private const string ClientSecret = "YOUR_CLIENT_SECRET";
   ```

**Note:** Client secrets for desktop/CLI apps cannot be fully protected per OAuth 2.0 spec. It's safe to commit these to your private repository but avoid public repositories.

### 2. Authorize Your Gmail Account(s)

For each Gmail account you want to access:

```bash
# Run the token setup script
dotnet run get-gmail-token.cs your-email@gmail.com

# This will:
# 1. Open a browser for Google OAuth authorization
# 2. Get a refresh token
# 3. Store it in 1Password as "Gmail - your-email@gmail.com"
```

**Prerequisites:**
- [1Password CLI](https://developer.1password.com/docs/cli/) installed (`brew install 1password-cli`)
- 1Password CLI authenticated

The refresh token allows the tool to access your Gmail without repeated logins (tokens are cached for one hour).

## Quick Reference

```bash
# Search emails
dotnet run gmail.cs -- search --email your-email@gmail.com --query "<gmail query>" --max-results 20

# Create a draft
dotnet run gmail.cs -- draft --email your-email@gmail.com --to "recipient@example.com" --subject "Subject" --body "Body"

# Reply to an email
dotnet run gmail.cs -- reply --email your-email@gmail.com --message-id "<id>" --body "Reply text"

# Download attachments
dotnet run gmail.cs -- download-attachments --email your-email@gmail.com --message-id "<id>" --output-dir "/tmp/attachments"
```

## Commands

### search

Search emails with Gmail query syntax.

```bash
dotnet run gmail.cs -- search --email your-email@gmail.com --query "from:someone@example.com" --max-results 10
```

**Options:**
- `--email` (required): Gmail account to search
- `--query` or `-q`: Gmail search query (supports all Gmail operators)
- `--max-results` or `-m`: Maximum results (default: 20)

**Common query patterns:**
- `from:sender@example.com` - From specific sender
- `subject:invoice` - Subject contains word
- `has:attachment` - Has attachments
- `newer_than:7d` - Last 7 days
- `is:unread` - Unread messages
- `label:important` - Specific label

### draft

Create a new email draft.

```bash
dotnet run gmail.cs -- draft --email your-email@gmail.com --to "recipient@example.com" --subject "Subject" --body "Email content"
```

**Options:**
- `--email` (required): Sender account
- `--to` or `-t` (required): Recipient email
- `--subject` or `-s`: Subject line
- `--body` or `-b` (required): Email body content

### reply

Create a reply draft to an existing email.

```bash
dotnet run gmail.cs -- reply --email your-email@gmail.com --message-id "18abc123def" --body "Reply text"
```

**Options:**
- `--email` (required): Account to reply from
- `--message-id` or `-m` (required): Gmail message ID to reply to (from search results)
- `--body` or `-b` (required): Reply content

### download-attachments

Download all attachments from an email.

```bash
dotnet run gmail.cs -- download-attachments --email your-email@gmail.com --message-id "18abc123def" --output-dir /tmp/attachments
```

**Options:**
- `--email` (required): Account
- `--message-id` or `-m` (required): Gmail message ID
- `--output-dir` or `-o` (required): Directory to save attachments

## Output Format

All commands output JSON. Success responses include `"success": true`. Example search result:

```json
{
  "success": true,
  "count": 2,
  "messages": [
    {
      "id": "18abc123def",
      "threadId": "18abc000000",
      "from": "sender@example.com",
      "to": "your-email@gmail.com",
      "subject": "Example Subject",
      "date": "Mon, 1 Jan 2025 10:00:00 +0100",
      "snippet": "Preview of email content...",
      "body": "Full email body text"
    }
  ]
}
```

## Authentication

First use per hour requires 1Password authorization (Touch ID). Subsequent calls use cached token. If token expired, you'll see a Touch ID prompt.

## Writing Ephemeral Programs

When writing programs that use gmail.cs, use CliWrap:

```csharp
#:package CliWrap@3.6.6

using CliWrap;
using CliWrap.Buffered;

var result = await Cli.Wrap("dotnet")
    .WithArguments(["run", "gmail.cs", "--", "search", "--email", "your-email@gmail.com", "--query", "is:unread"])
    .WithWorkingDirectory("/path/to/gmail/skill")  // Set to your gmail skill directory
    .ExecuteBufferedAsync();

var json = result.StandardOutput;
```
