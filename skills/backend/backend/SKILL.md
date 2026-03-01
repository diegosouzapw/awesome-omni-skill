---
name: backend
description: "API, database, server logic, webhooks. Auto-use for any API/DB work."
version: 3.0.0
allowed-tools:
  - Read
  - Edit
  - Write
  - Bash
  - Grep
  - Glob
---

# Backend

**Auto-use when:** API, route, endpoint, database, Supabase, schema, migration, webhook, server action

**Works with:** `frontend` for UI, `security` for auth patterns

---

## Auto-Apply Rules

### 1. Every Endpoint Must Have
```typescript
export async function POST(request: Request) {
  // 1. AUTH (required)
  const { userId } = await auth()
  if (!userId) return Response.json({ error: 'Unauthorized' }, { status: 401 })

  // 2. VALIDATION (required)
  const body = await request.json()
  const result = Schema.safeParse(body)
  if (!result.success) {
    return Response.json({ error: 'Invalid', details: result.error.flatten() }, { status: 400 })
  }

  // 3. OWNERSHIP (if accessing resource)
  const resource = await db.findUnique({ where: { id: result.data.id } })
  if (resource?.userId !== userId) {
    return Response.json({ error: 'Not found' }, { status: 404 })
  }

  // 4. EXECUTE with try/catch
  try {
    const data = await db.create({ data: result.data })
    return Response.json(data, { status: 201 })
  } catch (error) {
    console.error('Error:', error)
    return Response.json({ error: 'Failed' }, { status: 500 })
  }
}
```

### 2. Server Actions
```typescript
'use server'

import { auth } from '@/lib/auth'
import { db } from '@/lib/db'
import { revalidatePath } from 'next/cache'
import { z } from 'zod'

const Schema = z.object({
  title: z.string().min(1).max(200),
})

export async function createPost(input: unknown) {
  const { userId } = await auth()
  if (!userId) return { error: 'Unauthorized' }

  const result = Schema.safeParse(input)
  if (!result.success) return { error: 'Invalid', details: result.error.flatten() }

  try {
    const post = await db.post.create({
      data: { ...result.data, authorId: userId }
    })
    revalidatePath('/posts')
    return { data: post }
  } catch (error) {
    console.error('Error:', error)
    return { error: 'Failed' }
  }
}
```

### 3. Database Patterns
```typescript
// N+1 Prevention - ALWAYS use includes
const posts = await db.post.findMany({
  include: { author: true }  // NOT separate query
})

// Pagination - ALWAYS paginate lists
const posts = await db.post.findMany({
  skip: (page - 1) * limit,
  take: limit,
  orderBy: { createdAt: 'desc' }
})

// Correct types
// IDs: BIGINT (not INT)
// Timestamps: TIMESTAMPTZ (not TIMESTAMP)
// Money: DECIMAL (not FLOAT)
```

### 4. Webhooks
```typescript
export async function POST(request: Request) {
  const body = await request.text()
  const signature = headers().get('stripe-signature')!

  // 1. Verify signature
  let event
  try {
    event = stripe.webhooks.constructEvent(body, signature, secret)
  } catch {
    return new Response('Invalid signature', { status: 400 })
  }

  // 2. Idempotency check
  const exists = await db.webhookEvent.findUnique({ where: { eventId: event.id } })
  if (exists) return new Response('Already processed')

  // 3. Process
  switch (event.type) {
    case 'checkout.session.completed':
      await handleCheckout(event.data.object)
      break
  }

  // 4. Mark processed
  await db.webhookEvent.create({ data: { eventId: event.id } })

  return new Response('OK')
}
```

---

## Quick Reference

### API Route
```
app/api/users/route.ts         -> GET /api/users, POST /api/users
app/api/users/[id]/route.ts    -> GET/PATCH/DELETE /api/users/:id
```

### Server Action vs API Route
```
Form submission, mutations -> Server Action (preferred)
External API access needed -> API Route
Webhooks -> API Route
```

### Supabase with RLS
```typescript
// RLS handles authorization automatically
const { data } = await supabase
  .from('posts')
  .select('*')  // Only returns user's posts due to RLS
```

---

## Security Checklist

```
[] Auth check at start
[] Zod validation on all input
[] Ownership check on resource access
[] Generic errors to client
[] Detailed errors to logs
[] No secrets in code
[] Webhook signature verification
```

---

## Red Flags (STOP)

| If You See | Fix |
|------------|-----|
| No `await auth()` | Add auth check |
| `Schema.parse(body)` without try | Use safeParse |
| No ownership check | Add authorization |
| `return { error: error.message }` | Generic error |
| `db.query(\`...${input}\`)` | Use parameterized |
| No pagination | Add pagination |
