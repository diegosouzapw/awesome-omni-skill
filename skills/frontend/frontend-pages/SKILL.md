---
name: Frontend Pages
description: Create or modify React pages using MUI components, React Router, and the HATEOAS API client.
globs:
  - "frontend/src/pages/**"
  - "frontend/src/components/**"
  - "frontend/src/App.tsx"
---

# Frontend Pages

## Technology Stack
- **React 19** with **TypeScript**
- **MUI (Material UI) v7** for all UI components
- **React Router v7** for client-side routing
- **Vite** as build tool

## Page Types

### List Pages (e.g., `PatientListPage.tsx`)
- Fetch data using `apiClient` with HATEOAS link discovery.
- Support **server-side pagination** with `TablePagination` controls.
- Support **server-side search** with a text input and search button.
- Support **sortable columns** by clicking table headers (ascending/descending toggle).
- Query parameters: `page`, `pageSize`, `search`, `sortBy`, `sortDirection`.
- Display data in MUI `Table` with `TableSortLabel` for column headers.

### Detail Pages (e.g., `PatientDetailPage.tsx`)
- Fetch single item by ID from route params.
- Display all fields using MUI `Paper`, `Typography`, `Grid`.
- Provide navigation links (edit, delete, back to list) using HATEOAS links.

### Form Pages (e.g., `PatientFormPage.tsx`)
- Support both Create and Edit modes (determined by presence of route param `:id`).
- Use MUI `TextField`, `Select`, etc. for form inputs.
- Submit via `apiClient.post()` or `apiClient.put()`.
- Navigate back to detail/list on success.

### Statistics Page (`StatisticsPage.tsx`)
- Uses **recharts** library for charts.
- Chart types: `PieChart`, `BarChart`, `LineChart`.
- Fetches data from `/api/statistics/*` endpoints.
- Each chart in its own `Paper` card with title.

### Admin Page (`AdminPage.tsx`)
- Buttons for Seed and Reset operations.
- Displays current database statistics.

## Routing
- All routes defined in `App.tsx` under a `<Layout />` wrapper.
- Layout includes navigation bar with links to all sections.
- Pattern: `/entities`, `/entities/new`, `/entities/:id`, `/entities/:id/edit`.

## API Client (`frontend/src/api/apiClient.ts`)
- Singleton `ApiClient` class.
- `discoverApi()` — fetches root links from `GET /api` (cached).
- `getLink(rel)` — looks up a link by relation name.
- `findLink(links, rel)` — finds a link in a given array.
- `get<T>`, `post<T>`, `put<T>`, `delete` — typed HTTP methods.
- **All navigation uses HATEOAS links** — never hard-code API URLs in pages.

## TypeScript Types
- Located in `frontend/src/types/`.
- Must mirror backend DTOs: `patient.ts`, `doctor.ts`, `exam.ts`, `statistics.ts`, `hateoas.ts`.

## Adding a New Page
1. Create the page component in `frontend/src/pages/`.
2. Add the route in `App.tsx`.
3. Add navigation button in `Layout.tsx` if it's a top-level section.
4. Create/update types in `frontend/src/types/` to match backend DTOs.
5. Use `apiClient` for all API calls, discovering links from HATEOAS responses.
