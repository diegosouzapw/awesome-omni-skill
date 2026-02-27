---
description: Review UI code for Web Interface Guidelines compliance. Use when asked
  to "review my UI", "check accessibility", "audit design", "review UX", or "check
  my site against best practices".
name: web-design-guidelines
---

# Web Interface Guidelines

> **💡 MCP Tool Available**: Use **Context7**, **Tavily**, **BraveSearch**, or **Serper.dev** first; only if those fail, use **WebSearch** or **WebFetch** as needed.

Review files for compliance with Web Interface Guidelines.

## How It Works

1. Use Context7, Tavily, BraveSearch, or Serper.dev to load the latest Web Interface Guidelines when needed; only if those fail, use WebSearch or WebFetch as needed.
2. Read the specified files (or prompt user for files/pattern)
3. Check against all rules in the guidelines
4. Output findings in the terse `file:line` format

## Guidelines Source

Use Context7, Tavily, BraveSearch, or Serper.dev to query the Web Interface Guidelines documentation; only if those fail, use WebSearch or WebFetch as needed. The guidelines contain all rules and output format instructions.

## Usage

When a user provides a file or pattern argument:
1. Use Context7, Tavily, BraveSearch, or Serper.dev (or in-IDE docs) to load the relevant guidelines; only if those fail, use WebSearch or WebFetch as needed
2. Read the specified files
3. Apply all rules from the guidelines
4. Output findings using the format specified in the guidelines

If no files specified, ask the user which files to review.

---

## Related Skills

| Skill | When to Use |
|-------|-------------|
| **[frontend-design](../frontend-design/SKILL.md)** | Before coding - Learn design principles (color, typography, UX psychology) |
| **web-design-guidelines** (this) | After coding - Audit for accessibility, performance, and best practices |

## Design Workflow

```
1. DESIGN   → Read frontend-design principles
2. CODE     → Implement the design
3. AUDIT    → Run web-design-guidelines review ← YOU ARE HERE
4. FIX      → Address findings from audit
```
