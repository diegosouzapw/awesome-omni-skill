---
name: sandboxes-js-sdk
description: Use the Koyeb Sandbox JS SDK (@koyeb/sandbox-sdk) to create and manage sandboxes programmatically from Node.js. Use when the user wants JavaScript/TypeScript sandbox automation.
license: MIT
compatibility: Requires Node.js, npm/pnpm, and network access.
metadata:
  author: koyeb
  version: "0.0.2"
---

# Koyeb Sandboxes (JS SDK)

## When to use

Use this skill when the user asks to manage Koyeb sandboxes from JavaScript or TypeScript using the official SDK.

## Prerequisites

- Node.js environment
- Package installed: `@koyeb/sandbox-sdk`
- API token available as `KOYEB_API_TOKEN`

## Workflow

1. Install the SDK package.
2. Configure authentication using `KOYEB_API_TOKEN`.
3. Create, start, or stop sandboxes as needed.
4. Execute commands or retrieve logs/status through the SDK.
5. Clean up sandboxes when finished.

## Examples

- Install the SDK: `npm install @koyeb/sandbox-sdk`
- Use token auth: set `KOYEB_API_TOKEN` in your environment

## References

For detailed flags and SDK usage, see references/koyeb-sandbox-js-sdk.md.

- references/koyeb-sdk-auth.md
- references/koyeb-sandbox-js-sdk.md
