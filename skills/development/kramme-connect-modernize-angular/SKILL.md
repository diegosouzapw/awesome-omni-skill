---
name: kramme:connect:modernize-angular
description: Use this Skill when working in the Connect monorepo and needing to modernize legacy Angular components.
disable-model-invocation: false
user-invocable: true
---

# Connect - Modernize Legacy Angular Component

## Instructions

**When to use this skill:**

- You're working in the Connect monorepo
- You need to refactor legacy Angular components to modern patterns
- Component extends legacy `FormComponent` or `BaseComponent`
- Component uses `@Select` decorators for state management
- Component uses `FormNode` instead of typed `FormGroup`
- Component doesn't use `ChangeDetectionStrategy.OnPush`
- Component has manual subscription management
- Component dispatches actions directly instead of using ComponentStore

**Context:** Connect's frontend is modernizing Angular components to use NgRx ComponentStore for state management, OnPush change detection, standalone components, and proper TypeScript typing. This provides better type safety, performance, and maintainability.

### Guideline Keywords

- **ALWAYS** — Mandatory requirement, exceptions are very rare and must be explicitly approved
- **NEVER** — Strong prohibition, exceptions are very rare and must be explicitly approved
- **PREFER** — Strong recommendation, exceptions allowed with justification
- **CAN** — Optional, developer's discretion
- **NOTE** — Context, rationale, or clarification
- **EXAMPLE** — Illustrative example

Strictness hierarchy: ALWAYS/NEVER > PREFER > CAN > NOTE/EXAMPLE

---

### Reference Implementation

- **ALWAYS** refer to the Q&A components refactoring as the reference implementation:
  - `libs/connect/cms/qa/feature/src/lib/edit-topic/` - Edit topic component with form management
  - `libs/connect/cms/qa/feature/src/lib/settings-page/` - Settings page with conditional form logic
  - `libs/connect/cms/qa/feature/src/lib/topics-page/` - Topics page with complex state

---

### Migration Process

#### Phase 1: Assessment

- **ALWAYS** read all component files before starting:

  - Component TypeScript file
  - Component template
  - Component styles (if any)
  - Related store/state files

- **ALWAYS** identify patterns to migrate:

  - Legacy base class usage (`extends FormComponent`, `extends BaseComponent`)
  - `@Select` decorators for state
  - `FormNode` usage
  - Manual subscriptions (`subscribe()`, `takeUntil()`)
  - Direct action dispatching
  - Lifecycle hooks (`onInit()` vs `ngOnInit()`)

- **ALWAYS** identify business logic:
  - Form management
  - State updates
  - API calls
  - Conditional field logic
  - User interactions

---

#### Phase 2: Create ComponentStore

- **ALWAYS** create the store file in the same directory as the component (e.g., `component-name.store.ts`)
- **ALWAYS** define form controls interface separately from state
- **ALWAYS** define forms as class properties, NOT in state
- **ALWAYS** extract `initialState` as a constant
- **ALWAYS** use `readonly` for immutability
- **ALWAYS** use ECMAScript `#privateFields` for encapsulation
- **ALWAYS** use proper type narrowing in effects with filter: `(tuple): tuple is [void, DataType] => tuple[1] !== null`
- **ALWAYS** use `pipe()` directly in effects: `this.effect<Type>(pipe(...))` not `this.effect<Type>((param$) => param$.pipe(...))`

**EXAMPLE:** See `resources/examples/component-store.ts` for a complete ComponentStore reference implementation.

---

#### Phase 3: Refactor Component

- **ALWAYS** add `ChangeDetectionStrategy.OnPush`
- **ALWAYS** add `standalone: true`
- **ALWAYS** add ComponentStore to `providers` array
- **ALWAYS** use `inject()` for dependency injection
- **ALWAYS** place all `inject()` calls first in the class as readonly fields
- **ALWAYS** use ECMAScript `#privateField` syntax for private members
- **NEVER** use the `public` or `private` keywords in TypeScript
- **ALWAYS** remove base class extensions
- **ALWAYS** remove `@Select` decorators
- **ALWAYS** remove manual subscriptions
- **ALWAYS** remove `DestroyRef` and `takeUntilDestroyed` (ComponentStore handles cleanup)

**EXAMPLE:** See `resources/examples/component.ts` for a complete component reference implementation.

---

#### Phase 4: Update Template

- **ALWAYS** use native control flow (`@if`, `@for`, `@switch`) instead of `*ngIf`, `*ngFor`, `*ngSwitch`
- **ALWAYS** use the `*ngrxLet` directive or `ngrxPush` pipe to handle Observables
  - **ALWAYS** prefer the `ngrxPush` pipe over `async` for one-off async bindings
  - **PREFER** not using `*ngrxLet` or `ngrxPush` multiple times for the same Observable; instead assign it to a template variable using `@let`
- **PREFER** adding animations for conditional UI
- **ALWAYS** use form bindings with proper type checking

**EXAMPLE:** See `resources/examples/template.html` for native control flow and form binding examples.

---

#### Phase 5: UX Enhancements

##### Confirmation Modals

- **ALWAYS** add confirmation modals for destructive actions
- **ALWAYS** use MatDialog to open modals
- **ALWAYS** subscribe to `afterClosed()` and only proceed if confirmed

**EXAMPLE:**

```typescript
deleteItem(): void {
  this.#dialog
    .open(ConfirmDeleteModalComponent, {
      data: { itemName: this.data$.value?.name },
    })
    .afterClosed()
    .subscribe((confirmed) => {
      if (confirmed) {
        this.#componentStore.deleteItem();
      }
    });
}
```

##### User Feedback

- **ALWAYS** use `successMessage` in ApiAction definitions (not manual toasts in stores)
- **ALWAYS** use `CoSnackService` only for local operations (cancel, info messages)
- **NEVER** show success before API call completes

**EXAMPLE - Success Messages:**

```typescript
// ❌ WRONG - shows before API completes
readonly saveChanges = this.effect<void>(
  pipe(
    tap(() => {
      this.#store.dispatch(updateAction.start(this.form.getRawValue()));
      this.#snacks.success('Saved!'); // ← BAD
    })
  )
);

// ✅ CORRECT - shows only on actual success
export const updateAction = new ApiAction<State, Input, Output>(
  'Entity',
  'Update',
  'Feature',
  {
    showErrors: true,
    successMessage: 'Saved!', // ← GOOD
  }
);
```

##### Copy to Clipboard

- **PREFER** adding copy-to-clipboard buttons for IDs

**EXAMPLE:**

```html
<button
  matSuffix
  mat-icon-button
  matTooltip="Copy to clipboard"
  [cdkCopyToClipboard]="form.controls.id.value"
  (cdkCopyToClipboardCopied)="onIdCopied($event)"
>
  <fa-icon [icon]="copyIcon" />
</button>
```

---

#### Phase 6: Verification

- **ALWAYS** run lint: `corepack yarn nx lint <library-name>`
- **ALWAYS** check for:
  - No manual subscriptions in components
  - All effects use `pipe()` directly
  - Forms have proper type annotations
  - No `any` types
  - Proper change detection strategy
- **ALWAYS** test:
  - Form initialization
  - Save/cancel flows
  - Conditional field logic
  - Error handling
  - User feedback

---

### Common Patterns

See `resources/patterns.md` for detailed examples of conditional field disabling and nonNullable form controls.

---

### Critical Rules

#### Forms

- **NEVER** store forms in ComponentStore state
  - **ALWAYS** define forms as class properties in the store
- **ALWAYS** add `nonNullable: true` to form controls
- **ALWAYS** use typed `FormGroup` and `FormControl` (not `FormNode`)
- **ALWAYS** define form controls interface

#### Effects

- **ALWAYS** use `pipe()` directly: `this.effect<Type>(pipe(...))`
- **NEVER** use arrow functions: `this.effect<Type>((param$) => param$.pipe(...))`
- **ALWAYS** use proper type narrowing with filter

#### Subscriptions

- **NEVER** use manual subscriptions in components
  - **NOTE**: ComponentStore handles cleanup automatically
- **NEVER** use `DestroyRef` and `takeUntilDestroyed` for ComponentStore subscriptions
- **ALWAYS** wire observables directly to updaters/effects

#### User Feedback

- **NEVER** show success toasts before API calls complete
- **ALWAYS** use `successMessage` in ApiAction definitions
- **ALWAYS** use `CoSnackService` only for local operations

#### TypeScript

- **NEVER** use `any` types
  - **ALWAYS** use `unknown` when type is uncertain
- **ALWAYS** use ECMAScript `#privateField` syntax for encapsulation
- **NEVER** use the `public` or `private` keywords in TypeScript class members

#### State Management

- **NEVER** use `ComponentStore.get()`
  - **ALWAYS** read state via selectors
- **NEVER** keep empty effects

---

### Migration Checklist

See `resources/checklist.md` for the full phase-by-phase migration checklist.

---

### Additional Best Practices from AGENTS.md

- **ALWAYS** check AGENTS.md for the latest definite best practices

#### Angular Components

- **ALWAYS** set `changeDetection: ChangeDetectionStrategy.OnPush` in `@Component` decorator for new components
- **ALWAYS** use separate HTML files (do NOT use inline templates)
- **ALWAYS** place all `inject()` calls first in the class as readonly fields
- **ALWAYS** place `@Input` and `@Output` properties second in the class
- **ALWAYS** use `class` bindings instead of `ngClass`
- **ALWAYS** use `style` bindings instead of `ngStyle`
- **ALWAYS** use pipes for data transformation in templates, not methods in the component class

#### UI and Styling

- **PREFER** Angular Material/CDK for complex, interactive UI
- **NEVER** override internal APIs in Angular Material components
- **ALWAYS** use Tailwind for layout, spacing, and simple styling
- **ALWAYS** use `tw-` prefix (enforced in `libs/co/ui-tailwind-preset/tailwind.config.js`)
- **ALWAYS** define repeated patterns in CSS layer using `@apply` directive

#### FontAwesome Icons

- **ALWAYS** use FontAwesome icons via the `@fortawesome/angular-fontawesome` package
- **ALWAYS** use `<fa-icon>` component, not `<i>` tags with CSS classes
- **ALWAYS** import from `@fortawesome/pro-*-svg-icons` (not free packages)
- **ALWAYS** store icons as readonly component properties; prefer regular style by default

---

### Before Submitting Code Review

- **ALWAYS** ensure all affected tests pass locally
- **ALWAYS** run formatting: `yarn run format` (from `Connect/ng-app-monolith`)
- **ALWAYS** run linting: `yarn exec nx affected --targets=lint,test --skip-nx-cache`
- **ALWAYS** verify no linting errors are present
- **ALWAYS** ensure code follows established patterns as outlined in AGENTS.md

---

### Reference Files

**ALWAYS** refer to these files for complete examples:

- `libs/connect/cms/qa/feature/src/lib/edit-topic/cms-qa-edit-topic.store.ts`
- `libs/connect/cms/qa/feature/src/lib/settings-page/cms-qa-settings-page.store.ts`
- `libs/connect/cms/qa/feature/src/lib/topics-page/cms-qa-topics-page.store.ts`
- `AGENTS.md` - Angular Development Patterns section

### Examples

See the Instructions section.
