---
name: frontend-patterns
description: Next.js 16 frontend best practices for Splits Network portal and candidate apps
alwaysApply: false
applyTo:
  - "apps/portal/src/**"
  - "apps/candidate/src/**"
  - "apps/corporate/src/**"
---

# Frontend Patterns Skill

This skill provides guidance for building consistent, performant frontend applications using Next.js 16 App Router.

## Purpose

Help developers create high-quality frontend code following Splits Network standards:
- **Progressive Loading**: Load critical data first, lazy load secondary data
- **API Integration**: Use shared-api-client with proper error handling
- **State Management**: Client-side state patterns
- **Component Architecture**: Server vs Client Components
- **UI Patterns**: DaisyUI components and Tailwind utilities

## When to Use This Skill

Use this skill when:
- Creating new pages or components
- Implementing data fetching patterns
- Building forms or interactive UI
- Optimizing page performance
- Handling errors and loading states

## Core Principles

### 1. Progressive Loading Pattern ⚡

**CRITICAL FOR PERFORMANCE**: Never block page render waiting for all data.

```typescript
// ✅ CORRECT - Load critical data first, secondary data in parallel
const [job, setJob] = useState(null);
const [loading, setLoading] = useState(true);
const [applications, setApplications] = useState([]);
const [applicationsLoading, setApplicationsLoading] = useState(true);

// Load primary data immediately
useEffect(() => {
  async function loadJob() {
    const res = await client.get(`/jobs/${id}`);
    setJob(res.data);
    setLoading(false);
  }
  loadJob();
}, [id]);

// Load secondary data after primary loads
useEffect(() => {
  async function loadApplications() {
    const res = await client.get(`/applications?job_id=${id}`);
    setApplications(res.data);
    setApplicationsLoading(false);
  }
  if (job) loadApplications();
}, [job]);

// ❌ WRONG - Loading all data in one effect blocks entire page
useEffect(() => {
  async function loadAll() {
    const job = await client.get(`/jobs/${id}`);
    const applications = await client.get(`/applications?job_id=${id}`);
    // Page is blank for 2-3 seconds
  }
  loadAll();
}, []);
```

See [examples/progressive-loading.tsx](./examples/progressive-loading.tsx) for complete implementation.

### 2. API Client Usage

Use `@splits-network/shared-api-client` for all API calls:

```typescript
import { createApiClient } from '@splits-network/shared-api-client';

const client = createApiClient();

// Shared-api-client automatically prepends /api/v2
const { data } = await client.get('/jobs'); // Calls /api/v2/jobs

// Handle errors
try {
  const { data } = await client.post('/jobs', jobData);
  setJob(data);
} catch (error) {
  setError(error.message || 'Failed to create job');
}
```

**Important**: Routes use `/api/v2` prefix automatically. Frontend calls simple paths like `/jobs`, not `/api/v2/jobs`.

See [examples/api-client-usage.tsx](./examples/api-client-usage.tsx) for patterns.

### 3. Server vs Client Components

**Default to Server Components**, use Client Components only when needed:

```typescript
// ✅ Server Component (default) - Can fetch data directly
export default async function JobPage({ params }) {
  const job = await fetch(`${process.env.API_URL}/jobs/${params.id}`);
  return <JobDetails job={job} />;
}

// ✅ Client Component - Has interactivity
'use client';
export function JobApplicationForm({ jobId }) {
  const [formData, setFormData] = useState({});
  return <form>...</form>;
}
```

**Use Client Components when you need**:
- useState, useEffect, or other hooks
- Event handlers (onClick, onChange, etc.)
- Browser APIs (localStorage, window, etc.)
- Real-time updates or subscriptions

See [references/server-vs-client-components.md](./references/server-vs-client-components.md).

### 4. Loading States

Each section should have independent loading states:

```typescript
<div>
  {loading ? (
    <div className="skeleton h-32 w-full"></div>
  ) : (
    <JobHeader job={job} />
  )}
  
  {applicationsLoading ? (
    <div className="loading loading-spinner"></div>
  ) : (
    <ApplicationsList applications={applications} />
  )}
</div>
```

See [examples/loading-states.tsx](./examples/loading-states.tsx) for patterns.

### 5. Error Handling

Show errors at section level, not page level:

```typescript
{error && (
  <div className="alert alert-error">
    <i className="fa-duotone fa-regular fa-circle-exclamation"></i>
    <span>{error}</span>
  </div>
)}

{jobError && <ErrorAlert message={jobError} />}
{applicationsError && <ErrorAlert message={applicationsError} />}
```

See [examples/error-handling.tsx](./examples/error-handling.tsx) for patterns.

## Routing Patterns

### Protected Routes

All authenticated routes go in `app/portal/` route group:

```typescript
// apps/portal/src/app/portal/jobs/[id]/page.tsx
export default function JobDetailPage({ params }) {
  // Automatically protected by Clerk middleware
}
```

**Never create duplicate route groups** - always use `portal` for protected pages.

### Dynamic Routes

```typescript
// [id]/page.tsx - Single dynamic segment
export default function DetailPage({ params }: { params: { id: string } }) {
  const { id } = params;
}

// [...slug]/page.tsx - Catch-all route
export default function CatchAllPage({ params }: { params: { slug: string[] } }) {
  const path = params.slug.join('/');
}
```

See [references/routing-patterns.md](./references/routing-patterns.md) for more.

## Form Patterns

### Use DaisyUI Fieldset Pattern

```typescript
<fieldset className="fieldset">
  <legend className="fieldset-legend">Job Title *</legend>
  <input
    type="text"
    className="input w-full"
    value={title}
    onChange={(e) => setTitle(e.target.value)}
    required
  />
</fieldset>
```

See `docs/guidance/form-controls.md` for complete form control standards.

### Form Submission

```typescript
async function handleSubmit(e: FormEvent) {
  e.preventDefault();
  setSubmitting(true);
  setError(null);
  
  try {
    const { data } = await client.post('/jobs', formData);
    router.push(`/portal/jobs/${data.id}`);
  } catch (err) {
    setError(err.message || 'Failed to create job');
  } finally {
    setSubmitting(false);
  }
}
```

See [examples/form-submission.tsx](./examples/form-submission.tsx) for complete pattern.

## List/Table Patterns

### Server-Side Filtering Required

```typescript
// ✅ CORRECT - Server-side filtering with pagination
const [params, setParams] = useState({
  page: 1,
  limit: 25,
  search: '',
  status: 'active'
});

useEffect(() => {
  async function loadJobs() {
    const query = new URLSearchParams({
      page: params.page.toString(),
      limit: params.limit.toString(),
      search: params.search,
      status: params.status
    });
    const { data, pagination } = await client.get(`/jobs?${query}`);
    setJobs(data);
    setPagination(pagination);
  }
  loadJobs();
}, [params]);

// ❌ WRONG - Client-side filtering doesn't scale
const filteredJobs = allJobs.filter(job => job.status === 'active');
```

See [examples/list-with-filtering.tsx](./examples/list-with-filtering.tsx).

### Search Debouncing

```typescript
const [searchTerm, setSearchTerm] = useState('');

useEffect(() => {
  const timer = setTimeout(() => {
    setParams({ ...params, search: searchTerm, page: 1 });
  }, 300); // 300ms debounce
  
  return () => clearTimeout(timer);
}, [searchTerm]);
```

See [examples/debounced-search.tsx](./examples/debounced-search.tsx).

## Modal/Drawer Patterns

### Lazy Load Modal Data

Only fetch data when modal opens:

```typescript
const [isOpen, setIsOpen] = useState(false);
const [modalData, setModalData] = useState(null);
const [loading, setLoading] = useState(false);

async function openModal() {
  setIsOpen(true);
  setLoading(true);
  const { data } = await client.get(`/applications/${id}`);
  setModalData(data);
  setLoading(false);
}

// ❌ WRONG - Don't load modal data upfront
useEffect(() => {
  // Loads data user might never see
  loadAllModalData();
}, []);
```

See [examples/lazy-modal.tsx](./examples/lazy-modal.tsx).

## Performance Optimization

### Image Optimization

```typescript
import Image from 'next/image';

<Image
  src="/logo.png"
  alt="Splits Network"
  width={200}
  height={50}
  priority // Above the fold images
/>
```

### Dynamic Imports

```typescript
// Lazy load heavy components
const HeavyChart = dynamic(() => import('./HeavyChart'), {
  loading: () => <div className="loading loading-spinner"></div>,
  ssr: false // Don't render on server
});
```

See [references/performance-checklist.md](./references/performance-checklist.md).

## Common Anti-Patterns to Avoid

### ❌ Monolithic Data Loading
```typescript
// WRONG - Loads everything sequentially
const job = await fetchJob();
const applications = await fetchApplications();
const candidates = await fetchCandidates();
// Page blank for 5+ seconds
```

### ❌ Over-Fetching
```typescript
// WRONG - Loading data "just in case"
useEffect(() => {
  loadAllJobs();
  loadAllCandidates();
  loadAllApplications();
  // Most of this data is never used
}, []);
```

### ❌ Client-Side Filtering at Scale
```typescript
// WRONG - Breaks with large datasets
const filtered = allJobs.filter(j => j.status === 'active');
```

### ❌ Blocking Entire Page for Secondary Data
```typescript
// WRONG - Waiting for everything before showing anything
if (!job || !applications || !candidates) {
  return <Loading />;
}
```

## References

- [Progressive Loading Example](./examples/progressive-loading.tsx)
- [API Client Usage](./examples/api-client-usage.tsx)
- [Form Patterns](./examples/form-submission.tsx)
- [Server vs Client Components](./references/server-vs-client-components.md)
- [Performance Checklist](./references/performance-checklist.md)
- [Routing Patterns](./references/routing-patterns.md)

## Related Skills

- `api-specifications` - Backend API patterns
- `error-handling` - Error handling strategies
- `performance-optimization` - Advanced performance patterns
