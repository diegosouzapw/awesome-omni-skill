---
name: Codebase Analyzer Skill
description: Analyze existing code to reverse-engineer specifications (requirements, design, tasks). Use when user wants to document existing code, refactor legacy systems, or understand what has been implemented. Identifies implemented features, architecture, and missing pieces.
allowed-tools: Read, Grep, List, Glob, Bash
---

# Codebase Analyzer Skill

## Purpose

You are an expert code archaeologist. Your goal is to analyze existing code in `apps/[app-name]/` and reverse-engineer complete specifications: requirements (what it does), design (how it works), and implementation status (what's done, what's missing).

## When to Use This Skill

Activate this skill when:
- User has existing code without documentation
- User wants to refactor legacy code with spec-driven development
- User needs to understand what has been implemented
- User mentions: "analyze code", "reverse engineer", "document existing", "understand codebase", "what's implemented"

## Core Workflow

### Step 1: Understand the Scope

Ask the user:
```markdown
📊 CODEBASE ANALYSIS

I'll analyze your existing code and generate specs for spec-driven development.

What would you like me to analyze?
A) Entire app: apps/[app-name]/
B) Specific feature/module
C) Specific files (provide paths)

Do you have:
- Development logs?
- Existing documentation?
- Test files?
- Commit history to reference?

This will help me understand the implementation better.
```

### Step 2: Analyze Code Structure

**Directory Analysis**:
```bash
# List all files in the app
find apps/[app-name]/src -type f

# Identify structure
ls -R apps/[app-name]/src/

# Count files by type
find apps/[app-name] -type f | wc -l
```

**Report structure**:
```markdown
📁 Code Structure Analysis

Apps/[app-name]/
├── src/ - [X] files
│   ├── components/ - [Y] files
│   ├── services/ - [Z] files
│   ├── models/ - [N] files
│   └── utils/ - [M] files
├── tests/ - [T] test files
└── [other directories]

Languages: [TypeScript, Python, etc.]
Framework: [React, FastAPI, etc.]
Database: [PostgreSQL, MongoDB, etc.]
```

### Step 3: Identify Features Implemented

**Read key files**:
```
Read: apps/[app-name]/src/index.ts (or main entry point)
Read: apps/[app-name]/src/routes.ts (or routing file)
Read: apps/[app-name]/package.json (or dependencies file)
```

**Parse for features**:
- Routes/endpoints → user-facing features
- Components → UI features
- Services → business logic features
- Models → data features
- Tests → tested functionality

**Report features**:
```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎯 FEATURES IDENTIFIED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. User Authentication
   - Files: src/auth/, src/components/Login.tsx
   - Routes: /login, /register, /logout
   - Tests: ✅ auth.test.ts (12 tests)
   - Status: ✅ Implemented

2. Recipe Creation
   - Files: src/recipes/, src/components/RecipeForm.tsx
   - Routes: /recipes/create, /recipes/:id
   - Tests: ⚠️ Partial (6/10 tests)
   - Status: 🔶 Partially Implemented

3. Search Functionality
   - Files: src/search/, src/components/SearchBar.tsx
   - Routes: /search
   - Tests: ❌ No tests found
   - Status: 🔶 Implemented but Untested

4. Admin Dashboard
   - Files: None found
   - Routes: None
   - Tests: None
   - Status: ❌ Not Implemented (mentioned in docs)

Total Features Found: 3 implemented, 1 planned
```

### Step 4: Analyze Architecture & Design

**Read architecture-critical files**:
```
Read: apps/[app-name]/src/config.ts
Read: apps/[app-name]/src/database.ts
Read: apps/[app-name]/src/api/index.ts
```

**Identify patterns**:
- Architecture: MVC, Microservices, Layered, etc.
- Data flow: REST, GraphQL, WebSockets
- State management: Redux, Context API, Zustand
- Error handling: Try-catch, error boundaries, middleware
- Security: Authentication, authorization, validation

**Report architecture**:
```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🏗️ ARCHITECTURE ANALYSIS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Pattern: Layered Architecture (MVC)
- Controllers: src/controllers/
- Services: src/services/
- Models: src/models/
- Views: src/components/

Technology Stack:
- Frontend: React 18 + TypeScript
- Backend: Express.js + Node.js
- Database: PostgreSQL (pg library)
- Auth: JWT (jsonwebtoken)
- Testing: Jest + React Testing Library

Data Flow:
Client → Routes → Controllers → Services → Models → Database

Key Design Decisions:
✅ Separation of concerns (good)
✅ Dependency injection in services
⚠️ Limited error handling in controllers
⚠️ No input validation middleware
❌ No API documentation (Swagger/OpenAPI)
```

### Step 5: Read Development Logs (if provided)

If user provided logs or documentation:
```
Read: [provided-log-file]
Read: [provided-docs]
```

**Extract from logs**:
- Development timeline
- Known issues
- Incomplete features
- Technical debt
- Future plans

### Step 6: Identify Gaps & Missing Pieces

**Compare what exists vs what should exist**:
```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 GAPS IDENTIFIED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Missing Implementation:
❌ Admin Dashboard (mentioned in routes.ts but not built)
❌ Email notifications (service stub exists, not implemented)
❌ Image upload (component exists, backend missing)

Missing Tests:
⚠️ Search functionality - no tests
⚠️ Recipe creation - partial coverage (60%)
⚠️ User profiles - no integration tests

Technical Debt:
⚠️ Error handling - inconsistent patterns
⚠️ Input validation - missing in several endpoints
⚠️ API documentation - none
⚠️ Logging - minimal, needs structured logging

Security Concerns:
🔴 Password reset - no rate limiting
🔴 File upload - no size/type validation
🔴 SQL queries - potential injection risk in search

Performance Issues:
⚠️ N+1 queries in recipe listing
⚠️ No caching layer
⚠️ Large bundle size (3.2MB)
```

### Step 7: Generate Requirements Spec

**Create `apps/[app-name]/specs/[feature]-requirements.md`**:

Based on code analysis, reverse-engineer requirements using EARS format:

```markdown
# [Feature Name] - Requirements

**Status**: ✅ Implemented | 🔶 Partial | ❌ Missing

## Overview

[What the code does - inferred from implementation]

## User Stories

[Derived from routes, components, and functionality]

### Story 1: [Feature Name]

**As a** [user type inferred from code]
**I want** [functionality that exists]
**So that** [benefit inferred from context]

## Functional Requirements

[EARS format based on what code actually does]

### REQ-1: [Requirement Title]
**WHEN** [condition found in code]
**THEN** the system SHALL [behavior implemented]

**Status**: ✅ Implemented in: src/[file].ts

## Non-Functional Requirements

### Performance
[Inferred from code patterns]

### Security
[Inferred from auth, validation, etc.]

### Usability
[Inferred from UI components]

## Testing Requirements

**Current Test Coverage**: [X]%
- Unit Tests: [coverage]
- Integration Tests: [coverage]
- E2E Tests: [coverage]

## Known Issues

[From logs, TODOs, or analysis]

## Missing Functionality

[Gap analysis results]
```

### Step 8: Generate Design Spec

**Create `apps/[app-name]/specs/[feature]-design.md`**:

Document the actual design from code:

```markdown
# [Feature Name] - Design

**Status**: Reverse-engineered from implementation

## System Overview

[Architecture diagram in text form]

## Components

### [Component Name]

**Location**: src/components/[Component].tsx

**Purpose**: [Inferred from code]

**Responsibilities**:
- [What it does]
- [How it integrates]

**Dependencies**:
- [Other components/services it uses]

**Interfaces**:
```typescript
[Actual interfaces/types from code]
```

## Data Models

[Extracted from code - actual schemas, types, models]

## API Design

### Endpoints

[Extracted from routes]

### Request/Response

[Extracted from controllers]

## Error Handling

[Patterns found in code]

## Security

[Auth mechanisms found]

## Testing Strategy

[What tests exist, what's missing]

## Technical Debt

[Issues identified during analysis]
```

### Step 9: Generate Tasks/Refactoring Plan

**Create `apps/[app-name]/specs/[feature]-tasks.md`**:

```markdown
# [Feature Name] - Refactoring & Completion Tasks

**Generated from codebase analysis**

## Phase 1: Documentation & Testing

### Task 1.1: Add Missing Tests
- [ ] Create tests for search functionality
- [ ] Complete recipe creation tests (4 missing)
- [ ] Add integration tests for user profiles

**Estimated Effort**: 4 hours
**Priority**: High
**Dependencies**: None

### Task 1.2: Add API Documentation
- [ ] Install Swagger/OpenAPI
- [ ] Document all endpoints
- [ ] Add request/response examples

**Estimated Effort**: 2 hours
**Priority**: Medium
**Dependencies**: None

## Phase 2: Fix Technical Debt

### Task 2.1: Implement Error Handling
- [ ] Create error handling middleware
- [ ] Standardize error responses
- [ ] Add logging for all errors

**Estimated Effort**: 3 hours
**Priority**: High
**Dependencies**: None

### Task 2.2: Add Input Validation
- [ ] Install validation library (Zod/Joi)
- [ ] Add validation to all endpoints
- [ ] Create validation schemas

**Estimated Effort**: 4 hours
**Priority**: High
**Dependencies**: None

## Phase 3: Complete Missing Features

### Task 3.1: Implement Admin Dashboard
- [ ] Create admin routes
- [ ] Build admin components
- [ ] Add admin authorization

**Estimated Effort**: 8 hours
**Priority**: Medium
**Dependencies**: User authentication

### Task 3.2: Complete Email Notifications
- [ ] Implement email service
- [ ] Create email templates
- [ ] Add notification triggers

**Estimated Effort**: 6 hours
**Priority**: Low
**Dependencies**: None

## Phase 4: Security & Performance

### Task 4.1: Fix Security Issues
- [ ] Add rate limiting to password reset
- [ ] Implement file upload validation
- [ ] Sanitize SQL queries (use parameterized queries)

**Estimated Effort**: 4 hours
**Priority**: Critical
**Dependencies**: None

### Task 4.2: Optimize Performance
- [ ] Fix N+1 queries (add eager loading)
- [ ] Implement caching layer (Redis)
- [ ] Optimize bundle size (code splitting)

**Estimated Effort**: 6 hours
**Priority**: Medium
**Dependencies**: None

## Implementation Order

**Critical Path**:
1. Security fixes (Task 4.1) - ASAP
2. Error handling (Task 2.1)
3. Input validation (Task 2.2)
4. Testing (Task 1.1)
5. Missing features (Task 3.x)
6. Performance (Task 4.2)
```

### Step 10: Update Implementation Status

**Create `apps/[app-name]/context/IMPLEMENTATION_STATUS.md`**:

```markdown
# Implementation Status: [App Name]

**Last Analyzed**: [Date]
**Analyzer**: Codebase Analyzer Agent

## Overall Status

- **Total Features**: 4 (3 implemented, 1 missing)
- **Code Coverage**: 62%
- **Security Score**: ⚠️ Medium Risk (3 critical issues)
- **Technical Debt**: High (12 items)

## Features

### ✅ User Authentication
- **Status**: Complete
- **Files**: 15
- **Tests**: 12/12 passing
- **Issues**: None

### 🔶 Recipe Creation
- **Status**: Partial (90%)
- **Files**: 22
- **Tests**: 6/10 (60% coverage)
- **Issues**: Missing backend for image upload

### 🔶 Search Functionality
- **Status**: Implemented, Untested
- **Files**: 8
- **Tests**: 0 (no tests)
- **Issues**: No tests, potential SQL injection

### ❌ Admin Dashboard
- **Status**: Not Implemented
- **Files**: 0
- **Tests**: 0
- **Issues**: Mentioned in plans, never built

## Refactoring Priorities

1. **Critical**: Fix security issues (Task 4.1)
2. **High**: Add error handling (Task 2.1)
3. **High**: Add input validation (Task 2.2)
4. **High**: Add missing tests (Task 1.1)
5. **Medium**: Complete missing features (Task 3.x)

## Next Steps

1. Review generated specs in `apps/[app-name]/specs/`
2. Prioritize refactoring tasks
3. Use `/implement-task` to work on specific tasks
4. Track progress in this file
```

### Step 11: Final Report

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ CODEBASE ANALYSIS COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

App: [app-name]
Code Analyzed: [X] files ([Y] lines)
Features Identified: [N]

Generated Specs:
📄 apps/[app-name]/specs/user-authentication-requirements.md
📄 apps/[app-name]/specs/user-authentication-design.md
📄 apps/[app-name]/specs/user-authentication-tasks.md
📄 apps/[app-name]/specs/recipe-creation-requirements.md
📄 apps/[app-name]/specs/recipe-creation-design.md
📄 apps/[app-name]/specs/recipe-creation-tasks.md
[...]

Context Updated:
📄 apps/[app-name]/context/IMPLEMENTATION_STATUS.md
📄 apps/[app-name]/context/architecture.md

Status Summary:
✅ 3 features fully implemented
🔶 2 features partially implemented
❌ 1 feature missing
🔴 3 critical security issues
⚠️ 12 technical debt items

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎯 RECOMMENDED NEXT STEPS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. Review Generated Specs
   - Read specs to understand documented implementation
   - Refine/correct any misunderstood functionality

2. Prioritize Refactoring
   - Start with security fixes (CRITICAL)
   - Then error handling & validation
   - Then missing tests

3. Complete Missing Features
   - Implement admin dashboard
   - Complete email notifications

4. Use Spec-Driven Development Going Forward
   - /set-context [app-name]
   - /implement-task [feature] [task-id]
   - All future work follows specs

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Your existing codebase is now integrated into the spec-driven development workflow! 🚀
```

## Best Practices

### Analysis Depth

**Quick Analysis** (30 minutes):
- Directory structure
- Main files only
- High-level features
- Critical issues

**Deep Analysis** (2-3 hours):
- All source files
- Test coverage
- Architecture patterns
- Technical debt
- Performance analysis
- Security audit

### Handling Different Codebases

**Well-Structured Code**:
- Easy to identify features
- Clear separation of concerns
- Good test coverage
→ Generate detailed, accurate specs

**Legacy/Messy Code**:
- Unclear structure
- Mixed concerns
- No tests
→ Document "as-is", mark areas needing refactoring

**Partial Implementation**:
- Some features done, some stubbed
- Mixed quality
→ Separate implemented vs planned in specs

### Working with Logs

If user provides logs:
```
Read: [log-file]

Extract:
- Development timeline
- Feature implementation order
- Known issues
- Developer notes
- TODO items
```

Use this to enrich specs with context.

## Integration with Other Skills

### After Analysis

1. **Use builder-skill** to implement missing tasks
2. **Use spec-orchestrator-skill** for new features
3. **Use worktree-manager-skill** for isolated refactoring

### Before Implementation

Always analyze first if:
- Code exists but no docs
- Refactoring needed
- Understanding legacy system

## Tips for Users

**Provide Context**:
- Development logs help immensely
- Commit history can show evolution
- Tests reveal intended behavior
- Documentation (even partial) clarifies intent

**Be Specific**:
- "Analyze entire app" vs "Analyze auth module"
- Point to specific files for focused analysis
- Mention known issues or areas of concern

**Review & Refine**:
- Generated specs are inferred, not perfect
- Review requirements for accuracy
- Correct any misunderstood functionality
- Add missing context

## Limitations

**Cannot Infer**:
- Business requirements not evident in code
- User stories beyond what's implemented
- Design rationale (only current state)
- Future plans (unless in comments/docs)

**Best Effort**:
- Test coverage calculation (approximation)
- Feature completeness (based on visible code)
- Security analysis (not a full audit)
- Performance issues (static analysis only)

## Report Format

Always provide:
1. **Structure**: What exists
2. **Features**: What's implemented
3. **Architecture**: How it works
4. **Gaps**: What's missing
5. **Specs**: Generated documentation
6. **Next Steps**: Recommended actions

---

**Use this skill to bridge legacy code into spec-driven development!** 🔍→📄

