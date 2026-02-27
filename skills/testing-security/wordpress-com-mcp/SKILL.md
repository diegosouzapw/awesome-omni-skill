---
name: wordpress-com-mcp
description: Connect Codex or other MCP-capable clients to WordPress.com and run content/site/account tasks through the WordPress.com MCP server. Use when a user asks to connect WordPress.com, authorize access, inspect sites/posts/pages/media/comments/subscribers through MCP tools, or troubleshoot WordPress.com OAuth/MCP setup.
---

# WordPress.com MCP

Create or troubleshoot a WordPress.com MCP connection by following this sequence.

## 1) Confirm connection mode

Choose one of these paths:

1. Use the hosted connector flow in Claude (recommended when the user is in Claude and wants the fastest setup).
2. Use a custom MCP client integration (use for Codex-compatible clients that can add external MCP servers).

## 2) Gather prerequisites

Collect before setup:

- A WordPress.com account with access to the target site(s).
- MCP access enabled in WordPress.com (paid plans only).
- A client that supports MCP plus OAuth 2.1 Authorization Code with PKCE.
- The official endpoints from `references/endpoints.md`.

## 3) Connect in Claude connector flow

If the user is in Claude and asks for connector setup:

1. Enable MCP access in WordPress.com settings.
2. Open Claude settings, then Connectors, then browse connectors.
3. Search for `WordPress.com` and connect.
4. Sign in to WordPress.com in the OAuth window.
5. Approve requested access.
6. Verify with:
   - `Show me my site's traffic for the last 30 days.`

Treat Claude connector access as read-only per WordPress.com announcement guidance.

## 4) Connect in a custom MCP client

Use this for Codex-compatible clients with manual MCP server configuration.

1. Register an OAuth client (dynamic registration endpoint in `references/endpoints.md`).
2. Generate PKCE `code_verifier` and `code_challenge` (`S256`).
3. Open WordPress.com authorization URL with registered `redirect_uri`.
4. Exchange authorization code for an access token.
5. Send MCP `tools/call` requests to the MCP endpoint with `Authorization: Bearer <token>`.
6. Verify with:
   - `Show my latest 5 posts for <site>.`

## 5) Validate tool access

Run at least one read operation in each relevant area:

- Site discovery: list sites, choose target site.
- Content reads: posts/pages/media/comments.
- Audience reads: followers/subscribers when relevant.

If an operation fails, capture exact error text and move to troubleshooting.

## 6) Troubleshoot quickly

- `Unauthorized` or repeated login loops:
  - Ensure OAuth 2.1 + PKCE flow is used and redirect URI exactly matches registration.
- No tools visible after connect:
  - Re-check MCP enablement, then reconnect and re-authorize.
- Missing expected sites/content:
  - Confirm account/site permissions in WordPress.com.

## 7) Prompt templates

Use or adapt:

- `List all my WordPress.com sites and summarize key stats for each.`
- `Show draft posts for <site> from the last 30 days.`
- `Summarize recent comments across <site>.`
- `Which posts on <site> have not been updated in over a year?`

## 8) Safety behavior

Default to read-oriented workflows. Before any write-like or destructive action, confirm user intent and available tool permissions.

## 9) Publish posts via REST API fallback

If MCP is connected but does not expose write tools, publish through WordPress.com REST API using the same OAuth token.

1. Confirm user intent (title/content/status and target site).
2. Use `scripts/publish_post.py`:
   - `python3 scripts/publish_post.py --site tomwillmot.com --title "My title" --content "<p>Hello</p>" --status publish`
3. Pass token via env var or argument:
   - `WPCOM_ACCESS_TOKEN='<token>' python3 scripts/publish_post.py ...`
4. Verify success from returned JSON:
   - Check `ID`, `URL`, and `status`.
5. If publish fails with permission errors:
   - Confirm the token owner has author/editor privileges on the target site.
   - Re-authorize OAuth and retry.
