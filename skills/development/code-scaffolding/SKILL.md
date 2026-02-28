---
name: Code Scaffolding
description: Generate boilerplate code for new modules, services, and components
category: development
version: 1.0.0
triggers:
  - new-module-creation
  - scaffold-request
  - create-service
globs: "**/templates/**,**/generators/**"
---

# Code Scaffolding Skill

Generate boilerplate code for new modules, services, and components.

## Trigger Conditions
- New module or service creation command
- User invokes with "scaffold" or "create service" or "new module"

## Input Contract
- **Required:** Template type (service, module, component, API endpoint)
- **Required:** Name and basic configuration
- **Optional:** Technology stack preferences, feature flags

## Output Contract
- Generated files following project conventions
- Directory structure with all required files
- Configuration stubs (tests, CI, docs)

## Tool Permissions
- **Read:** Project templates, coding standards, existing patterns
- **Write:** New files in target directory
- **Execute:** Code generation tools

## Execution Steps
1. Determine template type and target location
2. Load project conventions and coding standards
3. Generate directory structure
4. Create source files with boilerplate following conventions
5. Create test file stubs
6. Create documentation stubs (README, API docs)
7. Verify generated code compiles/lints clean

## Success Criteria
- Generated code follows project coding standards
- Test stubs included for all public interfaces
- Documentation stubs included
- Code compiles/lints without errors

## Escalation Rules
- Escalate if no matching template exists for the request
- Escalate if scaffolding conflicts with existing code

## Example Invocations

**Input:** "Scaffold a new Go microservice called 'notification-service'"

**Output:** Created: cmd/notification-service/main.go, internal/notification/handler.go, internal/notification/service.go, internal/notification/repository.go, internal/notification/handler_test.go, Dockerfile, README.md. All follow clean architecture pattern with dependency injection. Health check endpoint included.
