---
name: generate-instructions
description: Analyze a directory and generate consolidated Cursor rules.
---

# Generate Directory Instructions

Analyze a specific directory in the monorepo and create a consolidated rule file for Cursor.

## Inputs

1. **Target Directory** (required) - The directory path to analyze (e.g., `packages/api`, `apps/web`)

## Output

Create a single file at `.cursor/rules/{DIR_NAME}.mdc` with:

- Frontmatter: `globs: "{DIR_PATH}/**"`
- All 7 sections populated with directory-specific information

## Steps

1. **Analyze the Directory** - Examine code structure, patterns, and conventions
2. **Generate Instructions File** - Create the consolidated file with all sections
3. **Validate Content** - Ensure all mentioned code/patterns actually exist
4. **Review Formatting** - Follow the guidelines below

## Sections to Include

### 1. Overview

- What this directory/package does
- Key concepts and domain terms
- Primary consumers/users of this code
- Integration points with other parts of the monorepo

### 2. Architecture & Patterns

- Folder structure within this directory
- Module boundaries and responsibilities
- Communication patterns (events, APIs, shared state)
- External service integrations

### 3. Stack Best Practices

- Language-specific idioms used here
- Framework patterns and conventions
- Dependency injection approach
- Error handling and validation patterns

### 4. Anti-Patterns

- Patterns to avoid in this directory
- Common mistakes to watch for
- Security pitfalls specific to this code

### 5. Data Models

- Key entities and their relationships
- DTOs and value objects
- Validation rules
- Database/storage patterns

### 6. Security & Configuration

- Environment variables used
- Secrets and sensitive data handling
- Authentication/authorization patterns
- API security considerations

### 7. Commands & Scripts

- Build commands for this directory
- Test commands
- Development scripts
- Deployment commands

## Guidelines

- Each section should be **concise** (aim for 5-10 bullet points per section)
- Use bullet points, not paragraphs
- Include specific file paths and code examples from THIS directory
- Only document patterns that actually exist in the code
- Skip sections that don't apply (but include the header with "N/A" note)
- The `globs` glob should match the target directory (e.g., `packages/api/**`)

## Output Format

```markdown
---
globs: "{DIR_PATH}/**"
---

# {DIR_NAME} Instructions

## Overview
- [specific bullet points]

## Architecture & Patterns
- [specific bullet points]

## Stack Best Practices
- [specific bullet points]

## Anti-Patterns
- [specific bullet points]

## Data Models
- [specific bullet points]

## Security & Configuration
- [specific bullet points]

## Commands & Scripts
- [specific bullet points]
```
