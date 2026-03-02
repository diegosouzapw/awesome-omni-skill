---
name: application-patterns
description: Common application development patterns and implementations
domain: domain-applications
version: 1.0.0
tags: [crud, authentication, admin, dashboard, forms, file-upload, search]
triggers:
  keywords:
    primary: [crud, authentication, admin panel, dashboard, file upload, search]
    secondary: [login, signup, form, data table, bulk operations, workflow, i18n]
  context_boost: [app, application, feature, implementation, pattern]
  context_penalty: [infrastructure, deployment, devops]
  priority: high
---

# Application Development Patterns

## Overview

Common patterns for building real-world applications. These patterns solve recurring problems in application development.

---

## CRUD Applications

### Data Flow Pattern
```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│  Form   │ ──→ │Validate │ ──→ │ Service │ ──→ │   DB    │
└─────────┘     └─────────┘     └─────────┘     └─────────┘
     ↑                                               │
     └───────────── Response ←───────────────────────┘
```

### Form Handling Best Practices

```typescript
// 1. Validation schema (shared frontend/backend)
const userSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2).max(100),
  role: z.enum(['admin', 'user', 'guest'])
});

// 2. Server action with error handling
async function createUser(formData: FormData) {
  const result = userSchema.safeParse(Object.fromEntries(formData));

  if (!result.success) {
    return { error: result.error.flatten() };
  }

  try {
    const user = await db.user.create({ data: result.data });
    return { success: true, data: user };
  } catch (e) {
    if (e.code === 'P2002') {
      return { error: { email: 'Email already exists' } };
    }
    throw e;
  }
}
```

---

## User Authentication

### Authentication Flow
```
┌────────────────────────────────────────────────────────────┐
│                    Authentication Flows                     │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Email/Password:                                           │
│  Login → Validate → Create Session → Set Cookie → Redirect │
│                                                            │
│  OAuth (Social Login):                                     │
│  Redirect → Provider Auth → Callback → Upsert User → Done │
│                                                            │
│  Magic Link:                                               │
│  Email → Generate Token → Send Link → Verify → Login       │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Session Management

| Strategy | Pros | Cons |
|----------|------|------|
| JWT | Stateless, scalable | Can't revoke easily |
| Server Session | Revocable, secure | Requires session store |
| Hybrid | Best of both | More complex |

### Security Checklist
- [ ] Password hashing (bcrypt/argon2)
- [ ] Rate limiting on login
- [ ] CSRF protection
- [ ] Secure cookie settings (httpOnly, secure, sameSite)
- [ ] Account lockout after failed attempts
- [ ] Password reset token expiration

---

## Admin Dashboards

### Data Table Pattern

```typescript
// Reusable data table with sorting, filtering, pagination
interface DataTableProps<T> {
  data: T[];
  columns: ColumnDef<T>[];
  pagination: { page: number; pageSize: number; total: number };
  sorting: { field: string; direction: 'asc' | 'desc' }[];
  filters: Record<string, unknown>;
  onStateChange: (state: TableState) => void;
}

// Server-side handling
async function getUsers(params: TableState) {
  const { page, pageSize, sorting, filters } = params;

  const query = {
    where: buildWhereClause(filters),
    orderBy: buildOrderBy(sorting),
    skip: (page - 1) * pageSize,
    take: pageSize,
  };

  const [users, total] = await Promise.all([
    db.user.findMany(query),
    db.user.count({ where: query.where })
  ]);

  return { data: users, total };
}
```

### Bulk Operations

```typescript
// Safe bulk delete with confirmation
async function bulkDelete(ids: string[]) {
  // 1. Validate permissions for each item
  const items = await db.item.findMany({
    where: { id: { in: ids } },
    select: { id: true, ownerId: true }
  });

  const authorized = items.filter(item =>
    canDelete(currentUser, item)
  );

  // 2. Soft delete or hard delete
  await db.item.updateMany({
    where: { id: { in: authorized.map(i => i.id) } },
    data: { deletedAt: new Date() }
  });

  return {
    deleted: authorized.length,
    skipped: ids.length - authorized.length
  };
}
```

---

## File Management

### Upload Strategies

| Method | Use Case | Max Size |
|--------|----------|----------|
| Direct to server | Small files | ~10MB |
| Presigned URL | Large files | Unlimited |
| Chunked upload | Very large files | Unlimited |
| Resumable | Unreliable network | Unlimited |

### Presigned URL Flow
```
Client                    Server                    S3
   │                         │                       │
   │── Request upload URL ──→│                       │
   │                         │── Generate presigned ─→│
   │←── Return presigned URL─│                       │
   │                         │                       │
   │───────── Upload file directly ─────────────────→│
   │                         │                       │
   │── Confirm upload ──────→│                       │
   │                         │── Verify file exists ─→│
   │←── Success ─────────────│                       │
```

### Image Processing Pipeline

```typescript
async function processUpload(file: File) {
  // 1. Validate file type and size
  if (!ALLOWED_TYPES.includes(file.type)) {
    throw new Error('Invalid file type');
  }

  // 2. Generate variants
  const variants = await Promise.all([
    sharp(file.buffer).resize(100, 100).toBuffer(),   // thumbnail
    sharp(file.buffer).resize(800, 600).toBuffer(),   // medium
    sharp(file.buffer).resize(1920, 1080).toBuffer(), // large
  ]);

  // 3. Upload to CDN
  const urls = await uploadToS3(variants);

  // 4. Store metadata
  return db.image.create({
    data: {
      original: urls.original,
      thumbnail: urls.thumbnail,
      medium: urls.medium,
      large: urls.large,
      mimeType: file.type,
      size: file.size,
    }
  });
}
```

---

## Search Implementation

### Search Architecture

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Database   │ ──→ │    Sync      │ ──→ │   Search     │
│  (Primary)   │     │   Worker     │     │   Engine     │
└──────────────┘     └──────────────┘     └──────────────┘
                                                  ↑
                                                  │
┌──────────────┐     ┌──────────────┐             │
│    Client    │ ──→ │  Search API  │ ────────────┘
└──────────────┘     └──────────────┘
```

### Search Features Checklist
- [ ] Full-text search
- [ ] Faceted filtering
- [ ] Autocomplete/suggestions
- [ ] Typo tolerance (fuzzy matching)
- [ ] Highlighting
- [ ] Synonyms
- [ ] Relevance tuning

---

## Workflow Engines

### State Machine Pattern

```typescript
const orderStateMachine = {
  initial: 'pending',
  states: {
    pending: {
      on: {
        PAY: 'paid',
        CANCEL: 'cancelled'
      }
    },
    paid: {
      on: {
        SHIP: 'shipped',
        REFUND: 'refunded'
      }
    },
    shipped: {
      on: {
        DELIVER: 'delivered',
        RETURN: 'returned'
      }
    },
    delivered: { type: 'final' },
    cancelled: { type: 'final' },
    refunded: { type: 'final' },
    returned: {
      on: {
        REFUND: 'refunded'
      }
    }
  }
};
```

### Approval Workflow

```typescript
interface ApprovalStep {
  id: string;
  approvers: string[];        // User IDs or roles
  requiredApprovals: number;  // How many need to approve
  timeout?: Duration;         // Auto-escalate after
  escalateTo?: string;        // Next approver on timeout
}

async function processApproval(stepId: string, userId: string, decision: 'approve' | 'reject') {
  const step = await db.approvalStep.findUnique({ where: { id: stepId } });

  // Record decision
  await db.approval.create({
    data: { stepId, userId, decision, timestamp: new Date() }
  });

  // Check if complete
  const approvals = await db.approval.count({
    where: { stepId, decision: 'approve' }
  });

  if (approvals >= step.requiredApprovals) {
    await advanceToNextStep(step);
  }
}
```

---

## Multi-language (i18n)

### Translation Structure

```
locales/
├── en/
│   ├── common.json    # Shared strings
│   ├── auth.json      # Auth module
│   └── dashboard.json # Dashboard module
├── zh-TW/
│   ├── common.json
│   ├── auth.json
│   └── dashboard.json
└── ja/
    └── ...
```

### Best Practices

1. **Key Naming**: Use namespaced keys
   ```json
   {
     "auth.login.title": "Sign In",
     "auth.login.email": "Email Address",
     "auth.login.submit": "Sign In"
   }
   ```

2. **Pluralization**: Handle plural forms
   ```json
   {
     "items": "{count, plural, =0 {No items} =1 {1 item} other {# items}}"
   }
   ```

3. **Variables**: Use interpolation
   ```json
   {
     "welcome": "Welcome, {name}!"
   }
   ```

4. **Date/Number Formatting**: Use Intl APIs
   ```typescript
   new Intl.DateTimeFormat(locale).format(date)
   new Intl.NumberFormat(locale, { style: 'currency', currency }).format(amount)
   ```

---

## Related Skills

- [[architecture-patterns]] - Overall system design
- [[frontend]] - UI implementation
- [[backend]] - Server implementation
- [[database]] - Data persistence
- [[security-practices]] - Security considerations
