---
name: portal-dev
description: Coding standards and patterns for the SIO Delhi portal. Use when writing new portal features, components, API endpoints, or fixing bugs in the portal codebase. Automatically loaded for portal development tasks.
---

# Portal Development Standards

## Tech Stack
- Frontend: React 19 + TypeScript 5.9 + Vite 7 (strict mode ON)
- Styling: Custom CSS design tokens (portal-tokens.css), NOT Tailwind in portal
- Auth: Clerk (JWT RS256, phone/email login)
- Backend: PHP (plain, no framework) on cPanel with MySQL via PDO
- State: React Context + hooks (PortalAuthContext, NotificationContext)

## TypeScript Rules
- No `any` — use `unknown` and narrow with type guards
- No `@ts-ignore` — fix the type instead
- Prefer `interface` over `type` for object shapes
- All API responses typed in `src/portal/types.ts`
- Use discriminated unions for state (loading | error | success)

## React Rules
- Functional components only, custom hooks for reusable logic (useX naming)
- No prop drilling > 2 levels — use context or composition
- useEffect only for sync with external systems (API, DOM, subscriptions)
- Never store derived state — compute inline or useMemo
- Handle loading, error, empty states in every async component
- Dialog pattern: parent owns `showDialog` state, dialog has `open/onConfirm/onCancel` props

## CSS Rules (Portal)
- Use portal design tokens: `--p-red`, `--p-bg-card`, `--p-border`, `--p-cream`, `--p-amber`, `--p-text-muted`
- Class naming: `portal-{component}-{modifier}` (e.g. `portal-btn-primary`, `portal-badge-revoked`)
- Add reusable styles to `portal-components.css`, inline styles OK for one-off layout
- Responsive: test at 320px, 768px, 1024px, 1440px

## PHP Backend Rules
- PDO prepared statements for ALL queries — never concatenate user input into SQL
- JSON responses: `{ success: true, data: ... }` or `{ success: false, error: "..." }`
- Function naming: `portalVerbNoun` (e.g. `portalGetUsers`, `portalCreateUnit`, `portalRevokeUser`)
- Always check JWT + role before data access
- Return appropriate HTTP status codes (200, 201, 400, 401, 403, 404, 500)

## API Call Pattern (Frontend)
```typescript
// In api.ts — all API calls go through apiFetch with auth header
// Errors thrown as Error objects, caught in component try/catch
// Always show error to user via setSaveError or setError state
```

## Date Handling
- Storage: DDMMYYYY string (e.g. "25031999")
- Display: DD/MM/YYYY or "25 Mar 1999"
- Input: `<DateInput>` component (calendar picker, always DD/MM/YYYY)
- Never use raw `<input type="date">` — always use DateInput component

## File Naming
```
components/     PascalCase.tsx      (DateInput.tsx, StatusBadge.tsx)
pages/          PascalCase.tsx      (DashboardPage.tsx, ViewMemberPage.tsx)
hooks/          camelCase.ts        (useNotifications.ts)
context/        PascalCase.tsx      (PortalAuthContext.tsx)
__tests__/      camelCase.test.ts   (api.test.ts)
*.ts            camelCase.ts        (api.ts, types.ts, constants.ts)
*.css           kebab-case.css      (portal-components.css)
```

## Build Verification
Always run after changes:
```bash
npx tsc --noEmit     # Type check
npx vite build       # Production build
```

## Key Files
- Types: `src/portal/types.ts`
- API client: `src/portal/api.ts`
- Constants & field definitions: `src/portal/constants.ts`
- Routes: `src/portal/PortalRoutes.tsx`
- Auth context: `src/portal/context/PortalAuthContext.tsx`
- Backend routes: `api/routes/portal.php`
- API router: `api/index.php`
- Design tokens: `src/portal/portal-tokens.css`
- Component styles: `src/portal/portal-components.css`

## Role Hierarchy
```
admin > zonal_secretary > regional_president > unit_president/campus_president > member
```

## Member Statuses
`active | inactive | migrated | revoked`
- inactive: reversible, stores reasons + who set it
- revoked: reversible, hidden from unit/regional, stores reason + who revoked
- Delete: permanent removal (admin/zonal only)
