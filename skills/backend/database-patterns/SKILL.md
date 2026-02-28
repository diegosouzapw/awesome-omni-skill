---
name: database-patterns
description: Supabase database query patterns and best practices for Splits Network
alwaysApply: false
applyTo:
  - "services/**/repository.ts"
  - "services/**/src/v2/**/repository.ts"
  - "services/**/migrations/**"
---

# Database Patterns Skill

This skill provides guidance for working with the Supabase Postgres database in Splits Network services.

## Purpose

Help developers write efficient, secure database queries following Splits Network standards:
- **Access Context**: Role-based data filtering
- **Query Optimization**: JOINs, indexes, and performance
- **Migrations**: Safe schema changes
- **Transactions**: Data consistency patterns
- **Type Safety**: Proper TypeScript integration

## When to Use This Skill

Use this skill when:
- Writing repository methods
- Creating database migrations
- Optimizing query performance
- Implementing role-based data access
- Handling transactions

## Core Principles

### 1. Repository Pattern with Access Context

**All repository list methods** must use access context for role-based filtering:

```typescript
import { resolveAccessContext } from '@splits-network/shared-access-context';
import { SupabaseClient } from '@supabase/supabase-js';

export class JobRepository {
  constructor(private supabase: SupabaseClient) {}

  async list(clerkUserId: string, filters: JobFilters) {
    // Resolve user context (role, permissions, accessible entities)
    const context = await resolveAccessContext(clerkUserId, this.supabase);
    
    const query = this.supabase
      .from('jobs')
      .select('*');
      
    // Apply role-based filtering
    if (context.role === 'recruiter') {
      // Recruiters see only assigned jobs
      const assignments = await this.getAssignments(context.userId);
      query.in('id', assignments.map(a => a.job_id));
    } else if (context.isCompanyUser) {
      // Company users see their organization's jobs
      query.in('company_id', context.accessibleCompanyIds);
    }
    // Platform admins see everything (no filter)
    
    // Apply search/filter criteria
    if (filters.search) {
      query.ilike('title', `%${filters.search}%`);
    }
    if (filters.status) {
      query.eq('status', filters.status);
    }
    
    return query;
  }
}
```

See [examples/repository-with-access-context.ts](./examples/repository-with-access-context.ts) for complete pattern.

### 2. Query Building Best Practices

**Use proper SELECT patterns**:

```typescript
// ✅ CORRECT - Specific columns for large tables
const { data } = await supabase
  .from('applications')
  .select('id, candidate_id, job_id, stage, created_at');

// ✅ CORRECT - Nested selects for related data
const { data } = await supabase
  .from('applications')
  .select(`
    *,
    candidate:candidates(id, name, email),
    job:jobs(id, title, company_id)
  `);

// ⚠️ USE SPARINGLY - Select * for small lookup tables
const { data } = await supabase
  .from('job_statuses')
  .select('*'); // OK for small enum tables
```

See [examples/query-building.ts](./examples/query-building.ts) for patterns.

### 3. JOIN Patterns for Data Enrichment

All tables are in `public` schema - JOINs work seamlessly:

```typescript
// Enrich applications with candidate and job data
const { data } = await supabase
  .from('applications')
  .select(`
    *,
    candidate:candidates(
      id,
      name,
      email,
      user:users(clerk_user_id)
    ),
    job:jobs(
      id,
      title,
      status,
      company:companies(id, name)
    ),
    recruiter:recruiters(
      id,
      user:users(name, email)
    )
  `)
  .eq('id', applicationId)
  .single();
```

**Benefits**:
- One query instead of N+1 queries
- Reduced API calls
- Consistent data shape
- Better performance

See [examples/join-patterns.ts](./examples/join-patterns.ts).

### 4. Filtering and Sorting

```typescript
// Multiple filters
query
  .eq('status', 'active')
  .gte('created_at', startDate)
  .lte('created_at', endDate)
  .in('company_id', [id1, id2]);

// Text search (case-insensitive)
query.ilike('title', `%${searchTerm}%`);

// Sorting
query.order('created_at', { ascending: false });

// Pagination
query
  .range((page - 1) * limit, page * limit - 1);
```

See [examples/filtering-and-sorting.ts](./examples/filtering-and-sorting.ts).

### 5. Transaction Patterns

For multi-table operations that must succeed/fail together:

```typescript
async createPlacement(data: PlacementCreate) {
  const { data: placement, error: placementError } = await supabase
    .from('placements')
    .insert({
      application_id: data.application_id,
      recruiter_id: data.recruiter_id,
      fee_amount: data.fee_amount,
      status: 'pending'
    })
    .select()
    .single();

  if (placementError) throw placementError;

  // Update application stage
  const { error: appError } = await supabase
    .from('applications')
    .update({ stage: 'placed' })
    .eq('id', data.application_id);

  if (appError) {
    // Rollback placement creation
    await supabase.from('placements').delete().eq('id', placement.id);
    throw appError;
  }

  return placement;
}
```

See [examples/transaction-patterns.ts](./examples/transaction-patterns.ts) for patterns.

### 6. Error Handling

```typescript
async getById(id: string) {
  const { data, error } = await supabase
    .from('jobs')
    .select('*')
    .eq('id', id)
    .single();

  if (error) {
    if (error.code === 'PGRST116') {
      throw new Error('Job not found');
    }
    throw new Error(`Database error: ${error.message}`);
  }

  return data;
}
```

**Common Error Codes**:
- `PGRST116` - No rows returned (not found)
- `23505` - Unique constraint violation
- `23503` - Foreign key violation

See [references/error-codes.md](./references/error-codes.md).

## Migration Patterns

### Creating Migrations

```sql
-- migrations/001_create_jobs_table.sql
CREATE TABLE IF NOT EXISTS jobs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(255) NOT NULL,
    company_id UUID NOT NULL REFERENCES companies(id),
    status VARCHAR(50) DEFAULT 'draft',
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Indexes for common queries
CREATE INDEX idx_jobs_company_id ON jobs(company_id);
CREATE INDEX idx_jobs_status ON jobs(status);
CREATE INDEX idx_jobs_created_at ON jobs(created_at DESC);

-- Full-text search index
CREATE INDEX idx_jobs_title_search ON jobs USING gin(to_tsvector('english', title));
```

See [examples/migration-template.sql](./examples/migration-template.sql).

### Migration Best Practices

1. **Always use IF NOT EXISTS**
2. **Add indexes for foreign keys**
3. **Add indexes for commonly filtered columns**
4. **Use TIMESTAMPTZ for timestamps**
5. **Set DEFAULT values appropriately**

See [references/migration-checklist.md](./references/migration-checklist.md).

## Performance Optimization

### Use Indexes Strategically

```sql
-- Foreign keys (always index)
CREATE INDEX idx_applications_candidate_id ON applications(candidate_id);
CREATE INDEX idx_applications_job_id ON applications(job_id);

-- Commonly filtered columns
CREATE INDEX idx_jobs_status ON jobs(status);
CREATE INDEX idx_applications_stage ON applications(stage);

-- Sorting columns (DESC for newest-first)
CREATE INDEX idx_jobs_created_at ON jobs(created_at DESC);

-- Composite indexes for common filter combinations
CREATE INDEX idx_jobs_company_status ON jobs(company_id, status);

-- Full-text search
CREATE INDEX idx_jobs_title_fts ON jobs USING gin(to_tsvector('english', title));
```

See [references/indexing-strategies.md](./references/indexing-strategies.md).

### Query Optimization

```typescript
// ❌ WRONG - N+1 queries
const jobs = await supabase.from('jobs').select('*');
for (const job of jobs) {
  const company = await supabase.from('companies').select('*').eq('id', job.company_id);
}

// ✅ CORRECT - Single query with JOIN
const { data } = await supabase
  .from('jobs')
  .select(`
    *,
    company:companies(id, name, industry)
  `);
```

See [examples/query-optimization.ts](./examples/query-optimization.ts).

## Type Safety

### Generate TypeScript Types

```typescript
// Use Supabase CLI to generate types
// pnpm supabase gen types typescript --project-id einhgkqmxbkgdohwfayv > types/database.ts

import { Database } from '@/types/database';

type Job = Database['public']['Tables']['jobs']['Row'];
type JobInsert = Database['public']['Tables']['jobs']['Insert'];
type JobUpdate = Database['public']['Tables']['jobs']['Update'];
```

See [references/type-generation.md](./references/type-generation.md).

## Common Patterns

### Count Queries

```typescript
async count(filters: JobFilters): Promise<number> {
  let query = supabase
    .from('jobs')
    .select('*', { count: 'exact', head: true });
    
  if (filters.status) {
    query = query.eq('status', filters.status);
  }
  
  const { count } = await query;
  return count || 0;
}
```

### Upsert Pattern

```typescript
// Insert or update if exists
const { data } = await supabase
  .from('recruiters')
  .upsert({
    user_id: userId,
    status: 'active',
    updated_at: new Date().toISOString()
  }, {
    onConflict: 'user_id'
  })
  .select()
  .single();
```

### Soft Delete Pattern

```typescript
async delete(id: string) {
  const { data } = await supabase
    .from('jobs')
    .update({ 
      deleted_at: new Date().toISOString(),
      status: 'deleted'
    })
    .eq('id', id)
    .select()
    .single();
    
  return data;
}

// Filter out soft-deleted records
async list() {
  const { data } = await supabase
    .from('jobs')
    .select('*')
    .is('deleted_at', null); // Only non-deleted
    
  return data;
}
```

See [examples/common-patterns.ts](./examples/common-patterns.ts).

## Anti-Patterns to Avoid

### ❌ Direct Schema Queries Without Access Context

```typescript
// WRONG - No role-based filtering
async list() {
  return await supabase.from('candidates').select('*');
  // Everyone sees all candidates!
}
```

### ❌ N+1 Query Problem

```typescript
// WRONG - Multiple queries in a loop
const applications = await supabase.from('applications').select('*');
for (const app of applications) {
  const candidate = await supabase.from('candidates').select('*').eq('id', app.candidate_id);
}
```

### ❌ Missing Indexes

```sql
-- WRONG - No index on foreign key
CREATE TABLE applications (
    candidate_id UUID REFERENCES candidates(id)
    -- Missing: CREATE INDEX idx_applications_candidate_id
);
```

### ❌ Selecting Everything for Large Tables

```typescript
// WRONG - Pulls all columns for thousands of rows
const { data } = await supabase.from('applications').select('*');
```

## References

- [Repository with Access Context](./examples/repository-with-access-context.ts)
- [JOIN Patterns](./examples/join-patterns.ts)
- [Query Optimization](./examples/query-optimization.ts)
- [Migration Template](./examples/migration-template.sql)
- [Error Codes Reference](./references/error-codes.md)
- [Indexing Strategies](./references/indexing-strategies.md)
- [Migration Checklist](./references/migration-checklist.md)

## Related Skills

- `api-specifications` - API layer patterns
- `authentication-authorization` - Access control patterns
- `performance-optimization` - Advanced optimization techniques
