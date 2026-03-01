---
name: notmcp
description: Local tool system for API integrations and automation. Use when connecting to external services, fetching data from APIs, or performing tasks that require credentials or network access.
---

# notmcp

You have access to a local toolbox of executable scripts. These tools let you interact with external APIs and services on behalf of the user.

## Discovering Tools

To see what tools are available:

```bash
~/.claude/skills/notmcp/bin/notmcp list
```

To search for tools by keyword:

```bash
~/.claude/skills/notmcp/bin/notmcp search <query>
```

## Running Tools

Run tools with JSON input:

```bash
~/.claude/skills/notmcp/bin/notmcp run <tool-name> --input '{"key": "value"}'
```

Tools return JSON to stdout. Exit code 0 means success.

If a tool doesn't need input, you can omit the `--input` flag:

```bash
~/.claude/skills/notmcp/bin/notmcp run <tool-name>
```

## Handling Credentials

If a tool needs credentials (API keys, tokens), it will fail with an error like:

```
Error: Missing credential(s): POSTHOG_API_KEY
```

When this happens:
1. Ask the user for the credential value
2. Store it securely:

```bash
echo "the-secret-value" | ~/.claude/skills/notmcp/bin/notmcp creds set CREDENTIAL_NAME
```

To see what credentials are already stored:

```bash
~/.claude/skills/notmcp/bin/notmcp creds list
```

## Connecting to Services

When the user wants to connect to a service, guide them **interactively, one step at a time**. Wait for the user to complete each step before proceeding to the next.

### Connection Process (Interactive)

**Step 1: Open the credentials page**
- Run `open "URL"` to open the page in their browser
- Tell them what page opened and wait for confirmation they see it

**Step 2: Guide them through the UI (one action at a time)**
- Give ONE instruction, then wait for them to do it
- Don't dump all steps at once - be conversational
- Example: "Click 'Generate new token'" → wait → "Now copy the token that appears"

**Step 3: Collect the credential**
- Ask them to paste the token/key
- Store it immediately when they provide it

**Step 4: Verify silently**
- Run a verification API call
- Only tell them if it fails - if it works, just confirm "Connected!"

### Important: Be Interactive

BAD (dumping everything):
```
Here's how to connect:
1. Go to URL
2. Click X
3. Click Y  
4. Copy Z
5. Paste it here
```

GOOD (interactive):
```
I'll open the GitHub tokens page for you.
[opens browser]
Let me know when you see the page.

[user: "ok I see it"]

Click "Generate new token" at the top.

[user: "done"]

Now copy the token that appears and paste it here.
```

### Verifying Connections

After storing credentials, verify they work by making a simple API call:
- Most REST APIs have a /me, /user, or /auth endpoint
- Use the http-get tool or make a quick urllib request
- If it fails, troubleshoot with the user before proceeding
- If it works, just say "Connected!" - don't over-explain

### Common Services

**Google (Gmail, Drive, Calendar)** - App Password
- URL: https://myaccount.google.com/apppasswords
- Requires 2FA enabled on the account
- Guide: "Select 'Other (Custom name)', enter 'notmcp', click Generate"
- Store as: GOOGLE_APP_PASSWORD and GOOGLE_EMAIL

**GitHub** - Personal Access Token
- URL: https://github.com/settings/tokens/new?description=notmcp&scopes=repo,user
- Guide: "Click 'Generate token' and copy it"
- Store as: GITHUB_TOKEN
- Verify: GET https://api.github.com/user with Authorization header

**Slack** - Bot Token
- URL: https://api.slack.com/apps
- Guide: "Create New App → From scratch → OAuth & Permissions → Install to Workspace"
- Store as: SLACK_BOT_TOKEN
- Verify: POST https://slack.com/api/auth.test

**Notion** - Integration Token
- URL: https://www.notion.so/my-integrations
- Guide: "New integration → Name 'notmcp' → Copy the Internal Integration Secret"
- Store as: NOTION_TOKEN
- Remind user to share specific pages with the integration

**OpenAI** - API Key
- URL: https://platform.openai.com/api-keys
- Guide: "Create new secret key → Copy it"
- Store as: OPENAI_API_KEY
- Verify: GET https://api.openai.com/v1/models

**Linear** - API Key
- URL: https://linear.app/settings/api
- Store as: LINEAR_API_KEY

**Stripe** - Secret Key
- URL: https://dashboard.stripe.com/apikeys
- Store as: STRIPE_SECRET_KEY

### Other Services

For services not listed:
1. Search for their API or Developer documentation
2. Find where to create API keys or tokens
3. Guide the user through the process
4. Store with a descriptive name: {SERVICE}_API_KEY
5. Verify with a simple API call before building tools

## Using Context7 for API Documentation (Optional)

Context7 provides up-to-date API documentation. If CONTEXT7_API_KEY is set, fetch current docs before creating tools to avoid using outdated or hallucinated endpoints:

```bash
~/.claude/skills/notmcp/bin/notmcp run context7-docs --input '{"library": "googleapis/gmail", "topic": "send"}'
```

**To set up Context7:**
1. Open: https://context7.com/dashboard
2. Sign up (free) and copy your API key
3. Store: `echo "xxx" | ~/.claude/skills/notmcp/bin/notmcp creds set CONTEXT7_API_KEY`

Without Context7, you can still create tools using your knowledge, but results may be less accurate for newer APIs.

## Creating Tools

**Only create new tools when the user explicitly asks** (e.g., "save this as a tool", "make this reusable", "create a tool for this").

To create a new tool:

```bash
~/.claude/skills/notmcp/bin/notmcp create tool-name
```

This creates a template at `~/.claude/skills/notmcp/scripts/tool-name.py`.

Then edit the script to implement the tool logic. Follow these conventions:

### Tool Contract

- **Input**: JSON via stdin (parsed with `json.load(sys.stdin)`)
- **Output**: JSON to stdout (use `print(json.dumps(result))`)
- **Logs**: Write debug info to stderr
- **Exit code**: 0 for success, nonzero for failure
- **Dependencies**: Use Python stdlib only (`urllib.request`, `json`, `os`, etc.)

### Tool Header Format

Every tool must have a docstring header declaring its metadata:

```python
#!/usr/bin/env python3
"""
name: tool-name
description: What this tool does (one line)
credentials:
  - API_KEY_NAME
  - ANOTHER_SECRET
input:
  param1: string (required)
  param2: int (optional, default 10)
output:
  result: description of output
"""
```

### Example Tool Structure

```python
#!/usr/bin/env python3
"""
name: example-api
description: Fetch data from Example API
credentials:
  - EXAMPLE_API_KEY
input:
  query: string (required)
output:
  results: list of matching items
"""

import json
import os
import sys
from urllib.request import urlopen, Request

def main():
    # Read input
    inp = json.load(sys.stdin) if not sys.stdin.isatty() else {}
    
    # Get credentials (injected by notmcp run)
    api_key = os.environ["EXAMPLE_API_KEY"]
    
    # Make API call
    query = inp.get("query", "")
    req = Request(f"https://api.example.com/search?q={query}")
    req.add_header("Authorization", f"Bearer {api_key}")
    
    response = urlopen(req)
    data = json.loads(response.read())
    
    # Return result
    print(json.dumps({"results": data["items"]}))

if __name__ == "__main__":
    main()
```

## Best Practices

1. **Handle pagination** - Don't return unbounded results. Implement limits and cursors.
2. **Handle rate limits** - Add delays or retry logic for APIs with rate limits.
3. **Return compact output** - Summarize large responses. Agents work better with concise data.
4. **Fail gracefully** - Return `{"error": "message"}` with a helpful error description.
5. **Use stdlib** - Avoid pip dependencies so tools are portable and self-contained.
