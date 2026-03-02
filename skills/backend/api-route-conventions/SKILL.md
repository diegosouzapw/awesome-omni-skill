---
name: api-route-conventions
description: Expert knowledge on Next.js API route patterns, authentication with getAuthOrTest, error handling, response formats, rate limiting, and webhook verification. Use this skill when user asks about "create api endpoint", "api route", "error handling", "authentication", "next.js api", or "route handler".
allowed-tools: Read, Write, Edit, Grep, Glob
---

# API Route Conventions Expert

You are an expert in Next.js API route conventions for this platform. This skill provides templates, patterns, and best practices for creating consistent, secure API endpoints.

## When To Use This Skill

This skill activates when users:
- Need to create a new API endpoint
- Debug authentication issues in routes
- Implement error handling patterns
- Work with webhook endpoints (Stripe, Clerk, QStash)
- Need consistent response formats
- Implement rate limiting or validation
- Convert old API routes to new patterns

## Core Knowledge

### Standard API Route Template

**Location:** `/app/api/[feature]/route.ts`

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { getAuthOrTest } from '@/lib/auth/get-auth-or-test';
import { logger, LogCategory } from '@/lib/logging';
import { db } from '@/lib/db';

export async function GET(req: NextRequest) {
  try {
    // 1. Authentication
    const auth = await getAuthOrTest();
    if (!auth?.userId) {
      return NextResponse.json(
        { error: 'Unauthorized' },
        { status: 401 }
      );
    }

    // 2. Parse query parameters
    const searchParams = req.nextUrl.searchParams;
    const param = searchParams.get('param');

    // 3. Validate input
    if (!param) {
      return NextResponse.json(
        { error: 'Missing required parameter: param' },
        { status: 400 }
      );
    }

    // 4. Business logic
    const data = await db.query.someTable.findMany({
      where: eq(someTable.userId, auth.userId)
    });

    // 5. Success response
    return NextResponse.json({
      success: true,
      data,
      meta: {
        count: data.length,
        timestamp: new Date().toISOString()
      }
    });

  } catch (error) {
    // 6. Error handling
    logger.error('Failed to process request', error, {
      endpoint: '/api/feature',
      userId: auth?.userId
    }, LogCategory.API);

    return NextResponse.json(
      {
        error: 'Internal server error',
        message: error instanceof Error ? error.message : 'Unknown error'
      },
      { status: 500 }
    );
  }
}

export async function POST(req: NextRequest) {
  try {
    // 1. Authentication
    const auth = await getAuthOrTest();
    if (!auth?.userId) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    // 2. Parse body
    const body = await req.json();

    // 3. Validate input
    if (!body.name || !body.type) {
      return NextResponse.json(
        { error: 'Missing required fields: name, type' },
        { status: 400 }
      );
    }

    // 4. Business logic with logging
    logger.info('Creating resource', {
      userId: auth.userId,
      name: body.name
    }, LogCategory.API);

    const [resource] = await db.insert(someTable)
      .values({
        userId: auth.userId,
        name: body.name,
        type: body.type
      })
      .returning();

    // 5. Success response
    return NextResponse.json({
      success: true,
      data: resource
    }, { status: 201 });

  } catch (error) {
    logger.error('Failed to create resource', error, {
      endpoint: '/api/feature'
    }, LogCategory.API);

    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

### Authentication Patterns

**Primary Auth:** `/lib/auth/get-auth-or-test.ts`

```typescript
import { getAuthOrTest } from '@/lib/auth/get-auth-or-test';

// Standard pattern
const auth = await getAuthOrTest();
if (!auth?.userId) {
  return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
}

// With email access
const auth = await getAuthOrTest();
const email = auth?.sessionClaims?.email as string | undefined;
```

**Auth Resolution Order:**
1. Test headers (`x-test-user-id`, `x-test-email`)
2. Dev bypass header (`x-dev-auth: dev-bypass`)
3. Environment bypass (`ENABLE_AUTH_BYPASS=true`)
4. Clerk `backendAuth()`

**Dev Bypass Methods:**

1. **Header-Based:**
```bash
curl http://localhost:3000/api/endpoint \
  -H "x-dev-auth: dev-bypass" \
  -H "x-dev-user-id: user_xxx"
```

2. **Environment-Based:**
```bash
# .env.local
ENABLE_AUTH_BYPASS=true
AUTH_BYPASS_USER_ID=user_xxx
```

3. **Test Headers:**
```bash
curl http://localhost:3000/api/endpoint \
  -H "x-test-user-id: user_xxx" \
  -H "x-test-email: test@example.com"
```

### Response Formats

**Success Response:**
```typescript
return NextResponse.json({
  success: true,
  data: result,
  meta: {
    count: result.length,
    page: 1,
    timestamp: new Date().toISOString()
  }
}, { status: 200 });
```

**Error Response:**
```typescript
return NextResponse.json({
  error: 'Error message',
  code: 'ERROR_CODE', // Optional
  details: { /* ... */ } // Optional
}, { status: 400 });
```

**Status Codes:**
- `200` - Success (GET, PUT, DELETE)
- `201` - Created (POST)
- `400` - Bad Request (validation failed)
- `401` - Unauthorized (no auth)
- `403` - Forbidden (auth but no permission, e.g., plan limits)
- `404` - Not Found
- `409` - Conflict (duplicate resource)
- `429` - Too Many Requests (rate limit)
- `500` - Internal Server Error

### Webhook Pattern

**Stripe Webhook:** `/app/api/stripe/webhook/route.ts`

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { StripeService } from '@/lib/stripe/stripe-service';

export async function POST(req: NextRequest) {
  try {
    // 1. Get raw body (required for signature verification)
    const body = await req.text();
    const signature = req.headers.get('stripe-signature');

    if (!signature) {
      return NextResponse.json(
        { error: 'No signature provided' },
        { status: 400 }
      );
    }

    // 2. Verify signature
    const event = StripeService.validateWebhookSignature(body, signature);

    // 3. Handle event
    switch (event.type) {
      case 'customer.subscription.created':
        await handleSubscriptionCreated(event.data.object);
        break;
      // ... other cases
      default:
        logger.info('Unhandled webhook event', {
          type: event.type
        });
    }

    // 4. Always return 200 (even for unhandled events)
    return NextResponse.json({ received: true });

  } catch (error) {
    logger.error('Webhook error', error);
    return NextResponse.json(
      { error: 'Webhook handler failed' },
      { status: 500 }
    );
  }
}
```

**QStash Webhook:** `/app/api/qstash/*/route.ts`

```typescript
import { Receiver } from '@upstash/qstash';

const receiver = new Receiver({
  currentSigningKey: process.env.QSTASH_CURRENT_SIGNING_KEY!,
  nextSigningKey: process.env.QSTASH_NEXT_SIGNING_KEY!,
});

export async function POST(req: Request) {
  const rawBody = await req.text();
  const signature = req.headers.get('Upstash-Signature');

  // Verify signature (skip in dev if configured)
  if (shouldVerifySignature()) {
    if (!signature) {
      return NextResponse.json({ error: 'Missing signature' }, { status: 401 });
    }

    const valid = await receiver.verify({
      signature,
      body: rawBody,
      url: callbackUrl
    });

    if (!valid) {
      return NextResponse.json({ error: 'Invalid signature' }, { status: 401 });
    }
  }

  // Process webhook...
}
```

### Validation Patterns

**Zod Schema Validation:**
```typescript
import { z } from 'zod';

const CreateCampaignSchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().optional(),
  searchType: z.enum(['instagram-reels', 'tiktok-keyword', 'youtube-keyword']),
  keywords: z.array(z.string()).min(1).max(10)
});

export async function POST(req: NextRequest) {
  const body = await req.json();

  // Validate
  const validation = CreateCampaignSchema.safeParse(body);
  if (!validation.success) {
    return NextResponse.json({
      error: 'Validation failed',
      details: validation.error.issues
    }, { status: 400 });
  }

  const data = validation.data;
  // Use validated data...
}
```

**Manual Validation:**
```typescript
function validateInput(data: any): { valid: boolean; error?: string } {
  if (!data.name || typeof data.name !== 'string') {
    return { valid: false, error: 'Invalid name' };
  }
  if (data.name.length > 100) {
    return { valid: false, error: 'Name too long' };
  }
  return { valid: true };
}

const validation = validateInput(body);
if (!validation.valid) {
  return NextResponse.json({ error: validation.error }, { status: 400 });
}
```

### Plan Enforcement Integration

```typescript
import { PlanEnforcementService } from '@/lib/services/plan-enforcement';

export async function POST(req: NextRequest) {
  const auth = await getAuthOrTest();
  if (!auth?.userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  // Validate plan limits BEFORE action
  const validation = await PlanEnforcementService.validateCampaignCreation(
    auth.userId
  );

  if (!validation.allowed) {
    return NextResponse.json({
      error: validation.reason,
      usage: validation.usage,
      upgrade_required: true
    }, { status: 403 });
  }

  // Create resource...

  // Track usage AFTER success
  await PlanEnforcementService.trackCampaignCreated(auth.userId);

  return NextResponse.json({ success: true });
}
```

## Common Patterns

### Pattern 1: Dynamic Route with ID

```typescript
// app/api/campaigns/[id]/route.ts

export async function GET(
  request: Request,
  { params }: { params: { id: string } }
) {
  const auth = await getAuthOrTest();
  if (!auth?.userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const campaignId = params.id;

  const campaign = await db.query.campaigns.findFirst({
    where: and(
      eq(campaigns.id, campaignId),
      eq(campaigns.userId, auth.userId) // Security: user can only access own data
    )
  });

  if (!campaign) {
    return NextResponse.json({ error: 'Campaign not found' }, { status: 404 });
  }

  return NextResponse.json({ data: campaign });
}

export async function DELETE(
  request: Request,
  { params }: { params: { id: string } }
) {
  const auth = await getAuthOrTest();
  if (!auth?.userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  // Verify ownership before delete
  const campaign = await db.query.campaigns.findFirst({
    where: and(
      eq(campaigns.id, params.id),
      eq(campaigns.userId, auth.userId)
    )
  });

  if (!campaign) {
    return NextResponse.json({ error: 'Not found' }, { status: 404 });
  }

  await db.delete(campaigns).where(eq(campaigns.id, params.id));

  return NextResponse.json({ success: true });
}
```

### Pattern 2: Pagination

```typescript
export async function GET(req: NextRequest) {
  const auth = await getAuthOrTest();
  if (!auth?.userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const searchParams = req.nextUrl.searchParams;
  const page = parseInt(searchParams.get('page') || '1');
  const limit = Math.min(parseInt(searchParams.get('limit') || '20'), 100);
  const offset = (page - 1) * limit;

  const [items, [{ total }]] = await Promise.all([
    db.query.campaigns.findMany({
      where: eq(campaigns.userId, auth.userId),
      orderBy: [desc(campaigns.createdAt)],
      limit,
      offset
    }),
    db.select({ total: count() })
      .from(campaigns)
      .where(eq(campaigns.userId, auth.userId))
  ]);

  return NextResponse.json({
    data: items,
    meta: {
      page,
      limit,
      total,
      totalPages: Math.ceil(total / limit)
    }
  });
}
```

### Pattern 3: Admin-Only Endpoint

```typescript
import { isAdmin } from '@/lib/auth/admin-utils';

export async function POST(req: NextRequest) {
  const auth = await getAuthOrTest();
  if (!auth?.userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  // Check admin status
  if (!await isAdmin(auth.userId)) {
    return NextResponse.json({ error: 'Forbidden' }, { status: 403 });
  }

  // Admin-only logic...
}
```

## Anti-Patterns (Avoid These)

### Anti-Pattern 1: No Auth Check

```typescript
// BAD: Anyone can access
export async function POST(req: NextRequest) {
  const body = await req.json();
  await db.insert(campaigns).values(body);
  return NextResponse.json({ success: true });
}
```

**Do this instead:**
```typescript
// GOOD: Always check auth
export async function POST(req: NextRequest) {
  const auth = await getAuthOrTest();
  if (!auth?.userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }
  // ...
}
```

### Anti-Pattern 2: Exposing Internal Errors

```typescript
// BAD: Exposes stack traces and DB details
catch (error) {
  return NextResponse.json({ error: error.toString() }, { status: 500 });
}
```

**Do this instead:**
```typescript
// GOOD: Log full error, return safe message
catch (error) {
  logger.error('Operation failed', error, { userId: auth?.userId });
  return NextResponse.json({
    error: 'Internal server error',
    message: process.env.NODE_ENV === 'development' ? error.message : undefined
  }, { status: 500 });
}
```

### Anti-Pattern 3: No Input Validation

```typescript
// BAD: Trusting user input
const { name, email } = await req.json();
await db.insert(users).values({ name, email });
```

**Do this instead:**
```typescript
// GOOD: Validate before using
const body = await req.json();
if (!body.name || typeof body.name !== 'string' || body.name.length > 100) {
  return NextResponse.json({ error: 'Invalid name' }, { status: 400 });
}
```

## Related Files

- `/lib/auth/get-auth-or-test.ts` - Authentication resolver
- `/lib/auth/admin-utils.ts` - Admin check
- `/lib/services/plan-enforcement.ts` - Plan validation
- `/lib/logging/index.ts` - Structured logging
- `/app/api/campaigns/route.ts` - Example CRUD endpoint
- `/app/api/stripe/webhook/route.ts` - Webhook pattern
- `/app/api/qstash/process-search/route.ts` - QStash pattern

## Testing API Endpoints

**Test with curl:**
```bash
# With dev bypass
curl -X POST http://localhost:3000/api/campaigns \
  -H "x-dev-auth: dev-bypass" \
  -H "x-dev-user-id: user_xxx" \
  -H "Content-Type: application/json" \
  -d '{"name":"Test Campaign","searchType":"instagram-reels"}'

# With Clerk session (production)
curl http://localhost:3000/api/campaigns \
  -H "Authorization: Bearer $CLERK_SESSION_TOKEN"
```

**Test script:**
```bash
node scripts/simple-api-logger.js
```
