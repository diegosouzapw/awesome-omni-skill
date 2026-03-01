---
name: angular-v21-development
description: Develop Angular v21 components, services, and directives with signals. Use when implementing standalone components, OnPush change detection, inject() function, and input()/output() functions.
---

# Angular v21 Development

Development guide for components, services, and directives based on Angular v21 modern patterns.

## When to Use This Skill

- Creating new components, services, or directives
- Implementing signals-based state management
- Implementing Reactive Forms
- Refactoring existing code to Angular v21 patterns
- Using inject() function for dependency injection

**When NOT to use:**

- Styling only → `tailwindcss-v4-styling`
- Page routing configuration → `analogjs-development`
- UI/UX design application → `material-design-3-expressive`

## Core Principles

- **Standalone First:** All components are standalone (do NOT write `standalone: true` in decorator, it's default)
- **OnPush Detection:** Always set `changeDetection: ChangeDetectionStrategy.OnPush`
- **Signals-First:** Prefer `signal()`, `computed()`, `effect()`
- **Modern DI:** Use `inject()` function instead of constructor injection
- **Function-Based APIs:** Use `input()` / `output()` functions instead of `@Input()` / `@Output()` decorators
- **Host Bindings in Decorator:** Use `host` object instead of `@HostBinding` / `@HostListener`
- **Native Control Flow:** Use `@if` / `@for` / `@switch` instead of `*ngIf` / `*ngFor` / `*ngSwitch`
- **Class Binding:** Use `[class]` binding instead of `ngClass`
- **Style Binding:** Use `[style]` binding instead of `ngStyle`

## Implementation Guidelines

### Component Creation

Patterns to apply when creating components:

1. Set `changeDetection: ChangeDetectionStrategy.OnPush` in `@Component` decorator
2. Define inputs/outputs with `input()` / `output()` functions
3. Calculate derived state with `computed()`
4. Use `@if` / `@for` / `@switch` control flow in templates

→ Details: [Component Examples](references/component-examples.md)

### Service Creation

Patterns to apply when creating services:

1. Use `@Injectable({ providedIn: 'root' })` for singleton
2. Inject dependencies with `inject()` function
3. Manage state with `signal()`, expose with `asReadonly()`
4. Define derived state with `computed()`
5. Update state with `set()` or `update()` (do NOT use `mutate()`)

→ Details: [Signal Patterns](references/signal-patterns.md)

### Reactive Forms

Patterns to apply when implementing forms:

1. Get `FormBuilder` via `inject()`
2. Use typed forms for type safety
3. Get values with `getRawValue()`
4. Add `ReactiveFormsModule` to imports

→ Details: [Component Examples](references/component-examples.md#reactive-forms)

### Host Bindings

Host binding implementation patterns:

1. Do NOT use `@HostBinding` / `@HostListener` decorators
2. Use `host` object in `@Component` / `@Directive` decorator

→ Details: [Component Examples](references/component-examples.md#host-bindings)

### Image Optimization

Patterns to apply when displaying images:

1. Use `NgOptimizedImage` (not for inline base64 images)
2. Always specify `width` / `height` attributes
3. Add `priority` attribute for above-the-fold images

→ Details: [Component Examples](references/component-examples.md#image-optimization)

## Workflow

1. **Requirement Check:** Define component responsibility and inputs/outputs
2. **TDD Red Phase:** Create test cases first
3. **TDD Green Phase:** Minimal implementation to pass tests
4. **TDD Refactor Phase:** Optimize code
5. **Pattern Verification:**
   - Is `changeDetection: ChangeDetectionStrategy.OnPush` set?
   - Are `input()` / `output()` functions used?
   - Is DI done with `inject()` function?
   - Are signals (`signal()`, `computed()`) used?
6. **Accessibility:** Check ARIA attributes, keyboard navigation

## Related Skills

- **analogjs-development:** Use together when creating page components (*.page.ts)
- **tailwindcss-v4-styling:** When styling is needed
- **material-design-3-expressive:** When applying UI/UX design patterns

## Reference Documentation

For detailed patterns and code examples, see:

- [Signal Patterns](references/signal-patterns.md) - Detailed signal usage
- [Component Examples](references/component-examples.md) - Various component patterns
