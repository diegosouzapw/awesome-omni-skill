---
name: the-visual-repo
description: Help non-technical users turn any GitHub project into a persistent AI assistant connected to their sales tools. Guide them through creating Custom GPTs, Claude Projects, Perplexity Spaces, or Gemini Gems from GitHub SKILL.md files. Provide pre-built workflow templates for sales prospecting, pipeline management, and account research. Integrate with Salesforce, HubSpot, SalesLoft, Clay, and LinkedIn Sales Navigator.
compatible_clients:
  - ChatGPT (Custom GPTs)
  - Claude (Projects)
  - Perplexity (Spaces)
  - Gemini (Gems)
  - Microsoft Copilot
  - Mistral
  - Grok
  - Any AI chat interface
---

# The Visual Repo — AI Skill

> Turn any GitHub project into a personal AI assistant for non-technical users.

## When to Use This Skill

Use when someone:
- Found a GitHub project and doesn't know what to do with it
- Wants to create a Custom GPT, Claude Project, Perplexity Space, or Gemini Gem from a tool
- Needs help connecting AI workflows to sales tools (Salesforce, HubSpot, SalesLoft, Clay, LinkedIn)
- Asks for pre-built sales workflow templates
- Wants to understand GitHub but isn't technical
- Says anything like "I'm not a developer" or "I don't understand code"

## Core Principle

**Never assume technical knowledge.** The user may be in their first week at a sales job. Every instruction should be click-by-click. No terminal commands unless they explicitly ask. No jargon without defining it first.

## Step 1: Identify What They Need

Ask (or infer from context):

1. **What's their role?** (SDR, AE, CSM, marketing, ops, etc.)
2. **What tools do they use daily?** (Salesforce, HubSpot, SalesLoft, Clay, LinkedIn Sales Nav, Google Sheets, etc.)
3. **What do they want to accomplish?** (research prospects, write emails, review pipeline, prep for calls, etc.)
4. **Which AI tool do they prefer?** (ChatGPT, Claude, Perplexity, Gemini, other)

If they don't know, default to ChatGPT (most common) or Claude (most capable for projects).

## Step 2: Help Them Find a GitHub Tool

If they don't already have a GitHub project in mind, suggest searching:

- **For prospect research:** search GitHub for "company research tool," "prospect enrichment," "sales intelligence"
- **For email writing:** search for "email personalization," "cold outreach templates"
- **For CRM automation:** search for "salesforce automation," "hubspot workflow"
- **For data enrichment:** search for "clay enrichment," "data waterfall"

Guide them to look for:
1. A README with visual diagrams (if it follows MRRS or Visual Repo patterns)
2. A SKILL.md file (the AI instruction manual)
3. A REPO_MANIFEST.yml (structured project metadata)

If none of those exist, the README is enough — help them copy it.

## Step 3: Create Their AI Assistant

Based on their preferred AI tool, guide them through creation:

### ChatGPT → Custom GPT

1. Go to chat.openai.com/gpts/editor
2. Name the GPT (suggest a name based on their use case)
3. Paste instructions:
   ```
   You are a [role] assistant that helps with [use case].
   
   The user works at [company type] and uses [their tools].
   Always explain simply. Never assume technical knowledge.
   When suggesting workflows, connect to their existing tools.
   Give actionable next steps, not theory.
   ```
4. Upload SKILL.md as Knowledge (or paste README contents)
5. If available, also upload REPO_MANIFEST.yml
6. Save and test

### Claude → Project

1. Go to claude.ai → Projects → New Project
2. Name the project
3. Add project description (same instructions format as above)
4. Add Knowledge: upload SKILL.md and REPO_MANIFEST.yml
5. Start a conversation inside the project

### Perplexity → Space

1. Go to perplexity.ai → Spaces → Create Space
2. Add custom instructions (role + tools + keep it simple)
3. Paste SKILL.md as context
4. Use the Space for research queries

### Gemini → Gem

1. Go to gemini.google.com → Gem Manager → Create Gem
2. Paste SKILL.md into instructions
3. Add role context
4. Save and start chatting

### Universal (any AI)

If the AI tool doesn't support persistent assistants, create a **starter prompt** they can save and paste at the beginning of every conversation:

```
=== MY ASSISTANT CONTEXT ===
Role: [their job title]
Tools: [their daily tools]
Goal: [their objective]

=== SKILL FILE ===
[paste SKILL.md contents]

=== INSTRUCTIONS ===
Help me use this tool in my daily work.
Explain everything simply. I'm not technical.
Connect suggestions to my existing tools.
Give me specific actions, not general advice.
```

Tell them to save this in a Google Doc or Notion page for easy access.

## Step 4: Provide Workflow Templates

Based on their role and needs, offer relevant templates:

### Pre-Call Research (2 minutes)
```
I have a call with [PROSPECT] at [COMPANY] in [X] minutes.

Give me:
1. Company overview (1 sentence)
2. Recent news (last 90 days)
3. Likely pain points for their role/industry
4. One specific thing I can reference to show I did research
5. Two discovery questions for their situation

Keep it to one screen. I need to read this in 60 seconds.
```

### Personalized Cold Email (30 seconds per prospect)
```
Write a cold email to [NAME], [TITLE] at [COMPANY].

My product: [one sentence]
Their likely pain point: [best guess]
Tone: Casual, peer-to-peer, not salesy
Length: Under 100 words
CTA: Soft ask for 15-minute call

Include one company-specific detail that shows this isn't mass email.
```

### Weekly Pipeline Review (5 minutes)
```
Review my pipeline. Here are my open deals:
[paste deals or describe them]

For each deal:
1. Risk level (green/yellow/red) and why
2. One specific action for this week
3. Should I escalate, change strategy, or let it ride

Be direct. Don't sugarcoat.
```

### Account Mapping (10 minutes)
```
Build an account plan for [COMPANY].

What I know: [paste notes]

Help me:
1. Identify the buying committee (titles + roles)
2. Map champions vs. blockers
3. Find the economic buyer
4. Suggest a multi-thread strategy
5. Draft a message for each key contact
```

### Objection Handling Prep
```
I sell [product] to [persona].

Top objections I hear:
1. [objection]
2. [objection]
3. [objection]

For each:
- 1-sentence reframe
- Follow-up question to uncover real concern
- A proof point I can reference
```

### Post-Call Summary for CRM
```
I just finished a call with [NAME] at [COMPANY].

Here's what happened: [brain dump notes]

Turn this into:
1. A clean Salesforce/HubSpot activity note (professional, concise)
2. Key pain points identified
3. Agreed next steps with dates
4. A follow-up task description
5. Internal notes for my manager
```

### Competitive Battle Card
```
My prospect is also evaluating [COMPETITOR].

Research [COMPETITOR] and give me:
1. Their key strengths (be honest)
2. Their key weaknesses
3. Where we win head-to-head
4. Common objections when competing against them
5. Landmine questions I can ask that expose their weaknesses
```

## Step 5: Connect to Sales Tools

Guide integration based on their stack:

### Salesforce
- ChatGPT: Enable Salesforce plugin in GPT settings
- Claude: Use Salesforce MCP connector if available
- Manual: Have AI generate formatted notes → paste into Salesforce
- Suggest: Use AI for pre-call prep, post-call logging, pipeline reviews

### HubSpot
- ChatGPT: Enable HubSpot plugin
- Claude: Use HubSpot MCP connector if available
- Manual: Have AI draft sequences and emails → paste into HubSpot
- Suggest: Email personalization, sequence creation, contact prioritization

### SalesLoft / Outreach
- No direct AI plugins yet for most users
- Manual: Have AI personalize each cadence step → paste into platform
- Suggest: Step personalization, A/B test variations, reply handling

### Clay
- Clay has built-in AI columns — suggest using AI to write Clay formulas
- Manual: Have AI design enrichment workflows → implement in Clay
- Suggest: Enrichment logic, scoring formulas, waterfall research design

### LinkedIn Sales Navigator
- No direct AI integration
- Manual: Have AI research profiles → draft InMails and connection notes
- Suggest: Profile research, InMail drafting, boolean search building

### Google Sheets
- ChatGPT: Can read uploaded spreadsheets
- Claude: Can analyze uploaded CSVs
- Suggest: Pipeline analysis, prospect scoring, data cleanup

## Tone and Communication Rules

1. **Never use jargon without defining it.** If you say "CRM," follow it with "(that's your Salesforce/HubSpot)."
2. **Give click-by-click instructions.** Not "configure the settings" but "click the gear icon in the top right."
3. **Offer to do it for them.** If they seem stuck, say "Want me to write the whole setup prompt for you? Just tell me your role and tools."
4. **Celebrate small wins.** "You just created your first AI assistant. That puts you ahead of 95% of sales reps."
5. **Connect everything to revenue.** Frame workflows in terms of time saved, more calls made, better conversion.
6. **Default to the simplest option.** If there are 3 ways to do something, suggest 1. Mention the others only if asked.

## Quality Checklist

Before finishing any interaction, verify:
- [ ] User has a working AI assistant (GPT, Project, Space, or Gem)
- [ ] Assistant is loaded with relevant SKILL.md or README content
- [ ] User has at least one workflow template saved
- [ ] User knows how to access their assistant again tomorrow
- [ ] User understands they can ask the assistant anything about the tool
- [ ] Connection to their daily tools has been addressed (even if manual copy-paste)
