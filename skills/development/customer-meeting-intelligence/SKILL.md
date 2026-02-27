---
name: customer-meeting-intelligence
description: "Comprehensive meeting intelligence system for customer-facing calls. Provides proactive pre-call briefings with account context, post-meeting action item extraction with draft approval, and thread continuity. Use when you need: (1) automated morning briefings for customer meetings, (2) post-meeting action item extraction from Fellow.ai and Gmail Gemini notes, (3) draft approval workflow before posting meeting notes, (4) customer context gathering including billing, support tickets, and communication history, (5) meeting follow-up automation with account intelligence."
---

# Customer Meeting Intelligence System

A comprehensive meeting intelligence platform that transforms reactive meeting notes into proactive customer engagement through automated briefings, context gathering, and intelligent follow-up workflows.

## Core Features

### 🌅 Pre-Call Intelligence (8:30 AM daily)
- **Calendar scanning** identifies customer meetings for the day
- **Account context** gathering from billing, support, and communication history  
- **Customer intelligence** includes spend trends, recent escalations, previous meeting notes
- **Proactive briefings** posted to Slack DM before each customer call
- **Thread continuity** links pre-call brief with post-meeting follow-up

### 📝 Post-Meeting Processing (3:33 PM daily)
- **Action item extraction** from Fellow.ai meetings and Gmail Gemini notes
- **Draft approval workflow** prevents accidental posting with review process
- **Enhanced context** enriches meeting notes with customer account health
- **Smart filtering** only processes meetings where you were a participant
- **Thread linking** connects back to morning pre-call brief

### 🔄 Draft Review & Approval
- **Slack draft generation** with meeting summary and action items
- **Approval reactions** (✅ approve, ✏️ edit, ❌ skip) for controlled posting
- **Context preservation** maintains account intelligence across meeting lifecycle
- **Team coordination** splits action items by assignee for clear ownership

## Quick Start

### Morning Pre-Call Briefings
```bash
# Generate today's pre-call briefings
bash scripts/pre-call-brief.sh

# Generate briefings for specific date  
bash scripts/pre-call-brief.sh --date 2026-02-21

# Dry run — see what would be generated
bash scripts/pre-call-brief.sh --dry-run
```

### Post-Meeting Action Items
```bash
# Process last 24h of meetings with enhanced context
bash scripts/meeting-next-steps.sh --slack

# Check specific time period
bash scripts/meeting-next-steps.sh --days 3

# Generate draft for approval workflow
bash scripts/meeting-next-steps.sh --output meeting-data.json
bash scripts/draft-approval.sh meeting-data.json
```

### Draft Approval Workflow
```bash
# Generate and post draft for review
bash scripts/draft-approval.sh meeting-data.json

# Post to specific customer channel after approval
bash scripts/draft-approval.sh meeting-data.json "#customer-channel"

# Test draft format without posting
DRY_RUN=true bash scripts/draft-approval.sh meeting-data.json
```

## Configuration

Copy `assets/config-template.json` to your working directory as `config/config.json` and customize:

```json
{
  "fellow": {
    "api_key": "your-fellow-api-key",
    "base_url": "https://api.fellow.app"
  },
  "filters": {
    "attendee_email": "your-email@company.com",
    "skip_keywords": ["1:1", "internal", "standup"]
  },
  "slack": {
    "dm_channel": "your-dm-channel-id",
    "bot_token_env": "SLACK_BOT_TOKEN"
  },
  "integrations": {
    "billing_agent_url": "http://your-billing-agent:8000/a2a/billing-account/rpc",
    "escalation_tracker": "~/memory/escalation-router-tracker.json",
    "salesforce_enabled": true
  }
}
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `FELLOW_API_KEY` | Yes | Fellow.app API key for meeting data |
| `SLACK_BOT_TOKEN` | Yes | Slack bot token with `chat:write` and `reactions:write` scopes |
| `TELNYX_API_KEY` | Optional | For billing A2A agent integration |
| `DRY_RUN` | Optional | Set `true` to preview without posting |

## Customer Intelligence Sources

### Account Context
- **Billing data** via A2A billing agent (spend trends, credit status, payment history)
- **Support tickets** from escalation tracking and ENGDESK systems  
- **Usage patterns** from customer spend monitoring dashboards
- **Relationship status** from previous meeting notes and communication history

### Communication History  
- **Gmail integration** via gog CLI for recent email threads
- **Fellow.ai meetings** for previous call context and outcomes
- **Slack conversations** for ongoing support discussions
- **Calendar analysis** for meeting frequency and engagement patterns

### Opportunity Intelligence
- **Salesforce integration** for open opportunities and pipeline status
- **Renewal tracking** for upcoming contract discussions
- **Expansion signals** from usage growth and feature requests
- **Competitive intelligence** from support tickets and meeting notes

## Advanced Features

### Smart Customer Detection
Automatically identifies customer meetings by:
- **External attendees** not from your company domain
- **Customer keywords** in meeting titles (configurable)  
- **Domain mapping** for known customer domains
- **VIP flagging** for strategic accounts requiring extra context

### Context Enrichment
Enhances meeting data with:
- **Recent escalations** from support ticket tracking (last 7 days)
- **Billing alerts** including credit risk and payment issues
- **Usage anomalies** from spend monitoring systems
- **Previous commitments** from historical meeting notes
- **Account health scores** from multiple data sources

### Thread Continuity
Maintains conversation flow by:
- **Morning brief threads** that connect to post-meeting notes
- **Customer history** linking all meetings for the same account
- **Action item tracking** across multiple meetings and time periods
- **Commitment monitoring** ensuring follow-through on promises made

## Integration Points

### Required Tools
- **gog CLI** for Gmail/Calendar access (`/opt/homebrew/bin/gog`)
- **Fellow.ai API** for meeting transcripts and action items
- **Slack API** for posting briefs and collecting approvals
- **curl/jq** for API interactions and JSON processing

### Optional Integrations  
- **Billing A2A agent** for customer account financial health
- **Escalation tracker** for recent support ticket activity
- **Salesforce API** for opportunity and renewal context
- **Memory files** for historical meeting notes and customer insights

## Best Practices

### Customer Names & Privacy
- **Internal communications** (Slack) use actual customer names
- **External communications** use "partner carrier", "losing carrier", etc.
- **Thread subjects** include customer name for easy identification
- **Context sharing** respects customer confidentiality boundaries

### Meeting Efficiency
- **Pre-call preparation** reduces time spent gathering context during calls
- **Action item clarity** ensures clear ownership and follow-through
- **Draft review** prevents miscommunication and incomplete notes
- **Historical context** builds on previous meeting outcomes

### Account Management
- **Relationship tracking** maintains continuity across team members
- **Escalation awareness** surfaces recent issues before they become problems  
- **Opportunity recognition** identifies upsell and expansion possibilities
- **Risk mitigation** flags accounts with declining engagement or usage

## Troubleshooting

### Common Issues
- **No meetings found**: Check calendar permissions and date filters
- **Missing context**: Verify API keys and integration endpoints  
- **Draft not posted**: Confirm Slack bot permissions and channel access
- **Customer not detected**: Review domain mapping and detection rules

### Error Recovery
- **Partial context**: System continues with available data sources
- **API failures**: Implements retry logic with exponential backoff
- **Missing participants**: Filters ensure only relevant meetings are processed
- **Channel errors**: Graceful fallback to DM delivery

## File Structure
```
scripts/
├── pre-call-brief.sh       # Morning customer meeting preparation
├── meeting-next-steps.sh   # Post-meeting action item extraction  
├── draft-approval.sh       # Review workflow for meeting notes
└── format-actions.sh       # Slack formatting utilities

assets/
└── config-template.json    # Configuration template

references/
└── (dynamically populated with customer-specific context)
```

## Automation Schedule

Set up these cron jobs for full automation:

**Morning Pre-Call Briefings:**
```bash
30 8 * * 1-5  # 8:30 AM CST, Monday-Friday
```

**Post-Meeting Action Items:**  
```bash
33 15 * * 1-5  # 3:33 PM CST, Monday-Friday
```

Both can run as isolated sub-agent sessions with automatic delivery to your Slack DM for review and action.