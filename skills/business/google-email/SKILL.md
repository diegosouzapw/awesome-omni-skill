---
name: google-email
description: Manage Gmail email sync, triage, and analysis workflows. Use when user wants to sync emails, triage inbox, check email analysis results, view email statistics, manage the email processing pipeline, find newsletters, check pending emails, or understand email workflow status. Triggers on phrases like "sync my emails", "triage inbox", "email status", "check newsletters", "email analysis", "what emails do I have", "pending emails", "unsubscribe from newsletters".
---

# Google Email Management

Manage Gmail sync, AI-powered triage, and email analysis workflows.

## Prerequisites

Use the `google-account-setup` skill to configure Gmail accounts before using these workflows. Accounts must have OAuth credentials (`has_credentials: true`) for sync and triage to work.

> **Note:** Replace `<claude-assist-server>` with the actual server URL (e.g., `http://localhost:2529`). If a request fails with connection refused, ask the user for the correct server endpoint.

## Workflow States

Emails progress through a three-state pipeline:

1. **discovered** - Listed from Gmail API but content not yet fetched
2. **new** - Full content fetched, ready for AI triage
3. **triaged** - Analysis complete with EmailAnalysis populated

## Automated Schedule

The server runs these scheduled tasks automatically:

- **Incremental sync**: Every 5 minutes (fetches new emails)
- **Batch triage**: Every 5 minutes (requires `ANTHROPIC_API_KEY`)
- **Full sync**: Daily at 4 AM (complete re-sync for consistency)

## Email Analysis Structure

Each triaged email has an `analysis` field with this structure:

```typescript
interface EmailAnalysis {
  overview: string;                    // 1-2 sentence summary
  mentioned_people: string[];          // Names mentioned in email
  mentioned_organizations: string[];   // Organizations mentioned
  potential_action_items: string[];    // Action items for recipient
  sender_type: 'automated' | 'human';
  message_type: 'spam' | 'newsletter' | 'alert' | 'group' | 'personal';
  unsubscribe_link: string | null;     // Extracted unsubscribe URL
  rationale: string;                   // Explanation of classification
}
```

### Sender Type

- **automated**: System-generated, no human composed (receipts, alerts, notifications, auto-responders)
- **human**: Human composed, regardless of sending tool (CRM, ticketing system, etc.)

### Message Type

- **spam**: Unsolicited email confidently recognized as spam (phishing, scams, suspicious cold outreach). NOT newsletters.
- **newsletter**: Any email with an unsubscribe link (periodic updates, marketing, announcements). Legitimacy determined later.
- **alert**: System notifications, transactional (receipts, confirmations, calendar). No unsubscribe link typical.
- **group**: Sent to mailing list or large recipient list, not individually addressed. Check TO/CC fields.
- **personal**: Direct person-to-person, individually addressed in TO with small/relevant CC.

### Multi-Turn Analysis

The AI triage uses a conversational approach:

1. **Turn 1**: Initial analysis with automatic JSON retry on parse error
2. **Turn 2** (conditional): If classified as newsletter but missing unsubscribe link, HTML body is examined to extract the link

## API Endpoints

### Check Account Status

```bash
curl <claude-assist-server>/api/google/accounts
```

Returns accounts with `email_sync_status` and `email_triage_status` showing real-time progress.

### Sync Operations

```bash
# Incremental sync all accounts
curl -X POST <claude-assist-server>/api/google/emails/sync

# Full sync specific account
curl -X POST <claude-assist-server>/api/google/emails/sync \
  -H "Content-Type: application/json" \
  -d '{"account": "personal", "full": true}'
```

### Query Emails

```bash
# Filter by workflow status and message type
curl "<claude-assist-server>/api/google/emails?workflow_status=triaged&message_type=newsletter&days=7"

# Get single email with full details
curl <claude-assist-server>/api/google/emails/123

# Get statistics
curl "<claude-assist-server>/api/google/emails/stats?days=7"
```

**Query Parameters:**

- `account` - Filter by account identifier (e.g., "personal", "work")
- `workflow_status` - Filter by state: `discovered`, `new`, `triaged`
- `message_type` - Filter by classification: `spam`, `newsletter`, `alert`, `group`, `personal`
- `search` - Full-text search on email content
- `days` - Look back N days (default: 30)
- `limit` - Results per page (default: 50, max: 100)
- `offset` - Pagination offset

### Triage Operations

```bash
# Batch triage all 'new' emails
curl -X POST <claude-assist-server>/api/google/emails/triage

# Triage single email
curl -X POST <claude-assist-server>/api/google/emails/123/triage

# Check progress
curl <claude-assist-server>/api/google/emails/triage/progress
```

## Common Workflows

### 1. Initial Sync After Account Setup

```bash
# Trigger full sync
curl -X POST <claude-assist-server>/api/google/emails/sync \
  -H "Content-Type: application/json" \
  -d '{"account": "personal", "full": true}'

# Monitor progress
curl <claude-assist-server>/api/google/accounts
```

### 2. Check Inbox Status

```bash
# Get statistics
curl "<claude-assist-server>/api/google/emails/stats?days=7"

# Check triage progress
curl <claude-assist-server>/api/google/emails/triage/progress
```

### 3. Find Newsletters for Review

```bash
curl "<claude-assist-server>/api/google/emails?message_type=newsletter&workflow_status=triaged&days=7"
```

### 4. Find Personal Emails Needing Attention

```bash
curl "<claude-assist-server>/api/google/emails?message_type=personal&workflow_status=triaged&days=3"
```

### 5. Find Emails with Action Items

Query triaged emails and filter by those with `potential_action_items` in the analysis:

```bash
curl "<claude-assist-server>/api/google/emails?workflow_status=triaged&days=7"
```

Then filter results where `analysis.potential_action_items` is non-empty.

### 6. Manual Triage Trigger

If automatic triage isn't running or you want immediate results:

```bash
# Triage all pending emails
curl -X POST <claude-assist-server>/api/google/emails/triage

# Or triage a specific email
curl -X POST <claude-assist-server>/api/google/emails/123/triage
```
