---
name: Resend Expert
description: Send emails via Resend API - transactional, batch, contacts, domains, webhooks, React Email
allowed-tools: Bash, Read, Edit, Write, WebFetch
---

# Resend Expert

Complete Resend email API expertise for transactional emails, marketing broadcasts, and contact management.

## Quick Reference

**Base URL**: `https://api.resend.com`
**Auth**: `Authorization: Bearer $RESEND_API_KEY`
**User's Key**: `re_Czsz1gQW_Dz4H2a9dH8tTjgteeDCjVujF`
**Verified Domain**: `sigstack.dev` (newsletter: tips@sigstack.dev)

## Core Endpoints

### Send Single Email
```bash
curl -X POST https://api.resend.com/emails \
  -H "Authorization: Bearer re_Czsz1gQW_Dz4H2a9dH8tTjgteeDCjVujF" \
  -H "Content-Type: application/json" \
  -d '{
    "from": "SigStack Tips <tips@sigstack.dev>",
    "to": ["recipient@example.com"],
    "subject": "Hello",
    "html": "<p>Email body</p>"
  }'
```

**Response**: `{"id": "uuid-string"}`

### Send Batch Emails (up to 100)
```bash
POST https://api.resend.com/emails/batch
# Body: Array of email objects (same structure as single)
# Note: No attachments or scheduling in batch mode
```

### Get/Update/Cancel Email
```bash
GET    /emails/{id}        # Retrieve email details
PATCH  /emails/{id}        # Update scheduled email
DELETE /emails/{id}/cancel # Cancel scheduled email
```

## Node.js SDK

```bash
npm install resend
```

```javascript
import { Resend } from 'resend';
const resend = new Resend('re_Czsz1gQW_Dz4H2a9dH8tTjgteeDCjVujF');

// Send email
const { data, error } = await resend.emails.send({
  from: 'SigStack <hello@sigstack.dev>',
  to: ['user@example.com'],
  subject: 'Welcome!',
  html: '<h1>Hello World</h1>',
});

// With React Email component
const { data, error } = await resend.emails.send({
  from: 'SigStack Tips <tips@sigstack.dev>',
  to: ['user@example.com'],
  subject: 'Welcome!',
  react: <WelcomeEmail name="User" />,
});

// Batch send
const { data, error } = await resend.batch.send([
  { from: '...', to: ['...'], subject: '...', html: '...' },
  { from: '...', to: ['...'], subject: '...', html: '...' },
]);
```

## Python SDK

```bash
pip install resend
```

```python
import resend
resend.api_key = "re_Czsz1gQW_Dz4H2a9dH8tTjgteeDCjVujF"

# Send email
email = resend.Emails.send({
    "from": "SigStack <hello@sigstack.dev>",
    "to": ["user@example.com"],
    "subject": "Hello",
    "html": "<p>Welcome!</p>"
})

# With attachments
email = resend.Emails.send({
    "from": "...",
    "to": ["..."],
    "subject": "...",
    "html": "...",
    "attachments": [
        {"filename": "invoice.pdf", "content": base64_content}
    ]
})
```

## Email Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `from` | string | Yes | `"Name <email@domain>"` format |
| `to` | string[] | Yes | Recipients (max 50) |
| `subject` | string | Yes | Subject line |
| `html` | string | No* | HTML body |
| `text` | string | No* | Plain text (auto-generated from html if omitted) |
| `react` | ReactNode | No* | React Email component (Node.js only) |
| `cc` | string[] | No | Carbon copy |
| `bcc` | string[] | No | Blind carbon copy |
| `reply_to` | string[] | No | Reply-to addresses |
| `scheduled_at` | string | No | ISO 8601 or natural language |
| `attachments` | array | No | `{filename, content, content_type?}` |
| `tags` | array | No | `{name, value}` pairs (256 char limit) |
| `headers` | object | No | Custom email headers |

*One of html, text, or react required

## Idempotency
```bash
-H "Idempotency-Key: unique-key-123"
# Prevents duplicate sends within 24 hours
```

## Contact Management

```bash
# Create contact
POST /contacts
{"email": "user@example.com", "first_name": "John", "unsubscribed": false}

# List contacts
GET /contacts?limit=50

# Update contact
PATCH /contacts/{id}

# Delete contact
DELETE /contacts/{id}
```

## Domain Management

```bash
# List domains
GET /domains?limit=20

# Create domain
POST /domains
{"name": "yourdomain.com"}

# Verify domain
POST /domains/{id}/verify

# Delete domain
DELETE /domains/{id}
```

## Webhooks

**Events**: `email.sent`, `email.delivered`, `email.bounced`, `email.complained`, `email.opened`, `email.clicked`, `contact.created`, `contact.updated`, `contact.deleted`

```bash
# Create webhook
POST /webhooks
{"url": "https://yourapp.com/webhook", "events": ["email.delivered", "email.bounced"]}
```

**Signature Verification** (Node.js):
```javascript
import { Webhook } from 'resend';
const webhook = new Webhook(process.env.RESEND_WEBHOOK_SECRET);
const payload = webhook.verify(body, headers);
```

## Templates

```bash
# Create template
POST /templates
{"name": "welcome", "html": "<p>Hello {{name}}</p>"}

# Send with template
POST /emails
{"from": "...", "to": ["..."], "template": {"id": "template-id", "variables": {"name": "John"}}}
```

## API Keys

```bash
# Create key
POST /api-keys
{"name": "production", "permission": "sending_access"}
# permission: "full_access" | "sending_access"

# List keys
GET /api-keys

# Delete key
DELETE /api-keys/{id}
```

## Rate Limits & Quotas
- Free tier: 100 emails/day, 3,000/month
- Batch: Max 100 emails per request
- Recipients: Max 50 per email
- Attachments: Max 40MB total

## Testing Domain
Use `onboarding@resend.dev` as sender for testing before domain verification.

## SDKs
Node.js, Python, PHP, Laravel, Ruby, Go, Rust, Java, .NET

## Common Patterns

**Newsletter (tips@sigstack.dev)**:
```javascript
await resend.emails.send({
  from: 'SigStack Tips <tips@sigstack.dev>',
  to: [subscriber.email],
  subject: 'Weekly Dev Tips',
  html: newsletterHtml,
});
```

**Transactional (noreply@sigstack.dev)**:
```javascript
await resend.emails.send({
  from: 'SigStack <noreply@sigstack.dev>',
  to: [user.email],
  subject: 'Reset your password',
  react: <PasswordResetEmail token={token} />,
});
```

**Marketing broadcast**:
```javascript
// Use Broadcasts API for marketing campaigns with unsubscribe handling
POST /broadcasts
```

## Verified Sender Addresses
- `tips@sigstack.dev` - newsletter
- `hello@sigstack.dev` - general contact
- `noreply@sigstack.dev` - transactional
- `*@sigstack.dev` - any address works

Use when: Sending transactional emails, marketing campaigns, contact management, domain setup, webhook configuration
