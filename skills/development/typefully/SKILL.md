---
name: typefully
description: This skill should be used when managing social media content through Typefully, including creating drafts, scheduling posts, cross-posting to multiple accounts, and multi-platform publishing (X, LinkedIn, Threads, Bluesky, Mastodon). Use it for social media management, thread creation, and workflow automation via the Typefully API v2.
---

# Typefully Social Media Management Skill

## Overview

This skill enables Claude to interact with the Typefully API v2 for professional social media management. Typefully is a platform for drafting, scheduling, and analyzing social media content across multiple accounts and platforms.

**Supported Platforms:**
- **X (Twitter)** - Tweets and threads
- **LinkedIn** - Professional posts
- **Threads** - Meta's text-based platform
- **Bluesky** - Decentralized social network
- **Mastodon** - Federated social platform

**Key Capabilities:**
- Create drafts and scheduled posts
- Multi-platform publishing from a single draft
- Cross-post content to multiple accounts with unique adaptations
- Retrieve analytics and engagement data
- Manage multiple social media accounts through a unified interface
- Safety-first approach with draft-only mode by default

## When to Use This Skill

Use this skill when:
- Creating posts for X, LinkedIn, Threads, or Bluesky
- Scheduling social media content for future publication
- Cross-posting announcements to multiple accounts
- Publishing the same content to multiple platforms simultaneously
- Retrieving social media analytics and performance data
- Managing social media workflows programmatically

**Example triggers:**
- "Create a Twitter thread about [topic] for my personal account"
- "Post this announcement to both X and LinkedIn"
- "Schedule this post for tomorrow at 2pm"
- "Cross-post this to my company and personal accounts with appropriate adaptations"
- "Show me last week's analytics"

## Setup and Configuration

### Initial Setup

1. **Obtain Typefully API Keys:**
   - Log into Typefully dashboard (https://typefully.com)
   - Navigate to Settings > Integrations
   - Generate an API key for each social media account
   - **Note:** API keys inherit permissions from your user account

2. **Configure the Skill:**
   - Create a `.env` file in the skill directory
   - Add API keys using the format: `TYPEFULLY_API_KEY_<ACCOUNT_NAME>=your_key_here`
   - Create a `config.json` file for global settings

3. **Example Configuration:**

`.env`:
```
TYPEFULLY_API_KEY_PERSONAL=your_personal_account_key
TYPEFULLY_API_KEY_COMPANY=your_company_account_key
TYPEFULLY_API_KEY_PROJECT=your_project_account_key
```

`config.json`:
```json
{
  "scheduling_enabled": false,
  "default_platforms": ["x"],
  "default_share": true
}
```

### Safety Settings

**Draft-Only Mode (Default):**
- `scheduling_enabled: false` creates drafts without auto-scheduling
- Allows human review before publication
- Recommended until confidence in content quality is established

**Enable Scheduling:**
- Set `scheduling_enabled: true` in `config.json`
- Only enable after validating draft quality
- Requires PUBLISH permission on API key

## Core Workflows

### Workflow 1: Create Draft for Single Account

**Use Case:** Draft a post for review before publishing

**Process:**
1. Use `scripts/typefully_client.py` to interact with the API
2. Load account configuration from `.env`
3. Create draft using TypefullyManager
4. Review draft in Typefully dashboard
5. Manually schedule or publish from dashboard

**Example:**
```python
from typefully_client import TypefullyManager

manager = TypefullyManager()

# Create draft for review (X only)
result = manager.create_draft(
    account="personal",
    content="Your tweet content here.\n\n\n\nSecond tweet in thread.",
    platforms=["x"],
    schedule=False
)
print(f"Edit draft: {result['edit_url']}")
```

**Command-line usage:**
```bash
python scripts/typefully_client.py create-draft \
    --account personal \
    --content "Your tweet content" \
    --platforms x
```

**Response:**
```json
{
  "id": "draft_abc123",
  "status": "draft",
  "edit_url": "https://typefully.com/?d=draft_abc123",
  "share_url": "https://typefully.com/share/abc123",
  "scheduled_date": null
}
```

### Workflow 2: Multi-Platform Publishing

**Use Case:** Post the same announcement to X and LinkedIn

**Example:**
```python
manager = TypefullyManager()

result = manager.create_draft(
    account="company",
    content="Major product update announcement.",
    platforms=["x", "linkedin"],
    schedule=True,
    schedule_date="next-free-slot"
)
```

**Command-line usage:**
```bash
python scripts/typefully_client.py create-draft \
    --account company \
    --content "Major product update" \
    --platforms x linkedin \
    --schedule
```

**Notes:**
- The same content is posted to all specified platforms
- Platform-specific formatting is handled by Typefully
- LinkedIn posts work best with longer, more professional content

### Workflow 3: Schedule Post (When Enabled)

**Use Case:** Schedule content for future publication

**Prerequisites:**
- `scheduling_enabled: true` in `config.json`
- API key has PUBLISH permission

**Process:**
```python
manager = TypefullyManager()

result = manager.create_draft(
    account="company",
    content="Scheduled announcement content",
    platforms=["x"],
    schedule=True,
    schedule_date="2024-12-20T14:30:00Z"  # ISO format or "next-free-slot"
)
```

**Scheduling Options:**
- `"now"` - Publish immediately
- `"next-free-slot"` - Use Typefully's optimal timing
- ISO-8601 datetime - Specific time (e.g., `"2024-12-20T14:30:00Z"`)

### Workflow 4: Cross-Post to Multiple Accounts

**Use Case:** Publish the same announcement across multiple accounts with unique content for each

**Process:**
1. Prepare content variations for each account
2. Create a JSON file mapping accounts to content
3. Use the cross-post functionality

**Example content map (content.json):**
```json
{
  "personal": "Excited to share: we just launched our new feature! Check it out:",
  "company": "Introducing our latest product update with enhanced capabilities:"
}
```

**Execute cross-post:**
```python
manager = TypefullyManager()

content_map = {
    "personal": "Casual, personal tone announcement",
    "company": "Professional, formal announcement"
}

results = manager.cross_post(
    accounts=["personal", "company"],
    content_map=content_map,
    platforms=["x"],
    schedule=False
)

for account, result in results.items():
    print(f"{account}: {result.get('edit_url', result.get('error'))}")
```

**Command-line usage:**
```bash
python scripts/typefully_client.py cross-post \
    --accounts personal company \
    --content-json content.json \
    --platforms x
```

### Workflow 5: Retrieve Analytics

**Use Case:** Get performance data for recently published content

**Process:**
```python
manager = TypefullyManager()

analytics = manager.get_analytics(account="personal", limit=20)

print(f"Published: {analytics['stats']['published_count']}")
print(f"Scheduled: {analytics['stats']['scheduled_count']}")
```

**Command-line usage:**
```bash
python scripts/typefully_client.py get-analytics \
    --account personal \
    --limit 20
```

### Workflow 6: List Drafts by Status

**Use Case:** View all scheduled or draft posts

**Draft Status Values:**
- `draft` - Saved but not scheduled
- `scheduled` - Queued for future publication
- `publishing` - Currently being posted
- `published` - Successfully posted
- `error` - Publication failed

**Command-line:**
```bash
# List all scheduled drafts
python scripts/typefully_client.py get-drafts \
    --account personal \
    --status scheduled

# List all drafts (any status)
python scripts/typefully_client.py get-drafts \
    --account personal \
    --limit 50
```

### Workflow 7: View Social Sets (Connected Platforms)

**Use Case:** See which platforms are connected for an account

**Command-line:**
```bash
python scripts/typefully_client.py list-social-sets --account personal
```

**Response shows connected platforms:**
```
Social sets for personal:
  - social_set_abc123: My Account
    x: @myhandle (connected)
    linkedin: (connected)
    threads: (not connected)
```

## Thread Formatting

### Creating Multi-Tweet Threads

Use **4 consecutive newlines** (`\n\n\n\n`) to separate tweets in a thread:

```python
content = """First tweet in the thread



Second tweet with more details



Third tweet wrapping up"""

manager.create_draft(account="personal", content=content, platforms=["x"])
```

The Python client automatically converts this to the API's posts array format.

## Integration with Other Skills

This skill can be integrated with other content creation and brand management workflows:

**Content Creation Pipeline:**
1. Generate content using content creation agents
2. Validate with brand voice guidelines
3. Create Typefully draft for review
4. Manually approve and schedule (or auto-schedule if enabled)

**Multi-Brand Management:**
1. Adapt content for different brand voices
2. Use cross-post with unique content per account
3. Maintain consistent messaging with appropriate tone

**Analytics and Reporting:**
1. Retrieve performance data via analytics
2. Generate reports on engagement
3. Inform future content strategy

## API Reference

For detailed API documentation, load `references/typefully_api.md` which includes:
- Complete v2 endpoint specifications
- Request/response formats with examples
- Multi-platform content structure
- Webhook events and verification
- Error handling guidance
- Migration notes from v1

Load reference when:
- Debugging API issues
- Implementing custom functionality
- Understanding response structures
- Setting up webhooks

## Error Handling

The client provides clear, user-friendly error messages:

- **401 Unauthorized**: "Invalid API key. Check your configuration and regenerate if needed."
- **403 Forbidden**: "API key doesn't have permission for this operation."
- **429 Rate Limit**: "Rate limit exceeded. Please wait before trying again."
- **400 Bad Request**: Detailed error message with specific parameter issues

Common issues and solutions:

**Account Not Found:**
- Verify `.env` file contains `TYPEFULLY_API_KEY_<ACCOUNT>=key`
- Check account name matches exactly (case-insensitive in manager)
- Run `list-accounts` to see configured accounts

**Scheduling Disabled Warning:**
- Expected when `scheduling_enabled: false`
- Draft created for manual review
- Enable in `config.json` only when ready

**No Social Sets Available:**
- Connect at least one platform in Typefully dashboard
- Verify API key is for correct Typefully account

**Platform Not Connected:**
- The specified platform isn't connected in Typefully
- Only enabled platforms will receive posts

## Best Practices

1. **Start with Draft Mode:**
   - Keep `scheduling_enabled: false` initially
   - Review drafts in Typefully dashboard
   - Enable scheduling only after validating quality

2. **Multiple Accounts:**
   - Use descriptive account names in `.env`
   - Maintain separate API keys per account
   - Organize content_map clearly for cross-posting

3. **Multi-Platform Strategy:**
   - X for quick updates and threads
   - LinkedIn for professional announcements
   - Test platform connections before important posts

4. **Content Quality:**
   - Validate content before creating drafts
   - Use brand voice guidelines for multi-account scenarios
   - Test with personal accounts before company accounts

5. **Error Resilience:**
   - Handle API errors gracefully
   - Fall back to draft mode on scheduling failures
   - Check rate limits when batch posting

## CLI Commands Reference

```bash
# Create draft
python scripts/typefully_client.py create-draft \
    --account ACCOUNT \
    --content "Content" \
    --platforms x linkedin \
    --schedule \
    --schedule-date "next-free-slot" \
    --title "Draft Title" \
    --tags tag1 tag2

# Cross-post to multiple accounts
python scripts/typefully_client.py cross-post \
    --accounts account1 account2 \
    --content-json content.json \
    --platforms x \
    --schedule

# List drafts
python scripts/typefully_client.py get-drafts \
    --account ACCOUNT \
    --status scheduled \
    --limit 20

# Get analytics
python scripts/typefully_client.py get-analytics \
    --account ACCOUNT \
    --limit 20

# List social sets (connected platforms)
python scripts/typefully_client.py list-social-sets --account ACCOUNT

# List configured accounts
python scripts/typefully_client.py list-accounts

# Get user info
python scripts/typefully_client.py get-me --account ACCOUNT
```

## Troubleshooting

**Problem:** "Account 'xyz' not found"
- **Solution:** Check `.env` file, verify account name matches configuration

**Problem:** Scheduling doesn't work despite `schedule=True`
- **Solution:** Verify `scheduling_enabled: true` in `config.json`

**Problem:** API returns 401 Unauthorized
- **Solution:** Regenerate API key in Typefully dashboard, update `.env`

**Problem:** Cross-post fails for some accounts
- **Solution:** Check each account's API key separately, ensure all are valid

**Problem:** Platform not receiving posts
- **Solution:** Verify platform is connected in Typefully social set

**Problem:** "No social sets available"
- **Solution:** Connect at least one platform in Typefully dashboard

For additional support, consult the official Typefully API documentation: https://typefully.com/docs/api
