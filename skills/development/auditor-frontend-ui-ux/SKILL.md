---
name: auditor-frontend-ui-ux
description: "Audit frontend code quality, UI/UX, forms, state management, and translations. Typically loaded by the audit-orchestrator skill via sub-agents, but can be used standalone."
---

# Frontend UI/UX Checklist

## Verification Patterns (Stub Detection)

Before checking code quality, verify implementations are real — not stubs or placeholders.

**Principle: Existence ≠ Implementation.** A file existing does not mean the feature works.

### 4-Level Verification

| Level | Check | How |
|-------|-------|-----|
| **Exists** | File is at expected path, exports component/function | File + export check |
| **Substantive** | Returns real JSX, not placeholder div | Check for meaningful content, state usage, event handlers with logic |
| **Wired** | Connected to the app — imported and used somewhere | grep for imports, route registration |
| **Functional** | Renders correctly, interactions work | Build passes, tests pass, visual verification |

### Red Flags (auto-fail)

- Components returning `<div>Placeholder</div>`, `<div>{/* TODO */}</div>`, or `null`
- Event handlers that are empty: `onClick={() => {}}`, `onSubmit={(e) => e.preventDefault()}` with no other logic
- Hooks returning hardcoded values: `return { user: null, login: () => {} }`
- State that exists but is never rendered in JSX
- Fetch/API calls that exist but whose responses are ignored
- Components with only static content where dynamic content is expected
- `console.log`-only handlers

### Wiring Checks

- **Component → API:** Component has fetch/axios/useQuery call AND uses the response in render
- **Form → Handler:** onSubmit calls API/mutation AND handles success/error, not just `preventDefault()`
- **State → Render:** State variables appear in JSX (`.map()`, interpolation), not just defined
- **Route → Component:** Component is registered in router AND renders at expected path
- **Hook → Consumer:** Custom hook is imported and called somewhere, return values are consumed

---

## Code Quality

### Pattern Consistency
- [ ] Component structure matches existing (folder layout, file naming)
- [ ] Naming conventions match codebase (components, hooks, utils)
- [ ] Import ordering and grouping consistent
- [ ] Export style matches (default vs named, barrel files)
- [ ] Hook patterns match existing (custom hooks location, naming)
- [ ] Event handler naming consistent (handleX, onX)
- [ ] State management approach matches (context, stores, local)
- [ ] API call patterns match (hooks, services, inline)
- [ ] Styling approach matches (CSS modules, Tailwind, styled-components)

### Clean Code
- [ ] No deep nesting (max 3 levels, extract components)
- [ ] No magic numbers/strings (use constants/theme)

### Type Safety
- [ ] TypeScript types explicit (no `any`, no implicit any)
- [ ] Props fully typed (interfaces/types)
- [ ] Return types declared for functions
- [ ] Null/undefined handled properly (optional chaining, defaults)
- [ ] No type assertions without reason (`as` casts)

---

## Visual & Layout

### Consistency
- [ ] Colors match design system
- [ ] Typography follows hierarchy
- [ ] Spacing is consistent (use design tokens)
- [ ] Icons are from the same family
- [ ] Borders/shadows follow patterns

### Layout
- [ ] Responsive at all breakpoints (mobile, tablet, desktop)
- [ ] No horizontal scroll on mobile
- [ ] Content doesn't overflow containers
- [ ] Proper alignment (left, center, right)
- [ ] Grid/flex used appropriately

### Interactions
- [ ] Hover states present and consistent
- [ ] Focus states visible (accessibility)
- [ ] Loading states for async actions
- [ ] Error states are clear
- [ ] Success feedback provided

### Usability
- [ ] Actions are discoverable
- [ ] Labels are clear and concise
- [ ] Forms have proper validation feedback
- [ ] Navigation is intuitive
- [ ] Empty states are handled

### Accessibility
- [ ] Sufficient color contrast (WCAG AA)
- [ ] Keyboard navigation works
- [ ] Screen reader friendly (ARIA labels)
- [ ] Focus order is logical
- [ ] Text is resizable

---

## Forms

### Validation
- [ ] Required fields enforced
- [ ] Input types validated (email, phone, URL)
- [ ] Length limits enforced (min/max)
- [ ] Custom validation rules applied
- [ ] Server-side validation mirrors client-side

### Error Display
- [ ] Errors shown next to relevant field
- [ ] Error messages are actionable (not just "Invalid")
- [ ] Errors clear when user corrects input
- [ ] Multiple errors shown (not just first)
- [ ] Server validation errors mapped to fields

### Submission
- [ ] Submit button disabled while submitting
- [ ] Loading indicator during submission
- [ ] Success feedback provided
- [ ] Error feedback on failure
- [ ] No double submission possible

### Form State
- [ ] Dirty state tracked (unsaved changes warning)
- [ ] Form resets properly after submit
- [ ] Default values populated correctly
- [ ] Edit mode loads existing data

### Form UX
- [ ] Tab order is logical
- [ ] Enter key submits form (where appropriate)
- [ ] Autofocus on first field
- [ ] Inline validation (on blur, not just submit)

### Form Accessibility
- [ ] Labels linked to inputs (htmlFor/id)
- [ ] Error messages announced to screen readers
- [ ] Required fields marked (aria-required)
- [ ] Invalid fields marked (aria-invalid)
- [ ] Focus moves to first error on submit failure

---

## State Management

### State Location
- [ ] State lives at the right level (not too high, not too low)
- [ ] Server state vs client state separated
- [ ] URL is source of truth for navigation state
- [ ] Form state is local unless needed elsewhere
- [ ] No duplicate state (derived data stored separately)

### Data Flow
- [ ] No prop drilling beyond 2-3 levels
- [ ] Context used for truly global state only
- [ ] Data flows one direction (parent → child)
- [ ] Side effects are explicit and traceable

### Performance
- [ ] No unnecessary re-renders from state changes
- [ ] Large lists are virtualized or paginated
- [ ] Expensive computations are memoized
- [ ] Selectors used for derived state

### Data Fetching
- [ ] Loading states handled
- [ ] Error states handled
- [ ] Stale data strategy defined (refetch, cache)
- [ ] Race conditions prevented (abort, latest-only)
- [ ] Optimistic updates rollback on failure

### Normalization & Persistence
- [ ] Relational data is normalized (no nested duplicates)
- [ ] Entities stored by ID in lookup objects
- [ ] Persisted state handles schema changes
- [ ] Sensitive data not stored in localStorage
- [ ] Hydration doesn't cause flicker

---

## Translations

- [ ] All supported languages have entries for new strings
- [ ] Interpolation variables preserved (match source language exactly)
- [ ] Length works in UI (no truncation in longer languages)
- [ ] Keys follow project naming convention
