---
name: mcp-security
description: This skill should be used when the user asks to "secure Notion MCP", "protect OAuth token", "MCP security best practices", "prevent token exposure", "credential management", or needs guidance on securing Notion MCP integration. Covers credential storage, validation, and security considerations.
version: 0.1.0
---

# MCP Security Guide

## Core Security Principles

Securing Notion MCP requires protecting three critical elements:

1. **OAuth Token** - The credential that grants access to your Notion workspace
2. **Token Storage** - Where the token is kept (`.local.md` file)
3. **Token Usage** - How and when the token is used in commands

Notion MCP is more secure than traditional API token approaches because:
- Tokens are user-based (tied to your Notion account)
- You explicitly approve permissions (OAuth consent)
- Tokens can be revoked instantly in Notion settings
- Tokens never leave your local machine

## Token Storage Security

### Where Tokens Are Stored

Tokens are stored in: `.claude/notion-mcp.local.md`

**File location varies by OS:**
- **Linux/Mac:** `~/.claude/notion-mcp.local.md`
- **Windows:** `C:\Users\[username]\.claude\notion-mcp.local.md`

### Why This Location is Safe

**Protected by `.gitignore`:**
- Automatically excluded from git commits
- Cannot be accidentally pushed to GitHub
- Safe if you push other code to public repos

**Protected by file permissions:**
- File is readable only by your user (on Unix systems)
- Other users on system cannot read
- System processes cannot access

**Not in sensitive locations:**
- Not in browser cache
- Not in environment variables
- Not in command history
- Not in logs

### What NOT to Do

❌ **Never:**
- Commit `.local.md` to git (even private repos)
- Share the token with others
- Post token in issues, Slack, email
- Store in environment variables
- Pass token as command argument
- Log token values
- Copy to cloud storage or backups

### Token Content Example

```yaml
---
notion_oauth_token: "notion_oauth_1234567890abcdefghijklmnopqrst"
workspace_id: "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
last_validated: "2026-01-29T10:30:00Z"
---

# Notion MCP Configuration

This file contains your Notion OAuth token. Keep it private!
```

Only the `notion_oauth_token` value is secret. The other fields are metadata.

## Pre-Tool-Use Validation Hook

Every Notion command validates the token before execution.

### How It Works

```
User runs: /notion:read [page-id]
    ↓
Pre-tool-use hook checks:
    1. Token file exists?
    2. Token not empty?
    3. Token starts with "notion_oauth_"?
    ↓
  ├─ All checks pass: Proceed
  └─ Any check fails: Block and suggest /notion:setup
```

### Why This Matters

**Prevents invalid token usage:**
- Won't waste time sending requests with bad tokens
- Gives clear error instead of cryptic API error
- Suggests immediate fix (`/notion:setup`)

**Prevents token logging:**
- Hook never logs actual token value
- Error messages don't expose token
- Debugging is safe

### What You See

If token is invalid:
```
Error: Notion OAuth token not found
Suggestion: Run /notion:setup to authenticate with Notion
```

Never shows token value even in error messages.

## OAuth Scopes and Permissions

When you approve OAuth in Notion, you grant:

**Read Scope:**
- Access all pages and databases
- Read content and properties
- Query and search workspace
- Cannot delete or modify

**Write Scope:**
- Create new pages
- Append content to pages
- Modify database entries
- Cannot delete pages or modify schema

**What Claude Code Can Do:**
- Read: ✓ (with your read approval)
- Write: ✓ (with your write approval)
- Delete: ✗ (intentionally restricted for safety)

### Revoking Permissions

You can revoke Claude Code access anytime:

1. Go to Notion → Settings
2. Find "Integrations" or "Connected Apps"
3. Find "Claude Code"
4. Click "Disconnect"

This immediately invalidates the token.

## Token Lifetime

### No Expiration
Notion OAuth tokens do not expire by default. Token remains valid until:
- You manually revoke in Notion settings
- You change your Notion password (invalidates)
- Token is compromised (should revoke)

### Validation Timestamps
The `last_validated` field helps track freshness:
- Updated each time `/notion:validate` runs
- Shows when token was last tested
- If very old (>90 days), consider validating

### When Validation Fails

If `/notion:validate` fails:
- Token may have been revoked
- Workspace access may have changed
- OAuth app may have been deauthorized
- Run `/notion:setup` to get new token

## Compromise Response Plan

If you suspect token is compromised:

### Immediate Actions

1. **Revoke the token immediately**
   - Go to Notion → Settings
   - Find Claude Code app
   - Click "Disconnect"
   - Token becomes instantly invalid

2. **Check Notion activity**
   - Look at Notion activity log
   - See if unauthorized changes were made
   - Check file history for modifications

3. **Get new token**
   - Run `/notion:setup`
   - Complete OAuth flow again
   - New token is issued

### Follow-Up Actions

4. **Change Notion password** (optional but recommended)
   - Ensures attacker cannot use old session

5. **Delete `.local.md` file** (optional)
   - Removes old compromised token from disk
   - `/notion:setup` will create new file

6. **Review workspace access**
   - Check who has access to workspace
   - Verify no unauthorized users
   - Check team permissions

## Network Security

### HTTPS Only
All Notion API calls use HTTPS encryption:
- Token transmitted securely
- Data encrypted in transit
- Cannot be intercepted

### OAuth Code Exchange
During setup, the OAuth flow:
1. Uses HTTPS for all requests
2. OAuth code is valid for only 60 seconds
3. Code is exchanged for token server-side
4. Token never exposed in browser URL

### No Third-Party Access
Notion MCP:
- Connects directly to official Notion API
- Does not route through Claude Code servers
- Data stays between you and Notion

## Workspace Role Considerations

Your Notion workspace role affects what you can do:

| Role | Permissions |
|------|------------|
| Owner | Full read/write |
| Editor | Full read/write |
| Commenter | Read only |
| Viewer | Read only (limited) |

If you have read-only role:
- `/notion:read` and search work
- `/notion:write` and `/notion:create` fail
- Ask workspace owner to upgrade your role

## Multi-User Considerations

If multiple users use same machine:

**Each user should have own token:**
- Each user completes `/notion:setup` separately
- Each gets own `.claude/notion-mcp.local.md`
- Token isolated to that user's account

**Never share tokens:**
- Never give token to colleague
- Never put token in shared docs
- Each user manages their own setup

## Periodic Security Checks

Recommended security practices:

**Monthly:**
- Run `/notion:status` to check configuration
- Verify token is still valid
- Check last validation timestamp

**Quarterly:**
- Review connected apps in Notion settings
- Verify Claude Code app is still listed
- Revoke if accidentally authorized twice

**When Passwords Change:**
- Notion password change: No action needed
- System password change: No action needed
- Password reset: Token still valid, proceed normally

**If Workspace Changes:**
- Member added/removed: No action
- Role changed: May need to re-setup if downgraded
- Access revoked: Token becomes invalid, re-authenticate

## Debugging Without Exposing Token

### Safe Debugging

✅ Check file exists:
```bash
ls -la .claude/notion-mcp.local.md
```

✅ Check file size (should be ~300+ bytes):
```bash
wc -c .claude/notion-mcp.local.md
```

✅ Check frontmatter structure:
```bash
head -5 .claude/notion-mcp.local.md
```

### Unsafe Debugging

❌ Never:
```bash
# DON'T: This shows the actual token
cat .claude/notion-mcp.local.md | grep notion_oauth_token

# DON'T: This sends token to third party
curl -H "Authorization: Bearer $TOKEN" https://external-site.com

# DON'T: This logs token
echo "Token is: $TOKEN"
```

### Run Diagnostic Commands

Instead of manual inspection, use plugin commands:
- `/notion:validate` - Tests token validity safely
- `/notion:status` - Shows configuration (without token)
- `/notion:troubleshoot` - Diagnoses issues safely

These never expose token values.

## Notion OAuth App Details

Claude Code is registered as official Notion integration:

**App Name:** Claude Code Notion MCP
**Permissions:** Read and Write
**Owner:** Anthropic
**Authorization:** User-based (not system-wide)

This is the official integration. Do not:
- Use third-party unofficial Notion integrations
- Create custom OAuth apps for Notion (unless you control them)
- Accept tokens from untrusted sources

## Additional Resources

For more detailed information:

- **`references/oauth-security.md`** - Deep dive on OAuth security
- **`references/token-compromise-guide.md`** - What to do if compromised
- **`references/workspace-permissions.md`** - Understanding Notion permissions
- **`references/audit-trail.md`** - Monitoring token usage

See the notion-mcp-setup skill for OAuth flow details.

See the notion-mcp-tools skill for tool reference.
