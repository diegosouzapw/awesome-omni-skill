---
name: Journal
description: Build a personal journaling practice with prompts, reflection, and pattern discovery.
metadata: {"clawdbot":{"emoji":"📔","os":["linux","darwin","win32"]}}
---

## Core Behavior
- User wants to write → provide space, optional prompts
- No pressure, no judgment → journaling should feel safe
- Surface patterns when asked → insights from past entries
- Create `~/journal/` as workspace

## Entry Flexibility
- Stream of consciousness welcome — no structure required
- Bullet points fine — not everything needs paragraphs
- Voice notes transcribed — capture thoughts while walking
- Short entries valid — "Today was hard" is enough

## File Structure
One file per entry: `2024-03-15.md`
- Date as filename — chronological, findable
- Optional: time of day if multiple entries
- Frontmatter optional: mood, tags, location

## When User Starts Writing
- Don't interrupt flow — capture first, reflect later
- Offer prompts only if asked or stuck
- Accept whatever format they give
- "Just write" is the goal

## Prompt Library (When Asked)
- "What's on your mind?"
- "What went well today?"
- "What would you do differently?"
- "What are you grateful for?"
- "What are you avoiding thinking about?"
- "If tomorrow goes perfectly, what happens?"
- Keep prompts in `prompts.md` for personalization

## End of Entry Options
- Save as is — most common
- Add tags for later searching
- Note mood: 1-5 or emoji
- "Continue later" flag for unfinished thoughts

## Weekly Review
- Offer to review the week
- Themes that emerged
- Mood patterns if tracked
- Wins and struggles
- "Anything to carry forward?"

## Monthly/Yearly Reflection
- What changed this month/year?
- Recurring themes or concerns
- Progress on long-term thoughts
- Reading old entries — often surprising

## Pattern Discovery
When asked "what have I been writing about?":
- Common themes across entries
- Mood trends if tracked
- Frequency of journaling
- Topics that appear then disappear

## What To Surface Proactively
- "It's been 5 days since you journaled" — only if they want nudges
- "A year ago you wrote about X" — memory resurfacing
- "This theme appeared 3 times this month" — pattern spotting
- Never share content without permission

## Progressive Enhancement
- Week 1: just write, any format
- Week 2: consistent file naming
- Month 2: add mood/tags if useful
- Month 3: weekly review practice
- Year 1: annual reflection

## Folder Structure
```
~/journal/
├── 2024/
│   ├── 2024-03-15.md
│   └── 2024-03-16.md
├── prompts.md
└── reflections/
    └── 2024-march-review.md
```

## Privacy and Security
- Local files only — no cloud unless user chooses
- Encryption option if sensitive
- No AI training on journal content — make this clear
- Backup encrypted if backing up at all

## Types of Journaling
- Daily log: what happened, how you felt
- Gratitude: what you're thankful for
- Morning pages: stream of consciousness on waking
- Evening reflection: review of the day
- Topic-specific: work, relationships, health
- Let user find their style

## What NOT To Suggest
- Complex templates before natural writing flows
- Mandatory daily journaling — guilt kills the practice
- Sharing entries anywhere
- Analysis before sufficient entries exist
- Fixing problems — sometimes just listening is enough

## Handling Difficult Entries
- Acknowledge without judgment
- Don't offer unsolicited advice
- "That sounds really hard" is often enough
- Suggest professional support if concerning patterns

## Integration Points
- Habits: "journal daily" as habit if wanted
- Mood tracking: simple scale alongside entries
- Goals: reflection on progress
- Contacts: processing relationship thoughts

## Searching Past Entries
- Full-text search across all entries
- Search by date range
- Search by mood if tracked
- Search by tag
- "What was I writing about last March?"
