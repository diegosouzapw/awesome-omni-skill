---
name: user-state-debugging
description: Expert knowledge on debugging user account issues, diagnostic scripts (inspect-user-state.js), fix scripts (fix-user-billing-state.js, reset-user-onboarding.js), onboarding problems, billing sync issues, and Clerk vs database mismatches. Use this skill when user asks about "user stuck", "onboarding broken", "billing out of sync", "debug user", "reset user", or "user state".
allowed-tools: Read, Bash, Grep
---

# User State Debugging Expert

You are an expert in debugging user account issues and using diagnostic scripts. This skill provides knowledge about common user state problems, diagnostic tools, and fix scripts.

## When To Use This Skill

This skill activates when users:
- Debug stuck onboarding flows
- Investigate billing sync issues
- Fix user account problems
- Troubleshoot trial activation failures
- Diagnose plan limit issues
- Resolve Clerk vs database mismatches
- Clean up test user data

## Core Knowledge

### Common User State Issues

**Issue Categories:**

1. **Onboarding Stuck**
   - User can't complete signup
   - Stuck at step-1 or step-2
   - Never reaches "completed"

2. **Billing Out of Sync**
   - Paid in Stripe but shows free plan
   - Plan limits incorrect
   - `billing_sync_status` shows error

3. **Trial Not Activated**
   - Completed checkout but trial inactive
   - `trial_status` is "pending"
   - Trial dates are null

4. **Clerk vs Database Mismatch**
   - User exists in Clerk but not in database
   - Email mismatch between systems
   - userId doesn't match

5. **Plan Limits Wrong**
   - User has wrong campaign/creator limits
   - Upgraded but limits not updated
   - Can't create campaigns despite having limit

### Primary Diagnostic Script

**Script:** `/scripts/inspect-user-state.js`

**Usage:**
```bash
# By email
node scripts/inspect-user-state.js --email user@example.com

# By Clerk user ID
node scripts/inspect-user-state.js --user-id user_2abc123xyz
```

**What It Shows:**
```
🔎 Inspecting user state
✅ Connected to Postgres
🆔 Resolved userId: user_2abc123xyz

👤 user_profiles:
{
  id: 'xxx',
  user_id: 'user_2abc123xyz',
  email: 'user@example.com',
  full_name: 'John Doe',
  onboarding_step: 'completed',
  trial_status: 'active',
  trial_start_date: '2025-01-15T10:00:00Z',
  trial_end_date: '2025-01-22T10:00:00Z',
  current_plan: 'glow_up',
  plan_campaigns_limit: 3,
  plan_creators_limit: 1000,
  stripe_customer_id: 'cus_xxx',
  stripe_subscription_id: 'sub_xxx',
  subscription_status: 'trialing',
  billing_sync_status: 'webhook_subscription_created',
  last_webhook_event: 'customer.subscription.created',
  created_at: '2025-01-15T09:55:00Z'
}

🎯 campaigns count: 2
[
  { id: 'xxx', name: 'Campaign 1', status: 'active', created_at: '...' },
  { id: 'yyy', name: 'Campaign 2', status: 'draft', created_at: '...' }
]

🧰 scraping_jobs count: 5

🪵 events (latest 20): 15

⏱️ Done in 245ms
```

**Key Fields to Check:**
- `onboarding_step` - Should be "completed" after signup
- `trial_status` - Should be "active" if in trial
- `current_plan` - Should match Stripe subscription
- `plan_campaigns_limit` - Should match plan definition
- `stripe_subscription_id` - Should exist if paid user
- `billing_sync_status` - Shows last webhook result
- `last_webhook_event` - Shows last Stripe webhook type

### Fix Scripts

**1. Reset User Onboarding**

**Script:** `/scripts/reset-user-onboarding.js`

**Use Case:** User stuck in onboarding, need to restart

```bash
node scripts/reset-user-onboarding.js user_2abc123xyz
```

**What It Does:**
- Sets `onboarding_step` to "pending"
- Clears trial dates
- Resets billing sync status
- Preserves Stripe data

**2. Fix Billing State**

**Script:** `/scripts/fix-user-billing-state.js`

**Use Case:** Billing out of sync, plan limits wrong

```bash
node scripts/fix-user-billing-state.js user_2abc123xyz
```

**What It Does:**
- Fetches subscription from Stripe
- Updates plan in database
- Sets correct plan limits
- Syncs trial status

**3. Complete Onboarding and Activate Plan**

**Script:** `/scripts/complete-onboarding-and-activate-plan.js`

**Use Case:** Manually complete onboarding for testing or fixing stuck user

```bash
# With plan selection
node scripts/complete-onboarding-and-activate-plan.js user_2abc123xyz glow_up

# Default to free plan
node scripts/complete-onboarding-and-activate-plan.js user_2abc123xyz
```

**What It Does:**
- Sets `onboarding_step` to "completed"
- Activates trial (if plan has trial)
- Sets plan limits
- Triggers welcome email

**4. Delete User Completely**

**Script:** `/scripts/delete-user-completely.js`

**Use Case:** Clean up test users, remove corrupted accounts

```bash
node scripts/delete-user-completely.js user_2abc123xyz
```

**WARNING:** Irreversible! Deletes:
- User profile
- All campaigns
- All scraping jobs
- All results
- All lists

**5. Reset User to Fresh State**

**Script:** `/scripts/reset-user-to-fresh-state.js`

**Use Case:** Keep user but remove all data

```bash
node scripts/reset-user-to-fresh-state.js user_2abc123xyz
```

**What It Does:**
- Deletes campaigns, jobs, results
- Resets usage counters
- Keeps user profile and billing
- Resets onboarding to pending

### Diagnostic Patterns

**Pattern 1: Onboarding Stuck Diagnosis**

```bash
# 1. Check user state
node scripts/inspect-user-state.js --email user@example.com

# 2. Look for key issues:
#    - onboarding_step != "completed"
#    - trial_status == "pending"
#    - stripe_subscription_id is null despite payment

# 3. Check Stripe dashboard
#    - Does customer exist?
#    - Is subscription active?
#    - Was webhook delivered?

# 4. Check application logs
grep "STRIPE-WEBHOOK" logs/app.log | grep "user@example.com"

# 5. Fix
node scripts/complete-onboarding-and-activate-plan.js user_xxx glow_up
```

**Pattern 2: Billing Sync Diagnosis**

```bash
# 1. Check user state
node scripts/inspect-user-state.js --user-id user_xxx

# 2. Compare with Stripe
#    - current_plan in DB vs active subscription in Stripe
#    - plan_campaigns_limit in DB vs expected for plan
#    - billing_sync_status value

# 3. Check last webhook
#    - last_webhook_event
#    - last_webhook_timestamp
#    - Look for errors in webhook delivery

# 4. Manual sync
curl -X POST http://localhost:3000/api/billing/sync-stripe \
  -H "x-dev-auth: dev-bypass" \
  -H "Content-Type: application/json" \
  -d '{"userId":"user_xxx"}'
```

**Pattern 3: Trial Not Activated**

```bash
# 1. Check trial state
node scripts/inspect-user-state.js --user-id user_xxx

# 2. Verify expected state:
#    - trial_status should be "active"
#    - trial_start_date should be set
#    - trial_end_date should be ~7 days after start
#    - subscription_status should be "trialing"

# 3. If wrong, check Stripe:
#    - Does subscription have trial_end?
#    - Is status "trialing"?

# 4. Fix manually
# Option A: Use API endpoint
curl -X POST http://localhost:3000/api/debug/trial-testing \
  -H "x-dev-auth: dev-bypass" \
  -H "Content-Type: application/json" \
  -d '{"userId":"user_xxx","action":"activate"}'

# Option B: Use SQL (careful!)
# UPDATE user_profiles SET
#   trial_status = 'active',
#   trial_start_date = NOW(),
#   trial_end_date = NOW() + INTERVAL '7 days',
#   onboarding_step = 'completed'
# WHERE user_id = 'user_xxx';
```

## Common Patterns

### Pattern 1: Full User Diagnosis

```bash
# Complete diagnostic flow
echo "=== 1. User State ===" &&
node scripts/inspect-user-state.js --email user@example.com &&
echo "" &&
echo "=== 2. Billing Status ===" &&
curl http://localhost:3000/api/billing/status \
  -H "x-dev-user-id: user_xxx" \
  -s | jq &&
echo "" &&
echo "=== 3. Campaigns ===" &&
curl http://localhost:3000/api/campaigns \
  -H "x-dev-user-id: user_xxx" \
  -s | jq
```

### Pattern 2: User Cleanup for Testing

```bash
# Clean slate for testing
node scripts/delete-user-completely.js user_test123 &&
# Create fresh test account in UI
# Or use Clerk API to create user
```

### Pattern 3: Bulk User Analysis

```bash
# List all users and their states
node scripts/list-users.js

# Find users with specific issue
node scripts/list-users.js | grep "trial_status.*pending"
```

## Troubleshooting Guide

### Problem: User Can't Complete Onboarding

**Symptoms:**
- Stuck at step-1 or step-2
- "Continue" button doesn't work
- No error message

**Diagnosis:**
```bash
# 1. Check current step
node scripts/inspect-user-state.js --email user@example.com
# Look at onboarding_step

# 2. Check browser console for errors
# Ask user to check browser console

# 3. Check API logs
# Look for POST /api/onboarding/step-1 or step-2 errors
```

**Solution:**
```bash
# Option 1: Reset and retry
node scripts/reset-user-onboarding.js user_xxx

# Option 2: Force complete
node scripts/complete-onboarding-and-activate-plan.js user_xxx free
```

### Problem: User Paid But Shows Free Plan

**Symptoms:**
- Stripe shows active subscription
- Database shows `current_plan: 'free'`
- User can't access paid features

**Diagnosis:**
```bash
# 1. Check user state
node scripts/inspect-user-state.js --user-id user_xxx

# 2. Check Stripe subscription
# - Copy stripe_customer_id from output
# - Look up in Stripe dashboard

# 3. Check webhook logs
grep "STRIPE-WEBHOOK" logs/app.log | grep "user_xxx"
```

**Solution:**
```bash
# Option 1: Trigger billing sync
curl -X POST http://localhost:3000/api/billing/sync-stripe \
  -H "x-dev-auth: dev-bypass" \
  -d '{"userId":"user_xxx"}'

# Option 2: Use fix script
node scripts/fix-user-billing-state.js user_xxx

# Option 3: Manually update database (careful!)
# Run SQL to set current_plan, plan limits, etc.
```

### Problem: Plan Limits Not Enforced

**Symptoms:**
- User exceeds limits but no error
- Can create unlimited campaigns
- `plan_campaigns_limit` is null or 0

**Diagnosis:**
```bash
# 1. Check plan limits
node scripts/inspect-user-state.js --user-id user_xxx
# Look at plan_campaigns_limit, plan_creators_limit

# 2. Check subscription_plans table
curl http://localhost:3000/api/admin/plans \
  -H "x-dev-auth: dev-bypass"

# 3. Verify plan enforcement is enabled
grep "PLAN_VALIDATION_BYPASS" .env.local
# Should NOT be set in production
```

**Solution:**
```bash
# 1. Update user's plan limits
node scripts/fix-user-billing-state.js user_xxx

# 2. Or manually via API
curl -X POST http://localhost:3000/api/admin/users/set-plan \
  -H "x-dev-auth: dev-bypass" \
  -H "Content-Type: application/json" \
  -d '{"userId":"user_xxx","plan":"glow_up"}'
```

### Problem: User Not in Database But Exists in Clerk

**Symptoms:**
- User can log in to Clerk
- API returns "User not found"
- `getUserProfile` returns null

**Diagnosis:**
```bash
# 1. Try to fetch user
node scripts/inspect-user-state.js --user-id user_xxx
# Will show "not found"

# 2. Verify user exists in Clerk
# Check Clerk dashboard

# 3. Check Clerk webhook logs
# Look for user.created webhook
```

**Solution:**
```bash
# Create user in database
node scripts/test-auto-create-user.js user_xxx

# Or trigger via API
curl http://localhost:3000/api/profile \
  -H "Authorization: Bearer $CLERK_TOKEN"
# This should auto-create user profile
```

## Related Files

- `/scripts/inspect-user-state.js` - Primary diagnostic script
- `/scripts/fix-user-billing-state.js` - Fix billing sync
- `/scripts/reset-user-onboarding.js` - Reset onboarding
- `/scripts/complete-onboarding-and-activate-plan.js` - Force complete
- `/scripts/delete-user-completely.js` - Delete user
- `/scripts/reset-user-to-fresh-state.js` - Clean user data
- `/scripts/list-users.js` - List all users
- `/scripts/find-user-id.js` - Find user by email
- `/lib/db/queries/user-queries.ts` - User query helpers
- `/app/api/debug/whoami/route.ts` - Check current auth state

## Quick Reference

**Diagnostic Commands:**
```bash
# Check user by email
node scripts/inspect-user-state.js --email user@example.com

# Check user by ID
node scripts/inspect-user-state.js --user-id user_xxx

# List all users
node scripts/list-users.js

# Find user ID by email
node scripts/find-user-id.js user@example.com
```

**Fix Commands:**
```bash
# Reset onboarding
node scripts/reset-user-onboarding.js user_xxx

# Fix billing
node scripts/fix-user-billing-state.js user_xxx

# Complete onboarding with plan
node scripts/complete-onboarding-and-activate-plan.js user_xxx glow_up

# Delete user
node scripts/delete-user-completely.js user_xxx

# Reset user data (keep profile)
node scripts/reset-user-to-fresh-state.js user_xxx
```

**API Endpoints:**
```bash
# Check auth state
curl http://localhost:3000/api/debug/whoami \
  -H "x-dev-auth: dev-bypass"

# Billing status
curl http://localhost:3000/api/billing/status \
  -H "x-dev-user-id: user_xxx"

# Sync with Stripe
curl -X POST http://localhost:3000/api/billing/sync-stripe \
  -H "x-dev-auth: dev-bypass" \
  -H "Content-Type: application/json" \
  -d '{"userId":"user_xxx"}'
```

## Best Practices

1. **Always Inspect Before Fixing** - Run `inspect-user-state.js` first
2. **Check Stripe Dashboard** - Verify subscription state matches
3. **Review Logs** - Look for webhook errors before manual fixes
4. **Backup First** - Export user data before destructive operations
5. **Test in Dev** - Try fixes on test users first
6. **Document** - Note what you fixed and why
7. **Verify Fix** - Re-run inspection after fixing
