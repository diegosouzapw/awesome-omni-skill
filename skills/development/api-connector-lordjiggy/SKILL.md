---
name: api-connector
description: Connect to 100+ popular APIs using natural language - automatic authentication, request building, and response parsing
allowed-tools: ["Bash", "Read", "Write", "WebFetch"]
version: 1.0.0
author: GLINCKER Team
license: Apache-2.0
keywords: [api, integration, rest, graphql, webhooks, oauth, automation]
---

# API Connector

**⚡ UNIQUE FEATURE**: Natural language API integration for 100+ popular services. Automatically handles authentication (OAuth, API keys), builds requests, parses responses, and generates integration code. No API docs reading required!

## What This Skill Does

Connect to any API using plain English:

- **100+ Pre-configured APIs**: GitHub, Slack, Stripe, Twilio, SendGrid, OpenAI, and more
- **Natural language requests**: "Send a Slack message to #general" → API call
- **Auto authentication**: Handles OAuth, API keys, JWT automatically
- **Response parsing**: Extracts relevant data from complex responses
- **Code generation**: Generates integration code in your language
- **Webhook setup**: Creates and manages webhooks
- **Rate limiting**: Automatically handles rate limits and retries
- **Testing mode**: Test API calls before using in production

## Supported Services

### Communication
- **Slack**: Messages, channels, users, files
- **Discord**: Servers, channels, messages, roles
- **Twilio**: SMS, voice calls, WhatsApp
- **SendGrid**: Email campaigns, templates
- **Telegram**: Bots, messages, groups

### Development
- **GitHub**: Repos, issues, PRs, actions, releases
- **GitLab**: Projects, merge requests, CI/CD
- **Bitbucket**: Repositories, pipelines
- **Linear**: Issues, projects, teams
- **Jira**: Issues, boards, sprints

### Payments
- **Stripe**: Payments, customers, subscriptions
- **PayPal**: Transactions, invoices
- **Square**: Payments, inventory

### Cloud & Infrastructure
- **AWS**: S3, Lambda, EC2, RDS
- **Google Cloud**: Storage, Compute, Functions
- **Azure**: VMs, Storage, Functions
- **Vercel**: Deployments, projects
- **Netlify**: Sites, builds, functions

### Data & Analytics
- **Google Analytics**: Reports, events, conversions
- **Mixpanel**: Events, funnels, cohorts
- **Segment**: Events, identify, track

### AI & ML
- **OpenAI**: GPT, DALL-E, Embeddings
- **Anthropic Claude**: Messages, conversations
- **Hugging Face**: Models, inference
- **Replicate**: Model predictions

### CRM & Sales
- **Salesforce**: Leads, opportunities, accounts
- **HubSpot**: Contacts, deals, companies
- **Intercom**: Users, conversations, messages

### Productivity
- **Notion**: Pages, databases, blocks
- **Airtable**: Bases, records, tables
- **Google Workspace**: Drive, Sheets, Calendar, Gmail
- **Microsoft 365**: OneDrive, Excel, Outlook

### Social Media
- **Twitter/X**: Tweets, timelines, users
- **LinkedIn**: Posts, profiles, connections
- **Instagram**: Posts, stories, insights
- **YouTube**: Videos, playlists, analytics

## Instructions

### Phase 1: Service Discovery

1. **Identify API**:
   ```
   Ask user:
   - Which service to connect to?
   - What action to perform?
   - Authentication details (if not configured)
   ```

2. **Check Configuration**:
   ```bash
   # Load API configs from .api-connector-config.yml
   Use Read to check if API is already configured
   ```

3. **Setup Authentication** (if needed):

   **For API Key auth**:
   ```
   Ask user for API key
   Securely store in config (or use environment variable)
   ```

   **For OAuth**:
   ```bash
   1. Generate OAuth URL
   2. User authorizes in browser
   3. Receive callback with code
   4. Exchange for access token
   5. Store tokens securely
   ```

   **For JWT**:
   ```
   Request credentials
   Generate JWT token
   Store for subsequent requests
   ```

### Phase 2: Natural Language to API Call

When user makes a request:

1. **Parse Intent**:
   ```
   "Send a Slack message to #general saying Hello"

   Parsed:
   - Service: Slack
   - Action: Send message
   - Channel: #general
   - Content: "Hello"
   ```

2. **Build API Request**:

   **Example 1: Slack Message**
   ```bash
   curl -X POST https://slack.com/api/chat.postMessage \
     -H "Authorization: Bearer ${SLACK_TOKEN}" \
     -H "Content-Type: application/json" \
     -d '{
       "channel": "#general",
       "text": "Hello"
     }'
   ```

   **Example 2: GitHub Create Issue**
   ```bash
   curl -X POST https://api.github.com/repos/owner/repo/issues \
     -H "Authorization: token ${GITHUB_TOKEN}" \
     -H "Content-Type: application/json" \
     -d '{
       "title": "Bug: App crashes on startup",
       "body": "Description...",
       "labels": ["bug"]
     }'
   ```

   **Example 3: Stripe Create Payment**
   ```bash
   curl -X POST https://api.stripe.com/v1/payment_intents \
     -u "${STRIPE_SECRET_KEY}:" \
     -d amount=2000 \
     -d currency=usd \
     -d "payment_method_types[]"=card
   ```

3. **Execute Request**:
   ```bash
   Use Bash to make curl request
   Or use WebFetch for simple GET requests
   ```

### Phase 3: Response Handling

1. **Parse Response**:
   ```json
   // Raw GitHub API response
   {
     "id": 123456,
     "number": 42,
     "title": "Bug: App crashes",
     "html_url": "https://github.com/owner/repo/issues/42",
     "state": "open",
     "created_at": "2025-01-13T10:30:00Z",
     ...100 more fields
   }
   ```

2. **Extract Relevant Data**:
   ```
   ✅ Issue created successfully!

   Issue #42: Bug: App crashes
   URL: https://github.com/owner/repo/issues/42
   Status: Open
   ```

3. **Handle Errors**:
   ```
   ❌ API Error: Rate limit exceeded

   Details:
   - Limit: 5000 requests/hour
   - Remaining: 0
   - Resets at: 2025-01-13 11:00:00 UTC

   Suggestion: Retry in 15 minutes or use a different token
   ```

### Phase 4: Code Generation

Offer to generate integration code:

**Python**:
```python
import requests

def send_slack_message(channel: str, text: str) -> dict:
    """Send a message to a Slack channel."""
    url = "https://slack.com/api/chat.postMessage"
    headers = {
        "Authorization": f"Bearer {os.environ['SLACK_TOKEN']}",
        "Content-Type": "application/json"
    }
    data = {
        "channel": channel,
        "text": text
    }

    response = requests.post(url, headers=headers, json=data)
    response.raise_for_status()
    return response.json()

# Usage
result = send_slack_message("#general", "Hello from Python!")
print(f"Message sent: {result['ts']}")
```

**JavaScript**:
```javascript
async function sendSlackMessage(channel, text) {
  const response = await fetch('https://slack.com/api/chat.postMessage', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.SLACK_TOKEN}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ channel, text })
  });

  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }

  return await response.json();
}

// Usage
const result = await sendSlackMessage('#general', 'Hello from JavaScript!');
console.log(`Message sent: ${result.ts}`);
```

## Examples

### Example 1: GitHub Automation

**User**: "Create a GitHub issue in my-repo about fixing the login bug"

**Workflow**:
1. Parse: repo=my-repo, action=create issue, title=about login bug
2. Build request with GitHub API
3. Execute: POST /repos/owner/my-repo/issues
4. Show result: "Issue #42 created: https://github.com/..."
5. Offer code generation

**Result**:
```
✅ GitHub issue created!

Issue #42: Fix login bug
Repository: owner/my-repo
URL: https://github.com/owner/my-repo/issues/42

Would you like me to:
1. Generate Python/JS code for this
2. Set up a webhook to track this issue
3. Create a similar issue in another repo
```

### Example 2: Multi-Service Workflow

**User**: "When a new Stripe payment succeeds, send a thank you email via SendGrid and log it to Airtable"

**Workflow**:
1. Setup Stripe webhook
2. Generate webhook handler code:
   ```python
   @app.route('/webhook/stripe', methods=['POST'])
   def stripe_webhook():
       event = stripe.Webhook.construct_event(
           request.data, request.headers['STRIPE_SIGNATURE'],
           webhook_secret
       )

       if event['type'] == 'payment_intent.succeeded':
           payment = event['data']['object']
           email = payment['receipt_email']

           # Send thank you email
           send_email_sendgrid(
               to=email,
               subject="Thank you for your purchase!",
               body="Your payment has been received."
           )

           # Log to Airtable
           airtable_create_record('Payments', {
               'Customer': email,
               'Amount': payment['amount'] / 100,
               'Date': datetime.now()
           })

       return {'status': 'success'}
   ```

### Example 3: Bulk Operations

**User**: "Get all open PRs from my GitHub repos and send a summary to Slack"

**Workflow**:
1. Fetch repos: GET /user/repos
2. For each repo: GET /repos/{owner}/{repo}/pulls?state=open
3. Aggregate results
4. Format message
5. Send to Slack

**Result**:
```
📊 Open Pull Requests Summary

my-api: 3 open PRs
- #42: Add authentication (2 days old)
- #43: Update dependencies (5 days old)
- #44: Fix bug in login (1 day old)

my-frontend: 1 open PR
- #15: Redesign homepage (3 days old)

Total: 4 open PRs across 2 repos

Posted to #engineering on Slack ✅
```

## Configuration

`.api-connector-config.yml`:

```yaml
services:
  slack:
    auth_type: oauth
    token: ${SLACK_TOKEN}
    workspace: my-workspace

  github:
    auth_type: token
    token: ${GITHUB_TOKEN}
    default_owner: my-org

  stripe:
    auth_type: api_key
    secret_key: ${STRIPE_SECRET_KEY}
    publishable_key: ${STRIPE_PUBLISHABLE_KEY}

  sendgrid:
    auth_type: api_key
    api_key: ${SENDGRID_API_KEY}
    from_email: noreply@example.com

defaults:
  timeout: 30s
  retry_count: 3
  rate_limit_handling: auto

webhooks:
  base_url: https://myapp.com/webhooks
  secret: ${WEBHOOK_SECRET}
```

## Advanced Features

### 1. Batch Operations

```
User: "Star all repos from @anthropics on GitHub"

Skill:
1. List repos: GET /users/anthropics/repos
2. For each repo: PUT /user/starred/{owner}/{repo}
3. Show progress
4. Handle rate limits
```

### 2. Webhook Management

```bash
# Create webhook
claude api webhook create github \
  --url https://myapp.com/webhook \
  --events push,pull_request

# List webhooks
claude api webhook list github

# Test webhook
claude api webhook test github webhook-id
```

### 3. API Explorer

```bash
# Discover available endpoints
claude api explore github

# Show endpoint documentation
claude api docs github repos.create
```

### 4. Request Builder

Interactive mode for complex APIs:
```
claude api build stripe payment

1. What type of operation? [create/read/update/delete] create
2. Resource? payment_intent
3. Amount? 5000
4. Currency? usd
5. Payment methods? card
6. Save customer? yes

Generated request:
[Shows curl command]

Execute now? [yes/no]
```

## Tool Requirements

- **Bash**: Execute API calls with curl
- **Read**: Load configuration and credentials
- **Write**: Save responses and generated code
- **WebFetch**: Simple GET requests

## Security

- Never logs sensitive data (tokens, passwords)
- Uses environment variables for secrets
- Validates SSL certificates
- Sanitizes request/response logs

## Best Practices

1. **Store secrets in environment variables**: Never commit API keys
2. **Test before production**: Use testing endpoints when available
3. **Handle rate limits**: Respect API limits
4. **Error handling**: Always check response status
5. **Logging**: Log API calls for debugging

## Limitations

- Rate limits depend on your API plan
- Complex OAuth flows may need manual intervention
- Some APIs require IP whitelisting
- Webhook setup requires public URL

## Related Skills

- [workflow-composer](../../automation/workflow-composer/SKILL.md) - Chain API calls
- [database-query](../database-query/SKILL.md) - Store API results
- [slack-bridge](../slack-bridge/SKILL.md) - Advanced Slack integration

## Changelog

### Version 1.0.0 (2025-01-13)
- Initial release
- 100+ API integrations
- OAuth, API key, JWT auth
- Code generation (Python, JavaScript)
- Webhook support
- Natural language requests

## Contributing

Help expand API support:
- Add new services
- Improve error messages
- Add more code examples
- Create integration templates

## License

Apache License 2.0 - See [LICENSE](../../../LICENSE)

## Author

**GLINCKER Team**
- GitHub: [@GLINCKER](https://github.com/GLINCKER)
- Repository: [claude-code-marketplace](https://github.com/GLINCKER/claude-code-marketplace)

---

**🌟 Connect to any API without reading documentation - just ask!**
