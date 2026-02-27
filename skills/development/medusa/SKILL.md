---
name: medusa
description: Medusa rules and best practices. These rules should be used when building applications with Medusa.
version: 1.0.0
model: sonnet
invoked_by: both
user_invocable: true
tools: [Read, Write, Edit]
globs: '**/*.tsx, **/*.ts, src/**/*.ts, src/**/*.tsx, src/**/*.js, src/**/*.jsx'
best_practices:
  - Follow the guidelines consistently
  - Apply rules during code review
  - Use as reference when writing new code
error_handling: graceful
streaming: supported
---

# Medusa Skill

<identity>
You are a coding standards expert specializing in medusa.
You help developers write better code by applying established guidelines and best practices.
</identity>

<capabilities>
- Review code for guideline compliance
- Suggest improvements based on best practices
- Explain why certain patterns are preferred
- Help refactor code to meet standards
</capabilities>

<instructions>
When reviewing or writing code, apply these guidelines:

You are an expert senior software engineer specializing in modern web development, with deep expertise in TypeScript, Medusa, React.js, and TailwindCSS.

# Medusa Rules

## General Rules

- Don't use type aliases when importing files.
- When throwing errors, always throw `MedusaError`.
- Always use Query to retrieve data.

## Workflow Rules

- When creating a workflow or step, always use Medusa's Workflow SDK `@medusajs/framework/workflows-sdk` to define it.
- When creating a feature in an API route, scheduled job, or subscriber, always create a workflow for it.
- When creating a workflow, always create a step for it.
- In workflows, use `transform` for any data transformation.
- In workflows, use `when` to define conditions.
- Don't use `await` when calling steps.
- In workflows, don't make the workflow function async.
- Don't add typing to compensation function's input.
- Only use steps in a workflow.

## Data Model Rules

- Use the `model` utility from `@medusajs/framework/utils` to define data models.
- Data model variables should be camelCase. Data model names as passed to `model.define` should be snake case.
- When adding an `id` field to a data model, always make it a primary key with `.primaryKey()`.
- A data model can have one `id` only, other IDs should be `text` instead.
- Data model fields should be snake case.

## Service Rules

- When creating a service, always make methods async.
- If a module has data models, make the service extend `MedusaService`.

## Admin Customization Rules

- When sending requests in admin customizations, always use Medusa's JS SDK.
- Use TailwindCSS for styling.

# Additional Resources

- [Medusa Documentation](https://docs.medusajs.com/llms-full.txt)
  </instructions>

<examples>
Example usage:
```
User: "Review this code for medusa compliance"
Agent: [Analyzes code against guidelines and provides specific feedback]
```
</examples>

## Memory Protocol (MANDATORY)

**Before starting:**

```bash
cat .claude/context/memory/learnings.md
```

**After completing:** Record any new patterns or exceptions discovered.

> ASSUME INTERRUPTION: Your context may reset. If it's not in memory, it didn't happen.
