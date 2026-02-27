---
name: agents-md-guidelines
description: Guidelines for writing small, stable AGENTS.md files. Use when creating, refactoring, or reviewing AGENTS.md.
---

# AGENTS.md Guidelines

## Core Principles

- Keep root AGENTS.md as small as possible; it loads on every request.
- Use progressive disclosure: link to deeper docs for details.
- Prefer stable concepts over volatile paths or time-sensitive notes.
- Avoid “ball of mud” growth; don’t add rules for every mishap.
- Keep instructions concise; assume the agent already knows basics.

## Essentials Only

Root AGENTS.md should usually include:
- One-sentence project description
- Package manager if non-default
- Non-standard build/typecheck commands
- Pointers to deeper docs

## Progressive Disclosure

- Move language, testing, and workflow rules into separate docs.
- Link to those docs from AGENTS.md.
- Keep guidance domain-based, not file-path based.
- Avoid deep reference chains; link one level deep when possible.

## Staleness Rules

- Avoid exact file paths unless they are stable interfaces.
- Avoid dates or time-based instructions.
- Prefer capability descriptions over structure hints.

## Monorepo Guidance

- Use root AGENTS.md for shared rules.
- Use package-level AGENTS.md for package-specific conventions.
- Keep each level focused and minimal.

## Instruction Design

- Start with a TL;DR checklist for fast scanning.
- Include clear triggers and skip cases to prevent overuse.
- Provide ordered steps with explicit outputs.
- Use strict format rules with correct/incorrect examples.
- Add a decision tree for edge cases when needed.
- Include validation and debugging tips to reduce failure loops.
- Make naming conventions explicit.
- Define archive or completion criteria.
- Keep terminology consistent.

## Quality Checklist

- Description includes what + when, third-person.
- Root AGENTS.md is short and link-heavy.
- TL;DR + triggers + steps are present when needed.
- No duplicated guidance across files.
- Terminology is consistent.
- Examples are concrete and brief.
- Instructions fit an "instruction budget".
