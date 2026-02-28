---
name: jikime-platform-supabase
description: Supabase specialist covering PostgreSQL 16, pgvector, RLS, real-time subscriptions, Edge Functions, and Postgres performance optimization. Use when building full-stack apps with Supabase backend or optimizing database performance.
version: 3.0.0
license: MIT
metadata:
  author: supabase
  organization: Supabase
  date: February 2026
  abstract: Comprehensive Supabase platform guide and Postgres performance optimization reference. Covers full-stack development with Next.js (Setup, CRUD, Auth, Storage, Realtime, Edge Functions, pgvector) plus 30 performance optimization rules across 8 categories, prioritized by impact. Each rule includes incorrect vs. correct SQL examples, query plan analysis, and specific performance metrics.
tags: ["platform", "supabase", "postgresql", "realtime", "auth", "pgvector", "performance", "optimization", "indexing", "connection-pooling", "monitoring"]
triggers:
  keywords: ["supabase", "postgresql", "RLS", "realtime", "pgvector", "index", "performance", "connection pool", "vacuum", "explain analyze"]
  phases: ["run"]
  agents: ["backend"]
  languages: ["typescript", "sql"]
progressive_disclosure:
  enabled: true
  level1_tokens: ~100
  level2_tokens: ~2000
user-invocable: false
---

# Supabase Platform Guide

Comprehensive guide for Supabase + Next.js full-stack development and Postgres performance optimization.

## When to Apply

Reference this skill when:
- Building full-stack apps with Supabase backend
- Writing SQL queries or designing schemas
- Implementing indexes or query optimization
- Setting up real-time features or pgvector for AI embeddings
- Configuring authentication, storage, or Edge Functions
- Reviewing database performance issues
- Configuring connection pooling or scaling
- Working with Row-Level Security (RLS)

## Quick Reference

| Feature | Description |
|---------|-------------|
| **PostgreSQL 16** | Full SQL, JSONB |
| **pgvector** | AI embeddings, vector search |
| **RLS** | Row Level Security |
| **Realtime** | Real-time subscriptions |
| **Auth** | Authentication, JWT |
| **Storage** | File storage |

## Setup

### Next.js Client

```bash
npm install @supabase/supabase-js @supabase/ssr
```

```typescript
// lib/supabase/client.ts
import { createBrowserClient } from '@supabase/ssr';

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  );
}

// lib/supabase/server.ts
import { createServerClient } from '@supabase/ssr';
import { cookies } from 'next/headers';

export function createClient() {
  const cookieStore = cookies();

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll();
        },
        setAll(cookies) {
          cookies.forEach(({ name, value, options }) =>
            cookieStore.set(name, value, options)
          );
        },
      },
    }
  );
}
```

## Database

### Table Creation

```sql
-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Posts table
CREATE TABLE posts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  author_id UUID REFERENCES users(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  content TEXT,
  published BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Index
CREATE INDEX posts_author_idx ON posts(author_id);
```

### CRUD Operations

```typescript
const supabase = createClient();

// Create
const { data, error } = await supabase
  .from('posts')
  .insert({ title: 'Hello', content: 'World', author_id: userId })
  .select()
  .single();

// Read
const { data: posts } = await supabase
  .from('posts')
  .select('*, author:users(name, email)')
  .eq('published', true)
  .order('created_at', { ascending: false })
  .limit(10);

// Update
const { data } = await supabase
  .from('posts')
  .update({ title: 'Updated' })
  .eq('id', postId)
  .select()
  .single();

// Delete
const { error } = await supabase
  .from('posts')
  .delete()
  .eq('id', postId);
```

## Row Level Security (RLS)

```sql
-- Enable RLS
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

-- Read: Public posts are viewable by everyone
CREATE POLICY "Public posts are viewable"
  ON posts FOR SELECT
  USING (published = true);

-- Update: Users can only update their own posts
CREATE POLICY "Users can update own posts"
  ON posts FOR UPDATE
  USING (auth.uid() = author_id);

-- Delete: Users can only delete their own posts
CREATE POLICY "Users can delete own posts"
  ON posts FOR DELETE
  USING (auth.uid() = author_id);

-- Insert: Only authenticated users can create posts
CREATE POLICY "Authenticated can insert"
  ON posts FOR INSERT
  WITH CHECK (auth.uid() IS NOT NULL);
```

## pgvector (AI Embeddings)

```sql
-- Enable extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Embeddings table
CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  content TEXT NOT NULL,
  embedding VECTOR(1536), -- OpenAI embedding dimensions
  metadata JSONB DEFAULT '{}'
);

-- HNSW index (fast search)
CREATE INDEX documents_embedding_idx
  ON documents USING hnsw (embedding vector_cosine_ops);

-- Similarity search function
CREATE OR REPLACE FUNCTION search_documents(
  query_embedding VECTOR(1536),
  match_count INT DEFAULT 5
)
RETURNS TABLE (id UUID, content TEXT, similarity FLOAT)
AS $$
  SELECT id, content, 1 - (embedding <=> query_embedding) AS similarity
  FROM documents
  ORDER BY embedding <=> query_embedding
  LIMIT match_count;
$$ LANGUAGE SQL;
```

```typescript
// Vector search
const { data } = await supabase.rpc('search_documents', {
  query_embedding: embedding,
  match_count: 5
});
```

## Realtime

```typescript
// Real-time subscription
const channel = supabase
  .channel('posts-changes')
  .on(
    'postgres_changes',
    { event: '*', schema: 'public', table: 'posts' },
    (payload) => {
      console.log('Change:', payload);
    }
  )
  .subscribe();

// Filtered subscription
const channel = supabase
  .channel('user-posts')
  .on(
    'postgres_changes',
    {
      event: 'INSERT',
      schema: 'public',
      table: 'posts',
      filter: `author_id=eq.${userId}`,
    },
    (payload) => {
      console.log('New post:', payload.new);
    }
  )
  .subscribe();

// Unsubscribe
supabase.removeChannel(channel);
```

## Authentication

```typescript
// Sign in
const { data, error } = await supabase.auth.signInWithPassword({
  email: 'user@example.com',
  password: 'password123',
});

// Sign up
const { data, error } = await supabase.auth.signUp({
  email: 'user@example.com',
  password: 'password123',
});

// OAuth
const { data, error } = await supabase.auth.signInWithOAuth({
  provider: 'google',
  options: { redirectTo: `${origin}/auth/callback` },
});

// Get current user
const { data: { user } } = await supabase.auth.getUser();

// Sign out
await supabase.auth.signOut();
```

## Storage

```typescript
// Upload
const { data, error } = await supabase.storage
  .from('avatars')
  .upload(`${userId}/avatar.png`, file, {
    cacheControl: '3600',
    upsert: true,
  });

// Public URL
const { data: { publicUrl } } = supabase.storage
  .from('avatars')
  .getPublicUrl(`${userId}/avatar.png`);

// Delete
await supabase.storage
  .from('avatars')
  .remove([`${userId}/avatar.png`]);
```

## Edge Functions (Deno)

```typescript
// supabase/functions/hello/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';

serve(async (req) => {
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  );

  const { data, error } = await supabase.from('posts').select('*');

  return new Response(JSON.stringify(data), {
    headers: { 'Content-Type': 'application/json' },
  });
});
```

```bash
# Deploy
supabase functions deploy hello
```

## PostgreSQL Performance Optimization

Comprehensive performance optimization guide for Postgres, maintained by Supabase. Contains 30 rules across 8 categories in `references/`, prioritized by impact to guide automated query optimization and schema design.

### Rule Categories by Priority

| Priority | Category | Impact | Prefix | Files |
|----------|----------|--------|--------|-------|
| 1 | Query Performance | CRITICAL | `query-` | 5 |
| 2 | Connection Management | CRITICAL | `conn-` | 4 |
| 3 | Security & RLS | CRITICAL | `security-` | 3 |
| 4 | Schema Design | HIGH | `schema-` | 5 |
| 5 | Concurrency & Locking | MEDIUM-HIGH | `lock-` | 4 |
| 6 | Data Access Patterns | MEDIUM | `data-` | 4 |
| 7 | Monitoring & Diagnostics | LOW-MEDIUM | `monitor-` | 3 |
| 8 | Advanced Features | LOW | `advanced-` | 2 |

### How to Use References

Read individual rule files from `references/` for detailed explanations and SQL examples.

Each rule file contains:
- Brief explanation of why it matters
- Incorrect SQL example with explanation
- Correct SQL example with explanation
- Optional EXPLAIN output or metrics
- Additional context and references
- Supabase-specific notes (when applicable)

### Available References

**Query Performance** (`query-`):
- `references/query-missing-indexes.md` - Add indexes on WHERE and JOIN columns (100-1000x faster)
- `references/query-composite-indexes.md` - Use composite indexes for multi-column queries (5-50x faster)
- `references/query-covering-indexes.md` - Use covering indexes for index-only scans
- `references/query-index-types.md` - Choose appropriate index types (B-tree, GIN, GiST, BRIN)
- `references/query-partial-indexes.md` - Use partial indexes for filtered queries

**Connection Management** (`conn-`):
- `references/conn-pooling.md` - Use PgBouncer for connection pooling (10x concurrency)
- `references/conn-prepared-statements.md` - Disable prepared statements in transaction mode
- `references/conn-idle-timeout.md` - Configure idle connection timeout
- `references/conn-limits.md` - Set appropriate connection limits

**Security & RLS** (`security-`):
- `references/security-rls-basics.md` - RLS fundamentals and setup
- `references/security-rls-performance.md` - Optimize RLS policy performance (2-10x faster)
- `references/security-privileges.md` - Configure database privileges

**Schema Design** (`schema-`):
- `references/schema-data-types.md` - Choose appropriate data types
- `references/schema-lowercase-identifiers.md` - Use lowercase identifiers
- `references/schema-primary-keys.md` - Design effective primary keys
- `references/schema-foreign-key-indexes.md` - Index foreign key columns
- `references/schema-partitioning.md` - Table partitioning strategies

**Concurrency & Locking** (`lock-`):
- `references/lock-advisory.md` - Advisory locks for application-level locking
- `references/lock-deadlock-prevention.md` - Deadlock prevention strategies
- `references/lock-short-transactions.md` - Keep transactions short
- `references/lock-skip-locked.md` - Use SKIP LOCKED for queue patterns

**Data Access Patterns** (`data-`):
- `references/data-batch-inserts.md` - Batch insert optimization
- `references/data-n-plus-one.md` - Avoid N+1 query patterns
- `references/data-pagination.md` - Efficient pagination strategies
- `references/data-upsert.md` - Upsert patterns

**Monitoring & Diagnostics** (`monitor-`):
- `references/monitor-explain-analyze.md` - Use EXPLAIN ANALYZE for query plans
- `references/monitor-pg-stat-statements.md` - Monitor with pg_stat_statements
- `references/monitor-vacuum-analyze.md` - VACUUM and ANALYZE maintenance

**Advanced Features** (`advanced-`):
- `references/advanced-full-text-search.md` - Full-text search implementation
- `references/advanced-jsonb-indexing.md` - JSONB indexing strategies

## Best Practices

- **RLS Required**: Enable RLS on all tables
- **Indexes**: Add indexes on frequently queried columns
- **Type Generation**: `supabase gen types typescript`
- **Server Client**: Use server client in server components
- **Error Handling**: Always check for errors on every query

## External References

- https://www.postgresql.org/docs/current/
- https://supabase.com/docs
- https://wiki.postgresql.org/wiki/Performance_Optimization
- https://supabase.com/docs/guides/database/overview
- https://supabase.com/docs/guides/auth/row-level-security

---

Last Updated: 2026-02-03
Version: 3.0.0
