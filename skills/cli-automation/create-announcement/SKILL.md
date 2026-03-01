---
name: create-announcement
description: This skill should be used when the user asks to "create an announcement", "write a Teams message for a PR", "write a Slack message for a PR", "announce PR changes", "generate a release announcement", "summarize PR for Teams", "summarize PR for Slack", "draft a channel post", or needs to turn GitHub pull request changes into a formatted announcement for IM platforms (MS Teams, Slack). Supports single-PR and multi-PR release announcements.
version: 1.1.0
---

# MS Teams Announcement from Pull Requests

## Overview

Generate professional Microsoft Teams channel announcements based on pull request changes. Analyze PR diffs, descriptions, and commit history using the GitHub MCP server, then produce a structured markdown message ready to paste into a Teams channel.

**Announcement structure:**
- **Title** - Emoji prefix + concise topic headline
- **Subtitle** - Context line with scope, version, or key qualifier
- **Message Body** - Opening context, bold section headers, bulleted details, links, and closing call to action

## Prerequisites

The GitHub MCP server must be available in the current session. Verify by checking available MCP tools at the start of the workflow. If the GitHub MCP server is not available, inform the user that it is required for fetching pull request data.

## Workflow

### Step 1: Gather PR Information

Determine whether the announcement covers a single PR or multiple PRs.

**For a single PR**, use the GitHub MCP server to fetch:
- PR title, description, and labels
- Changed files list and diff summary
- Commit messages

**For multiple PRs** (e.g., a release or migration), fetch the above for each PR and identify common themes, affected areas, and the overall scope of changes.

If the user provides PR numbers or URLs, use those directly. If not, ask the user to specify which PR(s) to announce.

### Step 2: Analyze Changes

Categorize the changes from the PR(s):

1. **What changed** - Upgrades, new features, deprecations, security patches, tooling additions
2. **What areas are affected** - Frameworks, libraries, developer workflows, build pipelines
3. **Why it matters** - Security, performance, developer experience, compliance
4. **What action is needed** - Verification, migration steps, version upgrades, nothing

Focus on impact and outcomes rather than implementation details. Translate technical diffs into language that communicates value to the target audience.

### Step 3: Compose the Announcement

Produce the announcement following the format and guidelines below.

#### Announcement Format

```markdown
Title: [emoji] [Topic]
Subtitle: [Context — Qualifier]

[Opening paragraph]

[Bold section headers with content]

[Links and closing]
```

#### Title Guidelines

- Begin with a relevant emoji that visually identifies the topic
- Follow with a concise topic (framework name, feature, tool)
- Keep under 60 characters (excluding emoji)
- Use version transition notation where applicable (e.g., "v18→19")

**Examples:**
- `🔒 Security Patch — Auth Library v3.2.1`
- `🚀 Welcome ECMAScript 2025`
- `⚛️ React 18→19`
- `⚠️ Drop Node.js 18 Support`
- `🤖 AI Assistant First-Class Support`
- `🤖 New Calude Code Skill: localize`

#### Subtitle Guidelines

- One line providing scope, version detail, or key qualifier
- Use em dash (—) to separate topic from qualifier
- Keep under 100 characters

**Examples:**
- `Critical Security Patch — March Release Target`
- `ES2025 is Here — New Language Features Now Available`
- `Node v22 is supported for local development and CI runners`

#### Message Body Guidelines

Structure the body with these elements as applicable:

1. **Opening paragraph** - One to three sentences establishing context: what happened, who did it, and when. Include the PR link inline.
2. **Bold section headers** - Use `**Header**` to organize content into logical sections. Common headers:
   - `**Notable Changes**` or `**What's New**` - for listing changes
   - `**Impact**` - for downstream effects
   - `**⚠️ IMPORTANT ⚠️**` - for critical action items or deprecations
   - `**What should I know**` - for linking to external docs or guides
3. **Bulleted details** - List specific changes, versions, or features under each section header
4. **Links** - Include PR URLs, changelog links, documentation links, and discussion links inline where relevant
5. **Closing** - End with a call to action, invitation for questions, or team contact line

**Writing style:**
- Professional but approachable — use "we" and team attribution
- Write in past tense for completed actions ("has been upgraded"), present tense for current state ("is now available")
- Include version numbers with transition notation where relevant (e.g., "5.5.4 -> 5.8.3")
- Bold key dates, version numbers, and action items for scannability
- Link to external resources (changelogs, official docs, guides) rather than repeating their content
- Keep total body under 250 words for single-PR announcements, under 400 words for multi-PR

### Step 4: Present and Refine

Present the composed announcement to the user in a copyable markdown block. Offer to adjust tone, length, or detail level if needed.

## Announcement Categories

Different change types call for different emphasis:

- **Framework/library upgrades** - Lead with version transition, list notable dependency changes with changelog links, mention release timeline
- **Security patches** - Lead with urgency, reference CVEs or advisories, specify release target
- **Deprecation notices** - Lead with the deadline, bold the deprecation date, provide migration steps
- **New tooling/features** - Lead with the value proposition, describe setup or usage, invite the team to try it
- **Documentation** - Lead with what's covered, link directly to the resource, invite contributions

## Multi-PR Announcements

When summarizing multiple PRs into a single announcement:

- Group changes by theme or affected area rather than listing per-PR
- Identify the most significant changes and lead with those
- Mention the total number of PRs and provide links to all of them
- Use a milestone or release-style title when appropriate

## Tone Calibration

Default to a professional, approachable tone suitable for engineering channels — the kind of message a tech lead would post. If the user specifies a different audience, adjust accordingly:

- **Engineering channel** - Include version numbers, dependency names, changelog links, CLI commands
- **Broader audience** - Focus on user-facing impact, avoid internal tooling details
- **Leadership** - Emphasize business value, timelines, and risk mitigation

## Output Expectations

Always output the final announcement inside a single fenced code block (```markdown) so the user can easily copy and paste it into Teams. Teams supports markdown formatting including bold, italic, bulleted lists, and links.

## Additional Resources

### Reference Files

For detailed examples of announcements across different categories:
- **`references/announcement-examples.md`** - Full example announcements covering upgrades, security patches, deprecations, new tooling, and documentation releases

### Example Files

- **`examples/single-pr-announcement.md`** - Example output for a framework upgrade announcement
- **`examples/multi-pr-announcement.md`** - Example output for a multi-topic release announcement
