---
name: App Development
description: Build features in the AI Coaching Platform Next.js app. Use for creating pages, components, server actions, TanStack tables, and understanding application architecture.
---

# App Development Skill

This skill provides comprehensive knowledge for developing features in the AI Coaching Platform Next.js application.

## Purpose

Use this skill when:
- Building new pages and components
- Creating server actions for data operations
- Working with the TanStack Table system
- Understanding the application architecture
- Implementing forms and data validation

## Skill Structure

```
app-development/
├── SKILL.md                # This file - main reference
├── architecture.md         # Application architecture overview
├── component-system.md     # Component patterns and primitives
├── data-flow.md           # Server actions, React Query, data handling
├── tanstack-table.md      # Table component system
└── workflows.md           # Common development workflows
```

## Quick Reference

### Tech Stack
- **Framework**: Next.js 15 with App Router
- **UI**: React 18, Tailwind CSS
- **Database**: MongoDB Atlas with Mongoose ODM
- **Authentication**: Clerk
- **State Management**: React Query, React Hooks
- **Validation**: Zod schemas

### Key Directories
- `src/app/` - Pages and layouts
- `src/app/actions/` - Server actions
- `src/components/` - Reusable components
- `src/hooks/scm/` - SCM React Query hooks (centralized)
- `src/lib/schema/mongoose-schema/` - Database models
- `src/lib/schema/zod-schema/` - Validation schemas
- `src/query/` - React Query providers

## Documentation Files

### Architecture
@.claude/skills/app-development/architecture.md
- Application structure overview
- File organization patterns
- Key directories and their purposes

### Component System
@.claude/skills/app-development/component-system.md
- Component hierarchy (core, composed, domain, features)
- Creating new components
- Styling with Tailwind CSS
- Common component patterns

### Data Flow
@.claude/skills/app-development/data-flow.md
- Server action patterns
- React Query usage
- Error handling
- Database operations

### TanStack Table
@.claude/skills/app-development/tanstack-table.md
- Table configuration
- Column definitions
- Features (sorting, filtering, pagination)
- Custom cell renderers

### Workflows
@.claude/skills/app-development/workflows.md
- Common development tasks
- Step-by-step guides
- Best practices

### Database Collections
@.claude/skills/app-development/database-collections.md
- MongoDB collection schemas
- Collection relationships
- Common query patterns

## Animation Components

All animation components should be saved in:
```
src/app/animations/
```

### Available Animations
- `DilationAnimation.tsx` - Dilation transformation with scale factor slider

### Creating New Animations
1. Create component in `src/app/animations/`
2. Follow SVG-based patterns for math visualizations
3. Include proper TypeScript types
4. Support flexible configuration via props

## Code Patterns

### Server Action Pattern
```typescript
"use server";

export async function myAction(input: MyInputType) {
  return withDbConnection(async () => {
    try {
      const validated = MyInputSchema.parse(input);
      const result = await MyModel.findOne({ ... });
      return { success: true, data: result.toJSON() };
    } catch (error) {
      return { success: false, error: handleServerError(error, "Failed") };
    }
  });
}
```

### Component Pattern
```typescript
"use client";

import React from "react";

interface MyComponentProps {
  // Props with JSDoc comments
}

export function MyComponent({ ...props }: MyComponentProps) {
  return (
    // JSX
  );
}
```

## Key Principles

1. **Type Safety** - Use proper TypeScript types, avoid `any`
2. **Validation** - Use Zod for all external data
3. **Server First** - Prefer server components and actions
4. **React Query** - Use for server state management
5. **Component Hierarchy** - Follow core → composed → domain → features

## Integration with Other Skills

- For p5.js animations → Use `create-p5-animation` skill
- For database operations → See CLAUDE.md for mongosh patterns

---

**Last Updated**: December 2024
