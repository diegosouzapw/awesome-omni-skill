---
name: create-rules
description: Create or update Cursor Rules (.mdc files) and Skills (SKILL.md). Use when creating rules, adding coding standards, setting up conventions, updating .cursor/rules/, or converting rules to skills. Defines standard format, naming, frontmatter, token budget.
---

# Create Rules and Skills

## Rule Types (3-Tier System)

| Type | Frontmatter | When loaded | Filename suffix |
|------|-------------|-------------|-----------------|
| **Always** | `alwaysApply: true`, description/globs blank | Every conversation | `-always.mdc` |
| **Auto** | `alwaysApply: false`, globs filled | When matching file is open | `-auto.mdc` |
| **Agent** | `alwaysApply: false`, description filled | Agent decides | `-agent.mdc` |
| **Manual** | `alwaysApply: false`, both blank | Only via @ mention | `-manual.mdc` |

## Token Budget (CRITICAL)

- Total always-apply: TARGET < 5,000 tokens
- alwaysApply: true ONLY for rules needed in EVERY chat
- File-specific rules -> auto (globs)
- Task-specific rules -> agent (description) or skill
- **Rule of Three**: codify a rule ONLY after 3 repetitions of the same mistake

## Naming Convention

`{CATEGORY}-rule-name-{always|auto|agent|manual}.mdc`

Categories: core, _base, protocol, standard, workflows, error

## Rule Structure

See template in [rule-template.mdc](assets/rule-template.mdc).

## Description Format (Agent Rules)

ACTION-TRIGGER-OUTCOME format:
"Применять при [TRIGGER]. [ACTION] для [OUTCOME]."

## Universality Requirement

Rules must work in ANY project:
- NO project entities -> use placeholders
- NO project URLs/creds -> .cursor/.secrets/
- NO project architecture -> AGENTS.md
- YES: universal principles, abstract patterns

## When to Create a Skill Instead of a Rule

- Workflow with multiple steps -> Skill
- Needs executable scripts -> Skill
- Long procedural content (>200 lines) -> Skill
- Only relevant for specific tasks -> Skill
- Short constraint needed broadly -> Rule

## After Creating/Updating

1. Re-read the file (self-check)
2. Output: Rule/Skill path, Type, Description
