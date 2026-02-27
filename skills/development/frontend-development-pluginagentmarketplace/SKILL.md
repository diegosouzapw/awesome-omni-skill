---
name: frontend-development
description: Frontend development - React, Vue, component architecture, state management
sasmp_version: "1.3.0"
bonded_agent: 02-frontend-development
bond_type: PRIMARY_BOND

# Skill Configuration
atomic: true
single_responsibility: frontend_implementation

# Parameter Schema
parameters:
  type: object
  required: [action]
  properties:
    action:
      type: string
      enum: [create_component, manage_state, configure_routing, optimize_build]
      description: The specific frontend action to perform
    framework:
      type: string
      enum: [react, vue, angular, svelte, nextjs, nuxt]
      default: react
    component_type:
      type: string
      enum: [presentational, container, compound, headless]
    typescript:
      type: boolean
      default: true

# Return Schema
returns:
  type: object
  properties:
    success: { type: boolean }
    code: { type: string }
    files: { type: array, items: { type: object } }
    performance_impact: { type: object }

# Retry Configuration
retry:
  max_attempts: 3
  backoff: exponential
  initial_delay_ms: 1000
  jitter: true

# Observability
logging:
  level: INFO
  events: [component_created, state_configured, build_optimized]
  metrics: [bundle_size_delta, component_complexity]
---

# Frontend Development Skill

Atomic skill for frontend development including component creation, state management, and build optimization.

## Responsibility

**Single Purpose**: Implement frontend components and configure client-side architecture

## Actions

### `create_component`
Create a new React/Vue component with proper patterns.

```typescript
// Input
{
  action: "create_component",
  framework: "react",
  component_type: "compound",
  typescript: true
}

// Output
{
  success: true,
  code: "import { memo } from 'react';\n...",
  files: [
    { path: "components/MyComponent.tsx", content: "..." },
    { path: "components/MyComponent.test.tsx", content: "..." }
  ],
  performance_impact: { bundle_size: "+2kb", render_time: "5ms" }
}
```

### `manage_state`
Configure state management solution.

### `configure_routing`
Set up client-side routing.

### `optimize_build`
Optimize bundle size and build performance.

## Validation Rules

```typescript
function validateParams(params: SkillParams): ValidationResult {
  if (!params.action) {
    return { valid: false, error: "action is required" };
  }

  if (params.action === 'create_component' && !params.component_type) {
    return { valid: false, error: "component_type required for create_component" };
  }

  return { valid: true };
}
```

## Error Handling

| Error Code | Description | Recovery |
|------------|-------------|----------|
| INVALID_FRAMEWORK | Unsupported framework | Check supported frameworks |
| A11Y_VIOLATION | Accessibility issue detected | Apply accessible pattern |
| BUNDLE_BUDGET_EXCEEDED | Component too large | Suggest code splitting |

## Logging Hooks

```json
{
  "on_invoke": "log.info('frontend-development invoked', { action, framework })",
  "on_success": "log.info('Component created', { files, bundle_impact })",
  "on_error": "log.error('Frontend skill failed', { error })"
}
```

## Unit Test Template

```typescript
import { describe, it, expect } from 'vitest';
import { frontendDevelopment } from './frontend-development';

describe('frontend-development skill', () => {
  describe('create_component', () => {
    it('should create React component with TypeScript', async () => {
      const result = await frontendDevelopment({
        action: 'create_component',
        framework: 'react',
        component_type: 'presentational',
        typescript: true
      });

      expect(result.success).toBe(true);
      expect(result.code).toContain('interface');
      expect(result.files).toHaveLength(2); // Component + test
    });

    it('should include accessibility attributes', async () => {
      const result = await frontendDevelopment({
        action: 'create_component',
        framework: 'react',
        component_type: 'presentational'
      });

      expect(result.code).toMatch(/aria-|role=/);
    });
  });

  describe('manage_state', () => {
    it('should configure zustand for simple state', async () => {
      const result = await frontendDevelopment({
        action: 'manage_state',
        framework: 'react'
      });

      expect(result.success).toBe(true);
    });
  });
});
```

## Integration

- **Bonded Agent**: 02-frontend-development
- **Upstream Skills**: fullstack-basics
- **Downstream Skills**: fullstack-testing

## Version History
| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2024-01 | Initial release |
| 2.0.0 | 2025-01 | Production-grade upgrade with React 19 patterns |
