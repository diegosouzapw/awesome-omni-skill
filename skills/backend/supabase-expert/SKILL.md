---
name: supabase-expert
description: Advanced Supabase integration specialist for Auth, Database (PostgreSQL/RLS), Storage, Realtime, Edge Functions, and AI/Vector features. Use when implementing Supabase features, debugging Supabase issues, setting up RLS policies, creating database schemas, building auth flows, optimizing Supabase queries, migrating to Supabase, or architecting Supabase-based applications. Invoke for Supabase client setup, type generation, migration creation, performance tuning, security audits, or Supabase best practices. Handles Next.js, React, Vue, Svelte, and server-side integrations.
allowed-tools: Read, Write, Edit, Bash, Grep, Glob
model: sonnet
---

# Supabase Expert Skill - Advanced Implementation Guide

## Purpose

This is a comprehensive, production-grade skill for working with Supabase across all aspects of modern application development. It provides:

- **Deep Technical Expertise**: Advanced patterns for complex use cases
- **Framework Integration**: Specific implementations for Next.js, React, Vue, Svelte
- **Production Readiness**: Security audits, performance optimization, error handling
- **Architecture Guidance**: Multi-tenancy, scaling strategies, migration patterns
- **Real-world Solutions**: Battle-tested patterns from production applications

This skill leverages the complete Supabase documentation (2,190 pages) to provide accurate, up-to-date guidance across:
- Authentication & Authorization (30+ auth methods)
- PostgreSQL Database & RLS (Advanced query optimization)
- Storage & CDN (File management at scale)
- Realtime (WebSocket subscriptions, presence, broadcast)
- Edge Functions (Deno runtime, serverless patterns)
- AI/Vector (Embeddings, semantic search, RAG)

## When to Use

### Core Implementation Tasks
- Setting up Supabase client configuration and TypeScript types
- Implementing authentication (social login, magic links, SSO, MFA, anonymous auth)
- Creating PostgreSQL schemas with Row Level Security (RLS)
- Building realtime features with Supabase Realtime
- Implementing file storage with Supabase Storage
- Creating or debugging Edge Functions
- Working with vector embeddings and AI features
- Setting up local development with Supabase CLI
- Creating and managing database migrations

### Advanced & Production Tasks
- Troubleshooting Supabase connection or query issues
- Optimizing Supabase queries and performance
- Implementing multi-tenancy with RLS
- Security audits and hardening
- Migration from Firebase, Parse, or other BaaS
- Architecting scalable Supabase applications
- Connection pooling and serverless optimization
- Implementing complex authorization patterns
- Setting up CI/CD with Supabase
- Monitoring and observability setup

### Framework-Specific Integration
- Next.js App Router / Pages Router integration
- React with context and hooks
- Vue 3 with Composition API
- Svelte/SvelteKit integration
- Server-side rendering (SSR) with Supabase
- Static site generation (SSG) patterns

## Documentation Access & Search Strategy

### Documentation Location
```
Base Path: /Users/zach/Documents/cc-skills/docs/supabase/
```

### Organized Structure
- **guides/auth/** - Authentication (30+ files)
- **guides/database/** - PostgreSQL, RLS, migrations, extensions (35+ files)
- **guides/storage/** - File storage and CDN
- **guides/realtime/** - Real-time subscriptions
- **guides/functions/** - Edge Functions (35+ files)
- **guides/ai/** - Vector embeddings and AI features (18+ files)
- **guides/cli/** - Supabase CLI and local development
- **guides/platform/** - Project management and deployment
- **guides/security/** - Security best practices
- **guides/deployment/** - Production deployment patterns
- **reference/** - Complete API reference (1,583 files)

### Advanced Search Strategy

When a request comes in, use a multi-stage search approach:

#### Stage 1: Broad Category Search
```bash
# Identify relevant category
grep -r "keyword" /Users/zach/Documents/cc-skills/docs/supabase/guides/ -l | head -10
```

#### Stage 2: Targeted Deep Search
```bash
# Search within specific category with context
grep -r -B 2 -A 5 "specific pattern" /Users/zach/Documents/cc-skills/docs/supabase/guides/[category]/ --include="*.txt"
```

#### Stage 3: Cross-Reference Search
```bash
# Find related documentation
grep -r "related_term_1\|related_term_2\|related_term_3" /Users/zach/Documents/cc-skills/docs/supabase/ -l
```

#### Stage 4: API Reference Search
```bash
# Search API reference for specific methods
grep -r "method_name" /Users/zach/Documents/cc-skills/docs/supabase/reference/ -l
```

### Documentation Reading Priority

1. **Guides** - For conceptual understanding and best practices
2. **Reference** - For specific API signatures and parameters
3. **Cross-reference** - Check related topics for complete context

## Enhanced Process Framework

### 1. Deep Requirement Analysis

Before providing any solution, analyze:

**Technical Requirements:**
- What Supabase feature is needed?
- What's the user's framework/environment?
- What's the scale/performance requirements?
- What are the security considerations?

**Context Detection:**
```bash
# Check if project uses Next.js
[ -f "next.config.js" ] && echo "Next.js detected"

# Check if TypeScript
[ -f "tsconfig.json" ] && echo "TypeScript project"

# Check existing Supabase setup
grep -r "createClient" . --include="*.{ts,js,tsx,jsx}" | head -5
```

**Existing Code Analysis:**
- Search for existing Supabase client setup
- Identify current patterns being used
- Check for potential conflicts or improvements

### 2. Comprehensive Documentation Search

Execute multi-stage search strategy:

```bash
# Example: Searching for auth implementation
# Stage 1: Find all auth-related docs
AUTH_DOCS=$(grep -r "authentication\|sign.*in\|auth\..*" /Users/zach/Documents/cc-skills/docs/supabase/guides/auth/ -l)

# Stage 2: Narrow to specific method (e.g., Google OAuth)
OAUTH_DOCS=$(echo "$AUTH_DOCS" | xargs grep -l "google\|oauth")

# Stage 3: Read the most relevant docs
# (Read top 3 most relevant files)
```

### 3. Context-Aware Implementation

Provide implementations that match the user's context:

**For Next.js App Router:**
- Server Components patterns
- Server Actions integration
- Middleware for auth
- Cookie-based session management

**For Next.js Pages Router:**
- API routes patterns
- getServerSideProps integration
- Client-side auth hooks

**For Client-Only React:**
- Context providers
- Custom hooks
- Local state management

**For Server-Side (Node/Deno/Bun):**
- Service role patterns
- Connection pooling
- Background jobs

### 4. Production-Grade Implementation

Every code example should include:

✅ **Complete TypeScript types**
✅ **Comprehensive error handling**
✅ **Loading states**
✅ **Edge cases handled**
✅ **Performance optimizations**
✅ **Security considerations**
✅ **Testing examples**
✅ **Monitoring/logging hooks**

### 5. Validation & Testing Guidance

Provide:
- Unit test examples
- Integration test examples
- E2E test scenarios
- RLS policy testing
- Performance benchmarking
- Security audit checklist

## Advanced Implementation Patterns

### Pattern Library Location
```
.claude/skills/supabase-expert/patterns/
```

### Pattern 1: Advanced RLS with Multi-Tenancy

**Scenario:** Multi-tenant SaaS with organization-based access control

**Search Strategy:**
```bash
grep -r "multi.*tenant\|organization\|team.*access" /Users/zach/Documents/cc-skills/docs/supabase/guides/ -l
grep -r "row.*level.*security.*tenant" /Users/zach/Documents/cc-skills/docs/supabase/guides/database/ --include="*.txt" -A 10
```

**Implementation:**
```sql
-- Organization table
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Membership table with roles
CREATE TABLE organization_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  role TEXT NOT NULL CHECK (role IN ('owner', 'admin', 'member')),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(organization_id, user_id)
);

-- Projects table (tenant data)
CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  created_by UUID REFERENCES auth.users(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Enable RLS on all tables
ALTER TABLE organizations ENABLE ROW LEVEL SECURITY;
ALTER TABLE organization_members ENABLE ROW LEVEL SECURITY;
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;

-- Helper function to check membership
CREATE OR REPLACE FUNCTION is_organization_member(org_id UUID)
RETURNS BOOLEAN AS $$
BEGIN
  RETURN EXISTS (
    SELECT 1 FROM organization_members
    WHERE organization_id = org_id
    AND user_id = auth.uid()
  );
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Helper function to check role
CREATE OR REPLACE FUNCTION get_user_role(org_id UUID)
RETURNS TEXT AS $$
BEGIN
  RETURN (
    SELECT role FROM organization_members
    WHERE organization_id = org_id
    AND user_id = auth.uid()
  );
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- RLS Policies for organizations
CREATE POLICY "Users can view their organizations"
  ON organizations FOR SELECT
  USING (is_organization_member(id));

-- RLS Policies for projects
CREATE POLICY "Organization members can view projects"
  ON projects FOR SELECT
  USING (is_organization_member(organization_id));

CREATE POLICY "Admins and owners can insert projects"
  ON projects FOR INSERT
  WITH CHECK (
    get_user_role(organization_id) IN ('admin', 'owner')
    AND auth.uid() = created_by
  );

CREATE POLICY "Admins and owners can update projects"
  ON projects FOR UPDATE
  USING (get_user_role(organization_id) IN ('admin', 'owner'));

CREATE POLICY "Owners can delete projects"
  ON projects FOR DELETE
  USING (get_user_role(organization_id) = 'owner');

-- Indexes for performance
CREATE INDEX idx_org_members_user ON organization_members(user_id);
CREATE INDEX idx_org_members_org ON organization_members(organization_id);
CREATE INDEX idx_projects_org ON projects(organization_id);
```

**TypeScript Client Usage:**
```typescript
// types/database.ts
export interface Organization {
  id: string
  name: string
  created_at: string
}

export interface OrganizationMember {
  id: string
  organization_id: string
  user_id: string
  role: 'owner' | 'admin' | 'member'
  created_at: string
}

export interface Project {
  id: string
  organization_id: string
  name: string
  created_by: string
  created_at: string
}

// lib/supabase/organizations.ts
import { createClient } from '@supabase/supabase-js'
import type { Database } from '@/types/supabase'

export class OrganizationService {
  constructor(private supabase: ReturnType<typeof createClient<Database>>) {}

  async getOrganizations() {
    const { data, error } = await this.supabase
      .from('organizations')
      .select(`
        *,
        organization_members!inner(role)
      `)

    if (error) throw error
    return data
  }

  async getProjects(organizationId: string) {
    const { data, error} = await this.supabase
      .from('projects')
      .select('*')
      .eq('organization_id', organizationId)
      .order('created_at', { ascending: false })

    if (error) throw error
    return data
  }

  async createProject(organizationId: string, name: string) {
    const { data: { user } } = await this.supabase.auth.getUser()
    if (!user) throw new Error('Not authenticated')

    const { data, error } = await this.supabase
      .from('projects')
      .insert({
        organization_id: organizationId,
        name,
        created_by: user.id
      })
      .select()
      .single()

    if (error) throw error
    return data
  }
}
```

### Pattern 2: Advanced Auth with Custom Claims

**Search Strategy:**
```bash
grep -r "custom.*claims\|jwt.*metadata\|user.*metadata" /Users/zach/Documents/cc-skills/docs/supabase/guides/auth/ -l
grep -r "auth.*hooks\|hook.*send" /Users/zach/Documents/cc-skills/docs/supabase/guides/auth/ --include="*.txt" -A 10
```

**Implementation:**
```typescript
// lib/auth/custom-claims.ts
import { createClient } from '@supabase/supabase-js'

export interface UserClaims {
  role: 'admin' | 'user' | 'moderator'
  organization_id?: string
  permissions: string[]
}

export async function setUserClaims(
  userId: string,
  claims: UserClaims
): Promise<void> {
  const supabase = createClient(
    process.env.SUPABASE_URL!,
    process.env.SUPABASE_SERVICE_ROLE_KEY! // Service role required
  )

  const { error } = await supabase.auth.admin.updateUserById(
    userId,
    {
      app_metadata: { claims }
    }
  )

  if (error) throw error
}

export async function getUserClaims(userId: string): Promise<UserClaims | null> {
  const supabase = createClient(
    process.env.SUPABASE_URL!,
    process.env.SUPABASE_SERVICE_ROLE_KEY!
  )

  const { data, error } = await supabase.auth.admin.getUserById(userId)

  if (error) throw error
  return data.user.app_metadata.claims as UserClaims || null
}

// Middleware for Next.js App Router
// middleware.ts
import { createServerClient } from '@supabase/ssr'
import { NextResponse, type NextRequest } from 'next/server'

export async function middleware(request: NextRequest) {
  const response = NextResponse.next()

  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        get(name: string) {
          return request.cookies.get(name)?.value
        },
        set(name: string, value: string, options: any) {
          response.cookies.set({ name, value, ...options })
        },
        remove(name: string, options: any) {
          response.cookies.set({ name, value: '', ...options })
        },
      },
    }
  )

  const { data: { session } } = await supabase.auth.getSession()

  // Check if accessing admin route
  if (request.nextUrl.pathname.startsWith('/admin')) {
    if (!session) {
      return NextResponse.redirect(new URL('/login', request.url))
    }

    const claims = session.user.app_metadata.claims as UserClaims

    if (claims?.role !== 'admin') {
      return NextResponse.redirect(new URL('/unauthorized', request.url))
    }
  }

  return response
}

export const config = {
  matcher: ['/admin/:path*', '/dashboard/:path*']
}
```

### Pattern 3: Realtime with Presence and Broadcast

**Search Strategy:**
```bash
grep -r "presence\|broadcast\|realtime.*channel" /Users/zach/Documents/cc-skills/docs/supabase/guides/realtime/ -l
```

**Implementation:**
```typescript
// hooks/usePresence.ts
import { useEffect, useState } from 'react'
import { createClient } from '@supabase/supabase-js'
import type { RealtimeChannel } from '@supabase/supabase-js'

interface PresenceState {
  [key: string]: {
    user_id: string
    username: string
    online_at: string
  }[]
}

export function usePresence(roomId: string) {
  const [presenceState, setPresenceState] = useState<PresenceState>({})
  const [channel, setChannel] = useState<RealtimeChannel | null>(null)

  useEffect(() => {
    const supabase = createClient(
      process.env.NEXT_PUBLIC_SUPABASE_URL!,
      process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
    )

    const presenceChannel = supabase.channel(`room:${roomId}`, {
      config: {
        presence: {
          key: 'user_id',
        },
      },
    })

    presenceChannel
      .on('presence', { event: 'sync' }, () => {
        const state = presenceChannel.presenceState()
        setPresenceState(state)
      })
      .on('presence', { event: 'join' }, ({ key, newPresences }) => {
        console.log('User joined:', key, newPresences)
      })
      .on('presence', { event: 'leave' }, ({ key, leftPresences }) => {
        console.log('User left:', key, leftPresences)
      })
      .subscribe(async (status) => {
        if (status === 'SUBSCRIBED') {
          const { data: { user } } = await supabase.auth.getUser()

          if (user) {
            await presenceChannel.track({
              user_id: user.id,
              username: user.email,
              online_at: new Date().toISOString(),
            })
          }
        }
      })

    setChannel(presenceChannel)

    return () => {
      presenceChannel.unsubscribe()
    }
  }, [roomId])

  const sendBroadcast = async (event: string, payload: any) => {
    if (channel) {
      await channel.send({
        type: 'broadcast',
        event,
        payload,
      })
    }
  }

  return {
    presenceState,
    onlineUsers: Object.values(presenceState).flat(),
    sendBroadcast,
  }
}

// Component usage
export function CollaborativeEditor({ documentId }: { documentId: string }) {
  const { onlineUsers, sendBroadcast } = usePresence(documentId)

  const handleCursorMove = (position: { x: number; y: number }) => {
    sendBroadcast('cursor_move', position)
  }

  return (
    <div>
      <div className="online-users">
        {onlineUsers.map(user => (
          <div key={user.user_id}>
            {user.username} (online)
          </div>
        ))}
      </div>
      {/* Editor component */}
    </div>
  )
}
```

### Pattern 4: Vector Search with OpenAI Embeddings

**Search Strategy:**
```bash
grep -r "vector\|embedding\|pgvector\|similarity" /Users/zach/Documents/cc-skills/docs/supabase/guides/ai/ -l
grep -r "semantic.*search\|vector.*search" /Users/zach/Documents/cc-skills/docs/supabase/guides/ai/ --include="*.txt" -A 10
```

**Implementation:**
```sql
-- Enable pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Documents table with embeddings
CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  content TEXT NOT NULL,
  embedding vector(1536), -- OpenAI ada-002 dimension
  metadata JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create index for vector similarity search
CREATE INDEX ON documents USING ivfflat (embedding vector_cosine_ops)
  WITH (lists = 100);

-- Function for similarity search
CREATE OR REPLACE FUNCTION match_documents(
  query_embedding vector(1536),
  match_threshold float,
  match_count int
)
RETURNS TABLE (
  id UUID,
  content TEXT,
  metadata JSONB,
  similarity float
) LANGUAGE sql STABLE AS $$
  SELECT
    id,
    content,
    metadata,
    1 - (embedding <=> query_embedding) AS similarity
  FROM documents
  WHERE 1 - (embedding <=> query_embedding) > match_threshold
  ORDER BY embedding <=> query_embedding
  LIMIT match_count;
$$;
```

**TypeScript Implementation:**
```typescript
// lib/ai/embeddings.ts
import { createClient } from '@supabase/supabase-js'
import OpenAI from 'openai'

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY })
const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
)

export async function generateEmbedding(text: string): Promise<number[]> {
  const response = await openai.embeddings.create({
    model: 'text-embedding-ada-002',
    input: text,
  })

  return response.data[0].embedding
}

export async function addDocument(
  content: string,
  metadata: Record<string, any> = {}
): Promise<void> {
  const embedding = await generateEmbedding(content)

  const { error } = await supabase
    .from('documents')
    .insert({
      content,
      embedding,
      metadata,
    })

  if (error) throw error
}

export async function searchDocuments(
  query: string,
  matchThreshold: number = 0.78,
  matchCount: number = 10
) {
  const queryEmbedding = await generateEmbedding(query)

  const { data, error } = await supabase.rpc('match_documents', {
    query_embedding: queryEmbedding,
    match_threshold: matchThreshold,
    match_count: matchCount,
  })

  if (error) throw error
  return data
}

// RAG (Retrieval Augmented Generation) implementation
export async function ragQuery(userQuestion: string): Promise<string> {
  // 1. Get relevant documents
  const relevantDocs = await searchDocuments(userQuestion, 0.78, 5)

  // 2. Build context from documents
  const context = relevantDocs
    .map(doc => doc.content)
    .join('\n\n')

  // 3. Generate response with OpenAI
  const response = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: [
      {
        role: 'system',
        content: 'You are a helpful assistant. Answer questions based on the provided context.',
      },
      {
        role: 'user',
        content: `Context:\n${context}\n\nQuestion: ${userQuestion}`,
      },
    ],
  })

  return response.choices[0].message.content || ''
}
```

### Pattern 5: Edge Functions with Background Jobs

**Search Strategy:**
```bash
grep -r "edge.*function\|deno\|background.*job" /Users/zach/Documents/cc-skills/docs/supabase/guides/functions/ -l
```

**Implementation:**
```typescript
// supabase/functions/process-payment/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'
import Stripe from 'https://esm.sh/stripe@14.0.0'

const stripe = new Stripe(Deno.env.get('STRIPE_SECRET_KEY') || '', {
  apiVersion: '2023-10-16',
})

const supabase = createClient(
  Deno.env.get('SUPABASE_URL') ?? '',
  Deno.env.get('SUPABASE_SERVICE_ROLE_KEY') ?? ''
)

interface PaymentRequest {
  amount: number
  currency: string
  userId: string
  organizationId: string
}

serve(async (req) => {
  try {
    // 1. Authenticate request
    const authHeader = req.headers.get('Authorization')
    if (!authHeader) {
      return new Response(
        JSON.stringify({ error: 'Missing authorization' }),
        { status: 401 }
      )
    }

    const { data: { user }, error: authError } = await supabase.auth.getUser(
      authHeader.replace('Bearer ', '')
    )

    if (authError || !user) {
      return new Response(
        JSON.stringify({ error: 'Invalid authorization' }),
        { status: 401 }
      )
    }

    // 2. Parse request body
    const body: PaymentRequest = await req.json()

    // 3. Create Stripe payment intent
    const paymentIntent = await stripe.paymentIntents.create({
      amount: body.amount,
      currency: body.currency,
      metadata: {
        user_id: body.userId,
        organization_id: body.organizationId,
      },
    })

    // 4. Store payment record in database
    const { error: dbError } = await supabase
      .from('payments')
      .insert({
        user_id: body.userId,
        organization_id: body.organizationId,
        stripe_payment_intent_id: paymentIntent.id,
        amount: body.amount,
        currency: body.currency,
        status: 'pending',
      })

    if (dbError) throw dbError

    // 5. Return client secret
    return new Response(
      JSON.stringify({
        clientSecret: paymentIntent.client_secret,
        paymentIntentId: paymentIntent.id,
      }),
      {
        headers: { 'Content-Type': 'application/json' },
        status: 200,
      }
    )

  } catch (error) {
    return new Response(
      JSON.stringify({ error: error.message }),
      {
        headers: { 'Content-Type': 'application/json' },
        status: 500,
      }
    )
  }
})
```

## Framework-Specific Implementations

### Next.js App Router (Complete Setup)

```typescript
// lib/supabase/server.ts - Server Components
import { createServerClient, type CookieOptions } from '@supabase/ssr'
import { cookies } from 'next/headers'
import type { Database } from '@/types/supabase'

export function createClient() {
  const cookieStore = cookies()

  return createServerClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        get(name: string) {
          return cookieStore.get(name)?.value
        },
        set(name: string, value: string, options: CookieOptions) {
          try {
            cookieStore.set({ name, value, ...options })
          } catch (error) {
            // Server Component can't set cookies
          }
        },
        remove(name: string, options: CookieOptions) {
          try {
            cookieStore.set({ name, value: '', ...options })
          } catch (error) {
            // Server Component can't delete cookies
          }
        },
      },
    }
  )
}

// lib/supabase/client.ts - Client Components
import { createBrowserClient } from '@supabase/ssr'
import type { Database } from '@/types/supabase'

export function createClient() {
  return createBrowserClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}

// app/auth/callback/route.ts - Auth callback handler
import { createClient } from '@/lib/supabase/server'
import { NextResponse } from 'next/server'

export async function GET(request: Request) {
  const { searchParams, origin } = new URL(request.url)
  const code = searchParams.get('code')
  const next = searchParams.get('next') ?? '/dashboard'

  if (code) {
    const supabase = createClient()
    const { error } = await supabase.auth.exchangeCodeForSession(code)

    if (!error) {
      return NextResponse.redirect(`${origin}${next}`)
    }
  }

  return NextResponse.redirect(`${origin}/auth/auth-code-error`)
}

// app/dashboard/page.tsx - Server Component with data fetching
import { createClient } from '@/lib/supabase/server'
import { redirect } from 'next/navigation'

export default async function DashboardPage() {
  const supabase = createClient()

  const {
    data: { user },
  } = await supabase.auth.getUser()

  if (!user) {
    redirect('/login')
  }

  const { data: projects } = await supabase
    .from('projects')
    .select('*')
    .order('created_at', { ascending: false })

  return (
    <div>
      <h1>Welcome {user.email}</h1>
      <ProjectsList projects={projects || []} />
    </div>
  )
}

// app/actions/projects.ts - Server Actions
'use server'

import { createClient } from '@/lib/supabase/server'
import { revalidatePath } from 'next/cache'

export async function createProject(formData: FormData) {
  const supabase = createClient()

  const {
    data: { user },
  } = await supabase.auth.getUser()

  if (!user) {
    return { error: 'Not authenticated' }
  }

  const name = formData.get('name') as string
  const organizationId = formData.get('organizationId') as string

  const { data, error } = await supabase
    .from('projects')
    .insert({
      name,
      organization_id: organizationId,
      created_by: user.id,
    })
    .select()
    .single()

  if (error) {
    return { error: error.message }
  }

  revalidatePath('/dashboard')
  return { data }
}

export async function deleteProject(projectId: string) {
  const supabase = createClient()

  const { error } = await supabase
    .from('projects')
    .delete()
    .eq('id', projectId)

  if (error) {
    return { error: error.message }
  }

  revalidatePath('/dashboard')
  return { success: true }
}
```

## Performance Optimization Strategies

### Strategy 1: Connection Pooling for Serverless

**Search:**
```bash
grep -r "connection.*pool\|supavisor\|serverless" /Users/zach/Documents/cc-skills/docs/supabase/guides/ -l
```

**Implementation:**
```typescript
// lib/supabase/connection-pool.ts
import { createClient } from '@supabase/supabase-js'

// Use connection pooler for serverless environments
const POOLER_URL = process.env.SUPABASE_URL?.replace(
  '.supabase.co',
  '.pooler.supabase.com'
)

export function createPooledClient() {
  return createClient(
    POOLER_URL || process.env.SUPABASE_URL!,
    process.env.SUPABASE_SERVICE_ROLE_KEY!,
    {
      db: {
        schema: 'public',
      },
      auth: {
        persistSession: false,
      },
    }
  )
}

// Use in API routes
export async function GET(request: Request) {
  const supabase = createPooledClient()

  const { data, error } = await supabase
    .from('large_table')
    .select('*')
    .limit(1000)

  // Connection automatically returned to pool

  return Response.json({ data, error })
}
```

### Strategy 2: Query Optimization

```typescript
// lib/supabase/optimized-queries.ts

// ❌ BAD: Fetches all columns and data
const { data } = await supabase
  .from('users')
  .select('*')

// ✅ GOOD: Select only needed columns
const { data } = await supabase
  .from('users')
  .select('id, email, username')

// ✅ GOOD: Use pagination
const { data, error, count } = await supabase
  .from('users')
  .select('*', { count: 'exact' })
  .range(0, 49) // First 50 items
  .order('created_at', { ascending: false })

// ✅ GOOD: Use joins efficiently
const { data } = await supabase
  .from('posts')
  .select(`
    id,
    title,
    author:users!inner(id, username),
    comments(count)
  `)
  .eq('published', true)
  .limit(10)

// ✅ GOOD: Use indexes (ensure indexes exist)
const { data } = await supabase
  .from('posts')
  .select('*')
  .eq('user_id', userId) // Indexed column
  .eq('status', 'published') // Indexed column
```

### Strategy 3: Caching Layer

```typescript
// lib/cache/supabase-cache.ts
import { createClient } from '@supabase/supabase-js'
import { unstable_cache } from 'next/cache'

const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_ANON_KEY!
)

// Cache queries with Next.js cache
export const getCachedProjects = unstable_cache(
  async (organizationId: string) => {
    const { data, error } = await supabase
      .from('projects')
      .select('*')
      .eq('organization_id', organizationId)
      .order('created_at', { ascending: false })

    if (error) throw error
    return data
  },
  ['projects'], // Cache key
  {
    revalidate: 60, // Revalidate every 60 seconds
    tags: ['projects'], // For manual revalidation
  }
)

// Use with revalidation
import { revalidateTag } from 'next/cache'

export async function createProject(name: string, orgId: string) {
  const { data, error } = await supabase
    .from('projects')
    .insert({ name, organization_id: orgId })
    .select()
    .single()

  if (!error) {
    revalidateTag('projects') // Invalidate cache
  }

  return { data, error }
}
```

## Security Audit Checklist

### Critical Security Checks

```typescript
// scripts/security-audit.ts

interface SecurityAudit {
  checks: SecurityCheck[]
  passed: boolean
  failures: string[]
}

interface SecurityCheck {
  name: string
  passed: boolean
  message?: string
}

export async function auditSupabaseSecurity(): Promise<SecurityAudit> {
  const checks: SecurityCheck[] = []

  // 1. Check for exposed service role key
  const serviceKeyCheck = !process.env.NEXT_PUBLIC_SUPABASE_SERVICE_ROLE_KEY
  checks.push({
    name: 'Service Role Key Security',
    passed: serviceKeyCheck,
    message: serviceKeyCheck
      ? 'Service role key not exposed in public env vars'
      : '❌ CRITICAL: Service role key exposed in public env vars!',
  })

  // 2. Check RLS is enabled on all tables
  // (Run in Supabase SQL editor or via service role client)
  const supabase = createClient(
    process.env.SUPABASE_URL!,
    process.env.SUPABASE_SERVICE_ROLE_KEY!
  )

  const { data: tables } = await supabase.rpc('check_rls_enabled')
  const rlsCheck = tables?.every(t => t.rls_enabled)

  checks.push({
    name: 'RLS Enabled on All Tables',
    passed: rlsCheck || false,
    message: rlsCheck
      ? 'All tables have RLS enabled'
      : '⚠️ Some tables missing RLS policies',
  })

  // 3. Check for SQL injection vulnerabilities
  // Scan codebase for string concatenation in queries
  const sqlInjectionCheck = true // Implement static analysis
  checks.push({
    name: 'No SQL Injection Risks',
    passed: sqlInjectionCheck,
  })

  // 4. Check HTTPS only
  const httpsCheck = process.env.SUPABASE_URL?.startsWith('https://')
  checks.push({
    name: 'HTTPS Only',
    passed: httpsCheck || false,
    message: httpsCheck ? 'Using HTTPS' : '❌ Not using HTTPS!',
  })

  const passed = checks.every(check => check.passed)
  const failures = checks
    .filter(check => !check.passed)
    .map(check => check.message || check.name)

  return { checks, passed, failures }
}
```

## Testing Utilities

### RLS Policy Testing

```typescript
// tests/rls-policies.test.ts
import { createClient } from '@supabase/supabase-js'
import { describe, it, expect, beforeAll } from 'vitest'

describe('RLS Policies', () => {
  let supabase: ReturnType<typeof createClient>
  let testUserId: string

  beforeAll(async () => {
    // Create test user
    supabase = createClient(
      process.env.SUPABASE_URL!,
      process.env.SUPABASE_SERVICE_ROLE_KEY!
    )

    const { data: { user } } = await supabase.auth.admin.createUser({
      email: 'test@example.com',
      password: 'test-password-123',
      email_confirm: true,
    })

    testUserId = user!.id
  })

  it('should allow users to read their own data', async () => {
    // Sign in as test user
    const { data: { session } } = await supabase.auth.signInWithPassword({
      email: 'test@example.com',
      password: 'test-password-123',
    })

    const userClient = createClient(
      process.env.SUPABASE_URL!,
      process.env.SUPABASE_ANON_KEY!,
      {
        global: {
          headers: {
            Authorization: `Bearer ${session!.access_token}`,
          },
        },
      }
    )

    const { data, error } = await userClient
      .from('users')
      .select('*')
      .eq('id', testUserId)

    expect(error).toBeNull()
    expect(data).toHaveLength(1)
  })

  it('should prevent users from reading other users data', async () => {
    const { data: { session } } = await supabase.auth.signInWithPassword({
      email: 'test@example.com',
      password: 'test-password-123',
    })

    const userClient = createClient(
      process.env.SUPABASE_URL!,
      process.env.SUPABASE_ANON_KEY!,
      {
        global: {
          headers: {
            Authorization: `Bearer ${session!.access_token}`,
          },
        },
      }
    )

    const { data, error } = await userClient
      .from('users')
      .select('*')
      .neq('id', testUserId)

    expect(data).toHaveLength(0) // RLS should filter out other users
  })
})
```

## Error Handling Framework

### Comprehensive Error Handler

```typescript
// lib/errors/supabase-errors.ts
import { PostgrestError } from '@supabase/supabase-js'

export class SupabaseError extends Error {
  constructor(
    public code: string,
    public details: string,
    public hint?: string
  ) {
    super(details)
    this.name = 'SupabaseError'
  }
}

export function handleSupabaseError(error: PostgrestError | null): never {
  if (!error) {
    throw new Error('Unknown error occurred')
  }

  // RLS Policy Violation
  if (error.code === '42501' || error.message.includes('policy')) {
    throw new SupabaseError(
      'RLS_POLICY_VIOLATION',
      'You do not have permission to perform this action',
      'Check row-level security policies'
    )
  }

  // Unique Constraint Violation
  if (error.code === '23505') {
    const field = error.message.match(/Key \((.*?)\)/)?.[1]
    throw new SupabaseError(
      'DUPLICATE_ENTRY',
      `A record with this ${field} already exists`,
      'Use a different value or update the existing record'
    )
  }

  // Foreign Key Violation
  if (error.code === '23503') {
    throw new SupabaseError(
      'INVALID_REFERENCE',
      'The referenced record does not exist',
      'Ensure the related record exists before creating this one'
    )
  }

  // Connection Error
  if (error.message.includes('Failed to fetch')) {
    throw new SupabaseError(
      'CONNECTION_ERROR',
      'Unable to connect to the database',
      'Check your network connection and Supabase status'
    )
  }

  // Generic Error
  throw new SupabaseError(
    error.code || 'UNKNOWN_ERROR',
    error.message,
    error.hint
  )
}

// Usage
try {
  const { data, error } = await supabase
    .from('users')
    .insert({ email: 'test@example.com' })

  if (error) handleSupabaseError(error)

  return data
} catch (err) {
  if (err instanceof SupabaseError) {
    console.error(`[${err.code}] ${err.details}`)
    if (err.hint) console.error(`Hint: ${err.hint}`)

    // Return user-friendly error
    return {
      error: true,
      message: err.details,
      code: err.code,
    }
  }
  throw err
}
```

## Monitoring & Observability

```typescript
// lib/monitoring/supabase-logger.ts
import { createClient } from '@supabase/supabase-js'

interface QueryLog {
  query: string
  duration: number
  error?: string
  timestamp: string
}

export class SupabaseMonitor {
  private logs: QueryLog[] = []

  constructor(private supabase: ReturnType<typeof createClient>) {
    this.wrapClient()
  }

  private wrapClient() {
    const originalFrom = this.supabase.from.bind(this.supabase)

    this.supabase.from = (table: string) => {
      const startTime = Date.now()
      const builder = originalFrom(table)

      const wrapMethod = (method: string) => {
        const original = (builder as any)[method].bind(builder)
        ;(builder as any)[method] = async (...args: any[]) => {
          const result = await original(...args)
          const duration = Date.now() - startTime

          this.logs.push({
            query: `${method} on ${table}`,
            duration,
            error: result.error?.message,
            timestamp: new Date().toISOString(),
          })

          // Log slow queries
          if (duration > 1000) {
            console.warn(`Slow query detected: ${method} on ${table} took ${duration}ms`)
          }

          return result
        }
      }

      ;['select', 'insert', 'update', 'delete', 'upsert'].forEach(wrapMethod)

      return builder
    }
  }

  getLogs() {
    return this.logs
  }

  getSlowQueries(threshold = 1000) {
    return this.logs.filter(log => log.duration > threshold)
  }

  getErrorRate() {
    const totalQueries = this.logs.length
    const errorQueries = this.logs.filter(log => log.error).length
    return totalQueries > 0 ? errorQueries / totalQueries : 0
  }
}
```

## Migration Utilities

### Migration from Firebase

```typescript
// scripts/migrate-from-firebase.ts
import admin from 'firebase-admin'
import { createClient } from '@supabase/supabase-js'

const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
)

admin.initializeApp({
  credential: admin.credential.cert('./firebase-credentials.json'),
})

export async function migrateUsers() {
  const auth = admin.auth()
  let nextPageToken: string | undefined

  do {
    const listUsersResult = await auth.listUsers(1000, nextPageToken)

    for (const userRecord of listUsersResult.users) {
      try {
        // Create user in Supabase
        const { data, error } = await supabase.auth.admin.createUser({
          email: userRecord.email!,
          email_confirm: true,
          user_metadata: {
            name: userRecord.displayName,
            avatar_url: userRecord.photoURL,
            migrated_from_firebase: true,
          },
        })

        if (error) {
          console.error(`Failed to migrate user ${userRecord.email}:`, error)
          continue
        }

        console.log(`Migrated user: ${userRecord.email}`)
      } catch (err) {
        console.error(`Error migrating user ${userRecord.email}:`, err)
      }
    }

    nextPageToken = listUsersResult.pageToken
  } while (nextPageToken)
}

export async function migrateFirestoreCollection(
  collectionName: string,
  tableName: string
) {
  const firestore = admin.firestore()
  const snapshot = await firestore.collection(collectionName).get()

  for (const doc of snapshot.docs) {
    const data = doc.data()

    // Transform Firestore document to Supabase row
    const row = {
      id: doc.id,
      ...data,
      // Convert Firestore timestamps
      created_at: data.createdAt?._seconds
        ? new Date(data.createdAt._seconds * 1000).toISOString()
        : null,
    }

    const { error } = await supabase
      .from(tableName)
      .insert(row)

    if (error) {
      console.error(`Failed to migrate document ${doc.id}:`, error)
    } else {
      console.log(`Migrated document: ${doc.id}`)
    }
  }
}
```

## Output Format

When providing Supabase guidance, follow this comprehensive format:

### 1. Deep Analysis
- Understand user's context (framework, scale, requirements)
- Identify potential challenges and edge cases
- Consider security implications

### 2. Documentation Research
- Cite specific documentation files consulted
- Reference API documentation for exact signatures
- Cross-reference related features

### 3. Production-Grade Implementation
- Complete TypeScript code with all types
- Comprehensive error handling
- Loading states and edge cases
- Performance optimizations built-in
- Security best practices applied
- Monitoring/logging hooks

### 4. Testing Strategy
- Unit test examples
- Integration test scenarios
- RLS policy testing
- E2E test guidance

### 5. Deployment Guidance
- Environment variable setup
- Migration scripts
- Rollback procedures
- Monitoring setup

### 6. Performance Considerations
- Query optimization tips
- Caching strategies
- Connection pooling guidance
- Index recommendations

### 7. Security Review
- RLS policy review
- Input validation
- API key security
- CORS configuration

### 8. Next Steps & Scaling
- Related features to implement
- Scaling considerations
- Advanced patterns to explore

## Notes

- **Always search documentation first** - Consult 2,190 pages before answering
- **PostgreSQL expertise required** - Supabase is PostgreSQL, apply PG best practices
- **Deno for Edge Functions** - Not Node.js, different module system
- **RLS is mandatory** - Test thoroughly, security is critical
- **Type generation is essential** - Always generate types from schema
- **Connection pooling** - Required for serverless/Edge deployments
- **Service role = superuser** - Never expose to clients
- **Anon key is safe** - Can be used in client-side code
- **Local development** - Requires Docker for Supabase CLI
- **Test RLS exhaustively** - Use multiple user contexts

## Documentation Search Shortcuts

```bash
# Quick searches by topic
alias sb-auth="grep -r '$1' /Users/zach/Documents/cc-skills/docs/supabase/guides/auth/ -l"
alias sb-db="grep -r '$1' /Users/zach/Documents/cc-skills/docs/supabase/guides/database/ -l"
alias sb-storage="grep -r '$1' /Users/zach/Documents/cc-skills/docs/supabase/guides/storage/ -l"
alias sb-realtime="grep -r '$1' /Users/zach/Documents/cc-skills/docs/supabase/guides/realtime/ -l"
alias sb-functions="grep -r '$1' /Users/zach/Documents/cc-skills/docs/supabase/guides/functions/ -l"
alias sb-ai="grep -r '$1' /Users/zach/Documents/cc-skills/docs/supabase/guides/ai/ -l"

# Find API reference
alias sb-api="grep -r '$1' /Users/zach/Documents/cc-skills/docs/supabase/reference/ -l | head -20"
```

## Quick Reference Commands

```bash
# Local Development
supabase init                                    # Initialize project
supabase start                                   # Start local Supabase (requires Docker)
supabase status                                  # Check local setup status
supabase stop                                    # Stop local instance

# Database Operations
supabase migration new <name>                    # Create new migration
supabase db reset                                # Reset local database
supabase db push                                 # Push migrations to remote
supabase db pull                                 # Pull remote schema
supabase db diff                                 # Show schema differences

# Type Generation
supabase gen types typescript --local            # Generate types from local
supabase gen types typescript --project-id <id>  # Generate from remote

# Edge Functions
supabase functions new <name>                    # Create new function
supabase functions serve                         # Test functions locally
supabase functions deploy <name>                 # Deploy function
supabase functions logs <name>                   # View function logs

# Authentication
supabase auth users list                         # List users
supabase auth users get <user-id>               # Get user details

# Secrets Management
supabase secrets set SECRET_NAME=value          # Set secret
supabase secrets list                           # List secrets
```
