---
name: mailbox-cli
displayName: "Mailbox CLI"
version: 0.0.0
description: Node.js CLI for IMAP/SMTP email management with a stable JSON output contract

# OpenClaw discovery keywords (keep this list short)
keywords:
  - mailbox
  - email
  - imap
  - smtp
  - cli
  - automation
  - openclaw
  - agent
  - sync
  - inbox
---

# Mailbox CLI

Install:

```bash
npm install -g @leeguoo/mailbox-cli
mailbox --help
```

Automation:

- Use `--json` and validate the top-level `success`/`error` fields.
- `error` is structured: `{ code, message, detail? }`.
- Destructive commands default to dry-run unless `--confirm` is provided.
- Contract docs: `docs/CLI_JSON_CONTRACT.md`
