---
name: dev_standards_skill
description: Development standards and architecture management skill. Enforces modular design, low coupling, clean code practices, and maintains project architecture graph for quick context understanding. Language-agnostic, works with TypeScript, Python, Go, Rust, Java, and more. Use when starting development tasks, refactoring, or analyzing project structure.
---

# Dev Standards Skill - Code Quality & Architecture Management

A comprehensive skill for maintaining high code quality standards and project architecture awareness throughout development.

## When to Use

- **Starting any development task** - Apply standards from the beginning
- **Code review or refactoring** - Ensure compliance with standards
- **New conversation context** - Quickly understand project architecture
- **Adding new features** - Check impact on existing architecture
- **Debugging** - Trace relationships between components

## Core Principles

### 1. Modular & Component-Based Design

**File Length Limits:**
- Single file should not exceed 300 lines (excluding tests)
- Functions should not exceed 50 lines
- Classes should not exceed 200 lines

**When to Split:**
- Extract reusable logic into utility modules
- Separate concerns into different files
- Create shared components for common patterns
- Use composition over inheritance

**Example Structure:**
```
src/
├── features/           # Feature modules
│   └── user/
│       ├── index.ts    # Public API
│       ├── types.ts    # Type definitions
│       ├── utils.ts    # Feature-specific utilities
│       └── hooks.ts    # Feature-specific hooks
├── shared/             # Shared modules
│   ├── components/     # Reusable UI components
│   ├── utils/          # Common utilities
│   ├── hooks/          # Common hooks
│   └── types/          # Shared types
└── core/               # Core business logic
    ├── api/            # API layer
    ├── store/          # State management
    └── services/       # Business services
```

### 2. Low Coupling

**Dependency Rules:**
- Features should not depend on other features directly
- Use dependency injection for external dependencies
- Communicate through events/messages for loose coupling
- Define clear interfaces/contracts between modules

**Anti-patterns to Avoid:**
```typescript
// ❌ Bad: Direct feature dependency
import { userService } from '../user/service';

// ✅ Good: Dependency injection
interface UserService {
  getUser(id: string): Promise<User>;
}

function createOrderService(userService: UserService) {
  // ...
}
```

### 3. Clean Comments

**When to Comment:**
- Complex algorithms that need explanation
- Non-obvious business logic or edge cases
- Workarounds for external library bugs
- Performance optimizations that look unusual

**When NOT to Comment:**
```typescript
// ❌ Bad: Obvious comment
// Get user by id
function getUserById(id: string) { }

// ✅ Good: Self-documenting code
function getUserById(id: string) { }

// ✅ Good: Necessary comment
// Using setTimeout instead of requestIdleCallback due to Safari bug
// See: https://bugs.webkit.org/show_bug.cgi?id=12345
function scheduleWork(callback: () => void) {
  setTimeout(callback, 0);
}
```

**Comment Style:**
- Use JSDoc for public APIs
- Keep inline comments concise
- Explain "why", not "what"

### 4. No Auto-Generated Documentation

**Prohibited:**
- ❌ Auto-generating README after task completion
- ❌ Creating summary documents unprompted
- ❌ Writing changelog entries automatically

**Allowed:**
- ✅ Updating existing documentation when explicitly asked
- ✅ Adding JSDoc comments to public APIs
- ✅ Maintaining architecture graph (via scripts)

## Architecture Graph System

### Overview

The architecture graph maintains a machine-readable, LLM-friendly representation of your project's structure and relationships. This enables quick context understanding across conversation sessions.

### Storage Location

```
<project-root>/
└── .arch/
    ├── graph.json          # Architecture graph data
    ├── modules.json        # Module definitions
    ├── dependencies.json   # Dependency relationships
    └── metadata.json       # Project metadata
```

### Graph Format

The graph uses a simple, LLM-friendly JSON format:

```json
{
  "modules": [
    {
      "id": "user-service",
      "name": "User Service",
      "path": "src/features/user/service.ts",
      "type": "service",
      "description": "Handles user authentication and profile management",
      "exports": ["UserService", "createUserService"],
      "responsibilities": [
        "User authentication",
        "Profile CRUD operations",
        "Session management"
      ]
    }
  ],
  "dependencies": [
    {
      "from": "order-service",
      "to": "user-service",
      "type": "uses",
      "description": "Validates user before creating order"
    }
  ],
  "dataFlow": [
    {
      "from": "api-handler",
      "to": "user-service",
      "data": "UserCredentials",
      "description": "Login request flow"
    }
  ]
}
```

### Module Types

- `service` - Business logic services
- `component` - UI components
- `utility` - Helper functions
- `type` - Type definitions
- `api` - API endpoints
- `store` - State management
- `hook` - React hooks or similar
- `middleware` - Request/response processors

### Relationship Types

- `uses` - Direct dependency
- `extends` - Inheritance
- `implements` - Interface implementation
- `emits` - Event emission
- `listens` - Event subscription
- `calls` - Function invocation

## Scripts

### Initialize Architecture System

```bash
bash scripts/init_arch.sh <project-root>
```

Creates `.arch/` directory and initializes graph files.

### Scan Project

```bash
bash scripts/scan_project.sh <project-root>
```

Automatically scans project and builds initial architecture graph by:
- Detecting file structure
- Parsing imports/exports
- Identifying module types
- Detecting common patterns

### Add Module

```bash
bash scripts/add_module.sh <project-root> '<module-json>'
```

Example:
```bash
bash scripts/add_module.sh . '{
  "id": "auth-service",
  "name": "Authentication Service",
  "path": "src/core/auth/service.ts",
  "type": "service",
  "description": "Handles JWT token management",
  "exports": ["AuthService"],
  "responsibilities": ["Token generation", "Token validation"]
}'
```

### Add Dependency

```bash
bash scripts/add_dependency.sh <project-root> '<dependency-json>'
```

Example:
```bash
bash scripts/add_dependency.sh . '{
  "from": "api-middleware",
  "to": "auth-service",
  "type": "uses",
  "description": "Validates JWT tokens on protected routes"
}'
```

### Query Architecture

```bash
bash scripts/query_arch.sh <project-root> <query-type> [params]
```

Query types:
- `module <module-id>` - Get module details
- `dependencies <module-id>` - Get module dependencies
- `dependents <module-id>` - Get modules that depend on this
- `path <file-path>` - Get module by file path
- `type <module-type>` - Get all modules of type
- `search <keyword>` - Search modules by name/description

### Show Architecture

```bash
bash scripts/show_arch.sh <project-root> [format]
```

Formats:
- `summary` - High-level overview (default)
- `full` - Complete graph
- `mermaid` - Mermaid diagram syntax
- `tree` - Tree structure

### Validate Architecture

```bash
bash scripts/validate_arch.sh <project-root>
```

Checks for:
- Circular dependencies
- Missing modules
- Broken references
- Coupling violations

## Workflow Integration

### Starting a New Task

1. **Load Architecture Context:**
   ```bash
   bash scripts/show_arch.sh <project-root> summary
   ```

2. **Query Relevant Modules:**
   ```bash
   bash scripts/query_arch.sh <project-root> search "authentication"
   ```

3. **Check Dependencies:**
   ```bash
   bash scripts/query_arch.sh <project-root> dependencies auth-service
   ```

4. **Apply Standards:** Follow modular design principles

5. **Update Graph:** After creating new modules
   ```bash
   bash scripts/add_module.sh <project-root> '<module-json>'
   bash scripts/add_dependency.sh <project-root> '<dependency-json>'
   ```

### Code Review Checklist

Before completing any task, verify:

- [ ] File length < 300 lines
- [ ] Function length < 50 lines
- [ ] No direct feature-to-feature dependencies
- [ ] Comments explain "why", not "what"
- [ ] Reusable logic extracted to shared modules
- [ ] Architecture graph updated
- [ ] No circular dependencies
- [ ] Clear module responsibilities

## Auto-Apply Standards

When this skill is active, automatically:

1. **Check File Length:** Warn if file exceeds 300 lines
2. **Suggest Splits:** Recommend module extraction for long files
3. **Detect Coupling:** Identify direct feature dependencies
4. **Review Comments:** Flag obvious or unnecessary comments
5. **Update Graph:** Prompt to update architecture after changes

## Best Practices

### Naming Conventions

```typescript
// Files: kebab-case
user-service.ts
auth-middleware.ts

// Functions: camelCase, verb prefix
getUserById()
validateToken()
createOrder()

// Classes: PascalCase, noun
class UserService {}
class OrderRepository {}

// Constants: UPPER_SNAKE_CASE
const MAX_RETRY_COUNT = 3;
const API_BASE_URL = '...';

// Types/Interfaces: PascalCase
interface User {}
type OrderStatus = 'pending' | 'completed';
```

### Module Organization

```typescript
// index.ts - Public API only
export { UserService } from './service';
export type { User, UserProfile } from './types';

// types.ts - Type definitions
export interface User {
  id: string;
  name: string;
}

// service.ts - Implementation
import type { User } from './types';
import { validateUser } from './utils';

export class UserService {
  // Implementation
}

// utils.ts - Internal utilities (not exported from index)
export function validateUser(user: User): boolean {
  // Validation logic
}
```

### Dependency Management

```typescript
// ✅ Good: Interface-based dependency
interface Logger {
  log(message: string): void;
}

class UserService {
  constructor(private logger: Logger) {}
}

// ✅ Good: Factory pattern
function createUserService(deps: {
  logger: Logger;
  db: Database;
}) {
  return new UserService(deps);
}

// ❌ Bad: Direct import of concrete implementation
import { ConsoleLogger } from '../logging/console-logger';
class UserService {
  private logger = new ConsoleLogger();
}
```

## Integration with Dev Workflow Skill

This skill complements `dev_workflow_skill`:

- **dev_workflow_skill**: Manages tasks, testing, commits, deployment
- **dev_standards_skill**: Ensures code quality and maintains architecture

Use together:
1. Start task with `/dev_workflow_skill`
2. Apply standards from `dev_standards_skill` during development
3. Update architecture graph as you build
4. Complete task with workflow skill's CI/CD automation

## Quick Reference

```bash
# Initialize
bash scripts/init_arch.sh .

# Scan existing project
bash scripts/scan_project.sh .

# Show architecture
bash scripts/show_arch.sh . summary

# Query specific module
bash scripts/query_arch.sh . module user-service

# Add new module
bash scripts/add_module.sh . '<module-json>'

# Add dependency
bash scripts/add_dependency.sh . '<dependency-json>'

# Validate
bash scripts/validate_arch.sh .
```

## Language Support

**Works with any language:**
- TypeScript/JavaScript (Node.js, React, Vue, etc.)
- Python (Django, Flask, FastAPI, etc.)
- Go (standard library, frameworks)
- Rust (Cargo projects)
- Java/Kotlin (Maven, Gradle)
- C/C++ (CMake, Make)
- Ruby (Rails, Sinatra)
- PHP (Laravel, Symfony)
- And more...

**Standards are language-agnostic:**
- File length limits apply to all languages
- Modular design principles are universal
- Low coupling strategies work everywhere
- Comment guidelines adapt to language conventions

## Dependencies

**Automatic detection** - scripts will use whichever is available:
1. **jq** (preferred) - Fast JSON processor
2. **Python 3** (fallback) - Usually pre-installed
3. **Python 2** (fallback) - Legacy support

**No installation required if you have Python!**

If neither is available:
```bash
# Install jq (recommended)
brew install jq              # macOS
sudo apt-get install jq      # Ubuntu
choco install jq             # Windows

# Or install Python
# https://www.python.org/downloads/
```

## Getting Started

### Quick Setup (3 commands)

```bash
# 1. Initialize
bash scripts/init_arch.sh .

# 2. Scan project (auto-detects language/framework)
bash scripts/scan_project.sh .

# 3. View architecture
bash scripts/show_arch.sh . summary
```

### Add Your First Module

```bash
bash scripts/add_module.sh . '{
  "id": "user-service",
  "name": "User Service",
  "path": "src/features/user/service.ts",
  "type": "service",
  "description": "Handles user operations",
  "exports": ["UserService"],
  "responsibilities": ["User CRUD", "Authentication"]
}'
```

## Important Notes

- Architecture graph is maintained by scripts, not manually edited
- Graph format is optimized for LLM understanding
- Works with any programming language
- Automatically uses jq or Python (no manual setup needed)
- Run `scan_project.sh` to auto-detect project type
- Update graph immediately after creating new modules
- Use `validate_arch.sh` to catch architectural issues early
- Standards are enforced during development, not after

## References

Detailed guides in `references/` directory (loaded on demand):
- `modular-design.md` - Module organization patterns
- `low-coupling.md` - Decoupling strategies  
- `clean-comments.md` - Comment best practices
- `architecture-graph.md` - Graph system details
- `quickstart.md` - 5-minute tutorial
- `examples.md` - Real-world usage examples
