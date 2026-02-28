---
name: init-context
description: Analyze the codebase and generate context rules for Cursor. Use when setting up or populating .cursor/rules/ files.
---

# Cursor Code Discovery - Automated Workflow

You are tasked with analyzing this codebase and completing Cursor rules files.

## Inputs

1. Target directory (optional) - A particular directory within the codebase to focus on. If not provided, analyze the entire codebase.

## Steps

1. Analyze the Codebase (or Directory) - Examine the codebase and gather information.
2. Populate Instruction Files - Use the existing files in `.cursor/rules/`.
3. Update Glob Patterns - If a target directory is provided, adjust the `alwaysApply: true` glob patterns accordingly.
4. Review Created Files - Confirm everything is accurate (make sure everything mentioned actually exists in the codebase).
5. Validate Formatting - Ensure proper formatting is followed based on the Guidelines below.

## Key Analysis Areas

1. **Overview**: Business domain, key concepts, primary user types, integration points, key business workflows
2. **Architecture & Patterns**: Folder structure, module boundaries, layer dependencies, communication patterns, external service integrations
3. **Stack Best Practices**: Language-specific idioms, framework patterns, dependency injection patterns, error handling and validation patterns
4. **Anti-Patterns**: Logging of sensitive data, hardcoded secrets, non-parameterized SQL queries
5. **Data Models**: Core domain entities and relationships, key value objects and DTOs, data validation rules, database migration patterns
6. **Security & Configuration**: Environment variable management, secrets handling (1Password, vaults, etc.), authentication/authorization flow, API security patterns, compliance requirements
7. **Commands & Scripts**: Build scripts, deployment scripts, database migration commands, custom CLI tools

## Guidelines

- Each file MUST be **40 lines or fewer** (including frontmatter)
- Use bullet points and concise language
- Preserve critical technical details
- Include specific examples from THIS codebase
- Adjust glob patterns as needed if a target directory is provided
- Content should be specific to this repository, not generic advice
