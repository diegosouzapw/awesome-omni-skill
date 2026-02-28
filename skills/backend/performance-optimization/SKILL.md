---
name: performance-optimization
description: Performance optimization patterns for backend services and frontend apps
alwaysApply: false
applyTo:
  - "services/**/*.ts"
  - "apps/**/*.tsx"
  - "apps/**/*.ts"
---

# Performance Optimization Skill

Guide for optimizing performance in Splits Network.

## Purpose

- **Database Queries**: Prevent N+1, use indexes, optimize JOINs
- **API Performance**: Caching, pagination, query optimization
- **Frontend Performance**: Code splitting, lazy loading, memoization
- **Bundle Optimization**: Reduce bundle size, tree shaking

## When to Use

- Optimizing slow API endpoints
- Reducing database query time
- Improving page load times
- Reducing bundle size

## Core Patterns

### 1. Prevent N+1 Queries

```typescript
// ❌ WRONG - N+1 query problem
const applications = await supabase.from('applications').select('*');
for (const app of applications) {
  app.candidate = await supabase.from('candidates').select('*').eq('id', app.candidate_id).single();
  app.job = await supabase.from('jobs').select('*').eq('id', app.job_id).single();
}

// ✅ CORRECT - Single query with JOINs
const { data } = await supabase
  .from('applications')
  .select(`
    *,
    candidate:candidates(*),
    job:jobs(*)
  `);
```

### 2. Use Database Indexes

```sql
-- Add indexes for frequently queried columns
CREATE INDEX idx_applications_candidate_id ON applications(candidate_id);
CREATE INDEX idx_applications_job_id ON applications(job_id);
CREATE INDEX idx_applications_stage ON applications(stage);
CREATE INDEX idx_jobs_status ON jobs(status);
CREATE INDEX idx_jobs_company_id ON jobs(company_id);

-- Composite indexes for common query combinations
CREATE INDEX idx_applications_candidate_stage ON applications(candidate_id, stage);
```

### 3. API Response Caching

```typescript
import { createClient } from 'redis';

const redis = createClient({ url: process.env.REDIS_URL });

async function getCachedJobs(companyId: string) {
  const cacheKey = `jobs:company:${companyId}`;
  
  // Try cache first
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);
  
  // Fetch from database
  const jobs = await repository.listByCompany(companyId);
  
  // Cache for 5 minutes
  await redis.setex(cacheKey, 300, JSON.stringify(jobs));
  
  return jobs;
}
```

### 4. Pagination

```typescript
// Always paginate large lists
async list(filters: JobFilters, page = 1, limit = 25) {
  const offset = (page - 1) * limit;
  
  const query = this.supabase
    .from('jobs')
    .select('*', { count: 'exact' })
    .range(offset, offset + limit - 1);
    
  const { data, count } = await query;
  
  return {
    data,
    pagination: {
      total: count,
      page,
      limit,
      total_pages: Math.ceil(count / limit)
    }
  };
}
```

### 5. Frontend Code Splitting

```typescript
// Lazy load heavy components
import { lazy, Suspense } from 'react';

const HeavyChart = lazy(() => import('./HeavyChart'));

export function Dashboard() {
  return (
    <Suspense fallback={<div>Loading chart...</div>}>
      <HeavyChart />
    </Suspense>
  );
}
```

### 6. Memoization

```typescript
import { useMemo } from 'react';

export function JobList({ jobs, filters }) {
  const filteredJobs = useMemo(() => {
    return jobs.filter(job => 
      job.status === filters.status &&
      job.location.includes(filters.location)
    );
  }, [jobs, filters.status, filters.location]);
  
  return filteredJobs.map(job => <JobCard key={job.id} job={job} />);
}
```

### 7. Bundle Size Optimization

```typescript
// Use specific imports
import { format } from 'date-fns/format'; // ✅ 5KB
import * as dateFns from 'date-fns';     // ❌ 100KB+

// Dynamic imports for routes
export default function Page() {
  return <div>Content</div>;
}

// Heavy dependencies loaded only when needed
async function handleExport() {
  const XLSX = await import('xlsx');
  XLSX.utils.book_new();
}
```

See [examples/](./examples/) and [references/](./references/).
