---
name: article-writer
description: Edit and refine blog articles and technical posts to match a specific writing style. Use when editing markdown articles, blog posts, or technical writing. Applies conversational, direct, no-BS style with strict guidelines (no em dashes, no empty framing phrases). Best for technical content aimed at experienced developers.
---

# Article Writer

## Overview

Edit blog articles and technical posts to match a specific writing style: conversational, direct, opinionated, and technically precise. Apply strict style guidelines including banned patterns (em dashes, empty framing phrases) and enforce consistent voice, structure, and formatting.

## When to Use This Skill

Activate this skill when:
- Editing markdown articles or blog posts
- Refining technical writing for publication
- Reviewing drafts for style consistency
- Converting formal writing to conversational style
- Checking articles for guideline violations

## Editing Workflow

Follow this workflow when editing articles:

### 1. Initial Assessment

Read the article draft completely to understand:
- Main topic and key points
- Target audience and technical level
- Current writing style and tone
- Overall structure and flow

### 2. Run Automated Style Check

Before manual editing, run the style checker to identify violations:

```bash
python3 scripts/check_style.py <article-file.md>
```

The checker identifies:
- Em dashes (— or –)
- Empty framing phrases ("Here's the thing:", "But here's the reality:", etc.)
- Excessive hedging words
- Overly long paragraphs (>6 sentences)

Review the automated findings and keep them in mind during editing.

### 3. Apply Core Guidelines

Reference `references/writing-guidelines.md` for banned patterns:

**Eliminate em dashes entirely:**
- Replace with commas, periods, colons, or parentheses
- Example: "The feature—which took weeks—finally shipped" → "The feature, which took weeks, finally shipped"

**Remove empty framing phrases:**
- Cut discourse markers that announce rather than state
- Example: "Here's the thing: your costs are too high" → "Your costs are too high"
- Common culprits: "Here's the reality:", "The point:", "At the end of the day:"

### 4. Match the Style Guide

Reference `references/style-guide.md` for comprehensive style patterns. Focus on these key areas:

**Voice & Tone:**
- Conversational and direct (like talking to a peer)
- Confident and opinionated (no hedging)
- Second person ("you") for direct address
- Strategic profanity for emphasis (use sparingly)
- Contractions throughout (it's, you're, don't)

**Structure:**
- Short paragraphs (2-5 sentences typically)
- One idea per paragraph
- Headers in sentence case, mostly lowercase
- Horizontal rules (---) for major section breaks only
- Frequent bullet points and numbered lists

**Emphasis:**
- Bold for key terms and strong emphasis
- Italics sparingly
- Inline code with backticks for technical terms
- Code blocks with language specification

**Content Patterns:**
- Strong opening hook (assertion, question, surprising fact)
- Concrete examples over abstract concepts
- Specific numbers and data ("20% improvement" not "significant")
- Address counterarguments directly
- Actionable advice with clear explanations

**Sentence Structure:**
- Vary sentence length for rhythm
- Mix short punchy sentences with longer explanatory ones
- Active voice preferred
- Specific over vague word choice

### 5. Check Against Example Article

Reference `references/example-article.md` to see the style in practice. Use it to:
- Verify tone matches (direct, confident, conversational)
- Check section structure (headers, paragraphs, lists)
- Confirm opening and closing patterns
- Validate technical depth and accessibility balance

### 6. Final Review Checklist

Before completing edits, verify:
- [ ] No em dashes anywhere
- [ ] No empty framing phrases
- [ ] Paragraphs are 2-5 sentences (with intentional exceptions)
- [ ] Headers are descriptive and sentence case
- [ ] Mix of sentence lengths creates good rhythm
- [ ] Technical terms accurate and properly formatted
- [ ] Strong opening hook present
- [ ] Concrete examples throughout
- [ ] Conversational but authoritative tone
- [ ] Bold/italics used for emphasis, not decoration

### 7. Provide Edit Summary

After editing, summarize changes made:
- List major structural changes
- Note pattern fixes (em dashes removed, framing phrases cut, etc.)
- Highlight tone adjustments
- Mention any sections that needed significant rewriting
- Flag any areas where style guide couldn't be fully applied (explain why)

## Editing Principles

### Progressive Enhancement

Edit in passes rather than trying to fix everything at once:
1. First pass: Structure and flow
2. Second pass: Voice and tone
3. Third pass: Guidelines compliance (em dashes, framing phrases)
4. Fourth pass: Polish and rhythm

### Preserve Author's Voice

While enforcing style guidelines:
- Maintain the author's key points and arguments
- Keep technical accuracy intact
- Preserve intentional emphasis
- Don't over-homogenize; some variation is natural

### Explain Significant Changes

When making major edits:
- Note why the change improves the piece
- Reference specific guideline violations
- Suggest alternatives if the change is debatable
- Explain trade-offs when style conflicts with clarity

### Balance Rules with Readability

Guidelines are strong defaults, not absolute laws:
- Occasionally break rules for clarity or impact
- Use judgment on sentence and paragraph length
- Allow strategic rule-breaking when it serves the writing
- Explain intentional guideline deviations

## Common Editing Patterns

### Removing Framing Phrases

❌ "Here's the thing: AI is changing development"
✅ "AI is changing development"

❌ "The point is that you need to adapt"
✅ "You need to adapt"

### Replacing Em Dashes

❌ "The feature—despite taking weeks—finally shipped"
✅ "The feature, despite taking weeks, finally shipped"
✅ "The feature (despite taking weeks) finally shipped"
✅ "The feature finally shipped. It took weeks."

### Shortening Paragraphs

When a paragraph exceeds 5-6 sentences, split it:
- Find natural break points (topic shifts)
- Create two focused paragraphs
- Ensure each has one main idea
- Add transitions if needed

### Strengthening Voice

Weak: "It might be possible that this approach could work"
Strong: "This approach works"

Weak: "One should consider using tests"
Strong: "Use tests"

Weak: "There are many benefits to AI coding"
Strong: "AI coding speeds up development, catches bugs earlier, and lets you ship faster"

### Adding Concreteness

Vague: "The performance improved significantly"
Concrete: "Latency dropped from 500ms to 50ms"

Vague: "Many developers are adopting AI tools"
Concrete: "GitHub's data shows 92% of developers now use AI coding tools"

## Resources

### references/

**writing-guidelines.md** - Core banned patterns and why they're problematic:
- Empty framing phrases (comprehensive list)
- Em dashes and alternatives
- Concise explanations of each rule

**style-guide.md** - Complete style specification covering:
- Voice & tone (conversational, direct, opinionated)
- Structure & formatting (paragraphs, headers, lists)
- Content patterns (openings, examples, actionable advice)
- Writing mechanics (sentence structure, word choice, rhythm)
- Final checklist for publishing

**example-article.md** - Full-length article demonstrating the style:
- Technical depth (AI coding guide)
- Target audience (experienced developers)
- Proper use of all style elements
- Real-world example of the guidelines in practice

Load these references when editing to ensure consistency with established patterns.

### scripts/

**check_style.py** - Automated style violation detector:
- Scans markdown files for em dashes and framing phrases
- Identifies excessive hedging and long paragraphs
- Provides specific suggestions for fixes
- Colored terminal output for easy scanning

Run before manual editing to catch mechanical violations quickly. Use the output to guide editing priorities.

Usage:
```bash
# Single file
python3 scripts/check_style.py article.md

# Multiple files
python3 scripts/check_style.py article1.md article2.md

# Entire directory
python3 scripts/check_style.py ./articles/
```

## Tips for Best Results

### When Editing Lightly

For minor edits (article mostly matches style):
1. Run check_style.py first
2. Fix flagged violations
3. Quick read for tone and flow
4. Verify checklist items

### When Editing Heavily

For major rewrites (article needs significant work):
1. Read full article to understand intent
2. Outline key points to preserve
3. Rewrite section by section
4. Reference style-guide.md frequently
5. Compare against example-article.md for tone
6. Run check_style.py at end
7. Final polish pass

### When Unsure

If style application is ambiguous:
- Prioritize clarity over strict adherence
- Explain the reasoning for the choice
- Offer alternative phrasings
- Ask the user which approach they prefer

### Continuous Improvement

After editing multiple articles:
- Note recurring issues or patterns
- Suggest additions to style guide if needed
- Identify areas where guidelines conflict
- Recommend script enhancements for new patterns
