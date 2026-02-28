---
name: linkt-outreach
description: Draft a LinkedIn connection message and post to Slack for manual sending. Use after /linkt-signals when user wants to reach out to a contact, or when user provides a LinkedIn URL and context for outreach.
user-invocable: true
allowed-tools: mcp__linkt__get_entity_v1_entity, mcp__linkt__get_signal_v1_signal, AskUserQuestion, Read, mcp__slack__slack_post_message
---

# Linkt Outreach Skill

Draft personalized LinkedIn connection requests and post to Slack for manual sending.

## Configuration

**Slack Channel:** `#linkt-connections` (ID: `C0ACE4HJLBD`)

## Prerequisites

This skill requires Slack MCP to be configured. See `docs/slack-setup.md` for setup instructions.

## Input Context

This skill expects one of:
1. **From /linkt-signals:** Signal context, company info, and contact with LinkedIn URL
2. **Direct invocation:** User provides LinkedIn URL and optional context

Required information:
- Contact LinkedIn URL
- Contact name and title
- Company name

Optional but helpful:
- Signal summary (what triggered the outreach)
- Company context (industry, what they do)
- Shared interests or relevant talking points

## Workflow

### Step 0: Load User Context

First, try to read `.claude/user-context.json` for personalization context.

**If the file exists**, load the user context for use in message drafting:
- Company name and description
- User's role
- Value proposition
- Talking points
- User's name (for sign-off)

**If the file doesn't exist**, continue without user context. Messages will be more generic but still functional. Optionally mention: "Tip: Run `/linkt-init` to set up your profile for more personalized messages."

### Step 1: Gather Context

If invoked directly (not from /linkt-signals), ask the user for:
- LinkedIn profile URL
- Why they want to connect (signal, shared interest, etc.)
- Any specific talking points

If context was passed from /linkt-signals, confirm the details:
```
**Outreach Context:**
- Contact: [Name] ([Title])
- Company: [Company Name]
- Signal: [Signal summary]
- LinkedIn: [URL]

Is this correct? (yes/edit)
```

### Step 2: Draft Connection Message

Craft a personalized LinkedIn connection request (max 200 characters for connection note).

**Message Guidelines:**
- Lead with relevance (the signal or shared interest)
- Be specific about why you're reaching out
- Keep it concise and professional
- Avoid generic templates
- Don't be salesy in the connection request

**Personalization Rules (when user context is available):**

If user context was loaded from `.claude/user-context.json`:

1. **Subtle company reference:** Mention user's company naturally when relevant to the signal. Don't force it.
   - Good: "At [Company], we help teams like yours with [relevant capability]"
   - Bad: "[Company] is the #1 solution for..." (too salesy)

2. **Value proposition:** Weave in the value proposition when it connects to the signal context.
   - If the signal is about AI initiatives and user helps with AI, mention the connection
   - If there's no natural fit, omit it entirely

3. **Talking points:** Use a talking point only if directly relevant to the contact's situation.
   - Recent funding → mention if discussing growth/scaling
   - Press coverage → mention if it relates to the signal topic
   - Don't shoehorn talking points where they don't fit

4. **Sign-off with name:** If user.name is set, end with "- [Name]" for a personal touch.

5. **Tone alignment:** Match the use_case:
   - sales → professional but warm, focus on mutual value
   - recruiting → friendly and opportunity-focused
   - partnerships → collaborative and strategic
   - networking → casual and interest-based

**Message Template Structure:**
```
Hi [First Name],

[1-2 sentences referencing the signal and why connecting would be valuable]

[Optional: sign-off with user's name if available]
```

**Example Messages:**

Without user context (generic):
```
Hi Sarah,

Your recent article on AI transformation resonated with me. Would love to connect and exchange insights.
```

With user context (personalized):
```
Hi Sarah,

Your article on AI in sales caught my attention - at Linkt, we help GTM teams discover prospects based on these signals. Would love to connect!

- Jack
```

### Step 3: Present Draft for Approval

Show the drafted message to the user:

```markdown
## LinkedIn Connection Request

**To:** [Name] ([Title]) at [Company]
**Profile:** [LinkedIn URL]

**Draft Message:**
---
[Message content]
---

Character count: [X]/200

**Actions:**
- Type 'send' to post to Slack
- Type 'edit' to modify the message
- Type 'cancel' to abort
```

### Step 4: Handle User Response

**If 'edit':**
Ask the user what they'd like to change, then redraft and present again.

**If 'cancel':**
Acknowledge and exit gracefully.

**If 'send':**
Proceed to Slack posting (Step 5).

### Step 5: Post to Slack for Manual Execution

After user approves the draft, post to `#linkt-connections` (channel ID: `C0ACE4HJLBD`).

Use `mcp__slack__slack_post_message` with `channel_id: "C0ACE4HJLBD"`.

**Message format:**
```
📬 *LinkedIn Connection Request*

*To:* [Name] ([Title]) at [Company]
*Profile:* [LinkedIn URL]
*Signal:* [Signal summary]

*Draft Message (200 char max):*
> [Message content]

_Copy the message above and send via LinkedIn._
```

### Step 6: Confirmation

After posting to Slack:

```markdown
**Posted to Slack!**

- Channel: #linkt-connections
- Contact: [Name] at [Company]
- Signal: [Signal type]

**Next steps:**
1. Open the LinkedIn profile: [URL]
2. Click "Connect" → "Add a note"
3. Paste the message from Slack
4. Send the connection request

**Tip:** Follow up in 3-5 days if they accept the connection.
```

## Safety Features

1. **Always show draft before posting** - User must approve the message
2. **Human executes the action** - Claude posts to Slack, human sends on LinkedIn
3. **Persistent record** - Slack provides audit trail of outreach
4. **No automation complexity** - Avoids browser automation issues

## Error Handling

- **Slack not configured:** Provide manual instructions and suggest running `docs/slack-setup.md`
- **LinkedIn URL invalid:** Ask user to verify the URL
- **Channel not found:** List available channels and ask user to choose

## Character Limits

- LinkedIn connection note: **200 characters max**
- Keep messages under 180 characters to be safe
- If message is too long, offer to trim it

## Do NOT

- Post to Slack without user approval of the draft
- Use generic/template messages without personalization
- Mention specific product pitches in connection requests
- Skip the confirmation step
