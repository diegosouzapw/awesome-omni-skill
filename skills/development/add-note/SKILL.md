---
name: add-note
description: Use this skill whenever important information is learned during a task or when the user explicitly asks to store something. Use when users ask to remember. Triggers on "remember this", "update memory", "share" or any persistent storage request.
---

Be **proactive**, but intentional:

- Store CLI commands, API endpoints, error resolutions, operational gotchas, patterns, and internal processes
- Prefer **small, focused, reusable notes**
- If the information would save time if recalled later, it likely belongs here

---

## What this skill does

Stores a structured knowledge note in Qdrant for retrieval by AI coding agents.

This skill uses the **Qdrant MCP server** via the **`qdrant-add-note`** tool.

---

## Collection

All notes are stored in the following collection: `notes-hybrid`

---

## Storage guardrails (IMPORTANT)

These rules must be followed exactly:

- **Do not store test notes** unless they contain a concrete, reusable learning
- **Context is required** unless the text is completely self-explanatory
- Only use the fields supported by the `qdrant-add-note` tool
- All required fields must be provided

If any rule is violated, do **not** store the note.

---

## Minimum acceptable note

A note must meet at least this standard:

**Required parameters:**

- `text`: concrete command, endpoint, or learning
- `context`: when or why this is useful
- `type`: one of `cli`, `api`, `learning`, `snippet`, or `pattern`
- `created_at`: ISO-8601 timestamp

If you cannot fill in all required fields meaningfully, do not store the note.

---

## Fields and how to use them

### text (required)

The primary knowledge content.

- One command, endpoint, rule, or learning
- Must be understandable without chat history

Good examples:

- `kubectl rollout restart deployment my-app -n prod`
- `Avoid using async forEach in Node.js; it does not await promises`

Bad examples:

- `testing the note system`
- `this worked`

---

### context (required unless self-explanatory)

Explains **when, why, or how** the text is useful.
Include:

- Conditions
- Warnings
- Operational context

If you cannot explain the usefulness, do not store the note.

---

### type (required)

Choose exactly one:

- `cli`
- `api`
- `learning`
- `snippet`
- `pattern`

---

### tool (optional but recommended)

The tool, system, or service involved.

Examples:

- `kubectl`
- `aws`
- `terraform`
- `internal-api`
- `github-actions`

---

### tags (optional)

Array of short, lowercase keywords describing concepts or domains.

Examples:

- `["kubernetes", "deployments"]`
- `["auth", "debugging"]`
- `["terraform", "aws"]`

---

### language (optional)

Use for code or CLI-related notes.

Examples:

- `bash`
- `python`
- `yaml`
- `json`

---

### source (optional)

Where this knowledge applies.

Examples:

- `personal-notes`
- `repo:infra`
- `service:billing`
- `team:platform`

---

### created_at (required)

ISO-8601 timestamp.
Use the current date/time if not explicitly provided.

---

## Tool usage

Use the **`qdrant-add-note`** MCP tool with the following parameters:

**Required:**

- `text` (string)
- `context` (string)
- `type` (string: `cli` | `api` | `learning` | `snippet` | `pattern`)
- `created_at` (string: ISO-8601 timestamp)

**Optional:**

- `tool` (string)
- `tags` (array of strings)
- `language` (string)
- `source` (string)

The tool handles vector generation and storage automatically.

---

## Pre-store quality checklist

Before storing, confirm:

- Would another agent benefit from this in 30 days?
- Is the note understandable without chat history?
- Does `context` clearly explain when or why it applies?
- Is this more than a test or confirmation?

If any answer is **no**, do not store the note.

---

## Agent reminder

When in doubt:
> **Check existing notes first. Store only what improves future decisions.**
