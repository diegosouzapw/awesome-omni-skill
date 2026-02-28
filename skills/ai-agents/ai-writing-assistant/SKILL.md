---
name: ai-writing-assistant
description: Leverage AI tools effectively for professional writing tasks. Provides prompt patterns, refinement workflows, voice preservation techniques, and quality checkpoints for AI-augmented drafting.
allowed-tools: Read, Glob, Grep
---

# AI Writing Assistant Skill

Use AI tools strategically to enhance your professional writing while maintaining authenticity and quality.

## Keywords

ai, chatgpt, copilot, grammarly, prompt, draft, writing assistant, voice, tone, refinement, editing, automation

## When to Use This Skill

This skill provides guidance when developers need to:

- Draft professional content with AI assistance
- Refine and improve existing drafts using AI
- Maintain authentic voice while using AI tools
- Choose appropriate AI workflows for different writing tasks
- Verify and polish AI-generated content
- Collaborate between AI drafts and human editing

## Core Philosophy

**AI as collaborator, not replacement.**

AI writing tools work best when you:

1. **Provide clear direction** - Good prompts produce good drafts
2. **Iterate on output** - First draft is never final
3. **Inject your voice** - Add personality and expertise
4. **Verify everything** - AI makes confident mistakes
5. **Know when NOT to use AI** - Some tasks need pure human judgment

## The Two Workflows

### Workflow 1: AI-First Drafting

**Use when:** Starting from blank, need structure, time-pressured

```text
┌─────────────────────────────────────────┐
│  1. PROMPT                              │
│  Provide context, constraints, examples │
├─────────────────────────────────────────┤
│  2. GENERATE                            │
│  Let AI produce initial draft           │
├─────────────────────────────────────────┤
│  3. EVALUATE                            │
│  Check accuracy, tone, completeness     │
├─────────────────────────────────────────┤
│  4. REFINE                              │
│  Add voice, fix errors, enhance         │
├─────────────────────────────────────────┤
│  5. VERIFY                              │
│  Final quality check before sending     │
└─────────────────────────────────────────┘
```

**Best for:**

- First drafts of emails
- Outline generation
- Template creation
- Brainstorming options
- Summarizing content

### Workflow 2: Human-First Refinement

**Use when:** You have ideas, need polish or alternatives

```text
┌─────────────────────────────────────────┐
│  1. DRAFT                               │
│  Write your rough version first         │
├─────────────────────────────────────────┤
│  2. ASK                                 │
│  Request specific improvements          │
├─────────────────────────────────────────┤
│  3. COMPARE                             │
│  Evaluate AI suggestions vs original    │
├─────────────────────────────────────────┤
│  4. MERGE                               │
│  Combine best of both versions          │
├─────────────────────────────────────────┤
│  5. VERIFY                              │
│  Final quality check                    │
└─────────────────────────────────────────┘
```

**Best for:**

- Improving clarity
- Finding better phrasing
- Expanding or condensing
- Tone adjustment
- Grammar and style fixes

## Prompt Patterns by Task

### Email Prompts

**Basic email draft:**

```text
Write a professional email to [recipient role] about [topic].

Context:
- [Relationship/history]
- [Goal of this email]
- [Any constraints]

Tone: [professional/friendly/urgent/casual]
Length: [brief/detailed]
```

**Email refinement:**

```text
Improve this email for clarity and professional tone:

[Paste your draft]

Keep the core message but:
- Make it more concise
- Fix any awkward phrasing
- Ensure call-to-action is clear
```

### Feedback Prompts

**Structuring feedback:**

```text
Help me structure feedback for a team member using the SBI model.

Situation: [When and where]
Behavior: [What I observed - facts only]
Impact: [Effect on team/project/outcomes]

Make it constructive and specific. Avoid vague language.
```

**Preparing for difficult conversation:**

```text
I need to have a difficult conversation about [topic].

Context: [Background]
My concern: [What I want to address]
Desired outcome: [What I hope to achieve]

Help me:
1. Plan my opening statement
2. Anticipate possible responses
3. Prepare redirects if conversation derails
```

### Presentation Prompts

**Outline generation:**

```text
Create an outline for a [duration] technical presentation on [topic].

Audience: [Who they are, their level]
Goal: [What should they learn/do after]
Key message: [Main takeaway]

Use the What-Why-How structure:
- What: Hook and problem statement
- Why: Why this matters to them
- How: The solution/approach
- Close: Key takeaways and CTA
```

**Slide content:**

```text
Create slide content for: [Section topic]

Requirements:
- Title: Action-oriented, 5-8 words
- Bullets: 3-5 points, 5-7 words each
- One main idea only

This slide is about: [Specific point]
```

### LinkedIn/Content Prompts

**Post creation:**

```text
Create a LinkedIn post about [topic].

Frame: [learning/win/behind-the-build/contrarian/quick-tip]
Hook: Start with something that makes people click "see more"
Length: 150-250 words
End with: Question to drive engagement

My angle/experience: [What makes my perspective unique]
```

**Content repurposing:**

```text
Turn this [blog post/documentation/talk] into a LinkedIn post:

[Paste content]

Extract the most interesting insight and structure as:
1. Hook (attention-grabbing opening)
2. Story/context (brief setup)
3. Insight (the key learning)
4. CTA (question for engagement)
```

## Voice Preservation Techniques

### The Voice Sample

Before using AI for your content, create a voice sample:

```text
Analyze my writing style from these samples:

[Paste 3-5 paragraphs of your writing]

Identify:
1. Sentence structure patterns
2. Word choice tendencies
3. Tone and formality level
4. Unique phrases or expressions
5. How I open and close sections
```

Then use in subsequent prompts:

```text
Write in a style that matches this voice profile:
[Paste analyzed voice profile]

Now write: [Your request]
```

### The Injection Method

Add personal touches after AI drafting:

1. **Add your examples** - Replace generic examples with real ones
2. **Insert your phrases** - Add expressions you actually use
3. **Include specifics** - Add names, dates, details AI couldn't know
4. **Add opinions** - AI stays neutral; add your actual take
5. **Include humor** - If that's your style, add it yourself

### The Hybrid Paragraph

Write key paragraphs yourself, let AI fill in the rest:

```text
Complete this draft, matching the style of paragraphs I've written:

My paragraphs (keep exactly as-is):
[Your key paragraphs]

Fill in sections marked [AI DRAFT]:
1. [AI DRAFT - Introduction]
2. [Your paragraph]
3. [AI DRAFT - Transition]
4. [Your paragraph]
5. [AI DRAFT - Conclusion]
```

## Quality Checkpoints

### Checkpoint 1: Accuracy Check

**Before sending anything AI-assisted, verify:**

- [ ] All facts are correct (AI makes up statistics)
- [ ] Names and titles are accurate
- [ ] Technical details are right
- [ ] Links work (AI invents URLs)
- [ ] Quotes are real (AI fabricates quotes)
- [ ] Dates and numbers are correct

### Checkpoint 2: Tone Check

**Read aloud and ask:**

- [ ] Does this sound like me?
- [ ] Would I actually say this in person?
- [ ] Is the formality level appropriate?
- [ ] Are there phrases I'd never use?
- [ ] Does it feel authentic?

### Checkpoint 3: Content Check

- [ ] Is the main point clear?
- [ ] Is it the right length?
- [ ] Does it answer the recipient's actual question?
- [ ] Is the call-to-action obvious?
- [ ] Would I be proud to have my name on this?

### Checkpoint 4: Risk Check

**For sensitive content:**

- [ ] Could this be misunderstood?
- [ ] Is anything potentially offensive?
- [ ] Am I sharing anything I shouldn't?
- [ ] What if this were forwarded/screenshot?
- [ ] Should a human review this?

## When NOT to Use AI

### Tasks Requiring Human Judgment

- **Layoff or termination communications** - Too sensitive for AI
- **Legal or compliance content** - Needs expert review
- **Highly personal feedback** - Requires genuine human care
- **Crisis communications** - Stakes too high
- **Salary or promotion discussions** - Requires human touch
- **Content about specific people** - Risk of inaccuracy

### Tasks AI Struggles With

- **Nuanced cultural context** - AI may miss subtlety
- **Company-specific jargon** - AI doesn't know your abbreviations
- **Recent events** - Training cutoffs limit knowledge
- **Highly technical accuracy** - AI confidently makes mistakes
- **Sarcasm or subtle humor** - Often misses the mark

## Common AI Writing Mistakes

### Mistake 1: Over-Formality

**AI writes:** "I am writing to inform you that..."

**Fix:** Replace with natural phrasing: "Quick update on..."

### Mistake 2: Hedge Words

**AI writes:** "It might be potentially beneficial to perhaps consider..."

**Fix:** Be direct: "I recommend..."

### Mistake 3: Hollow Phrases

**AI writes:** "In today's fast-paced world..."

**Fix:** Delete or replace with specific context

### Mistake 4: Excessive Structure

**AI writes:** "Firstly... Secondly... Thirdly..."

**Fix:** Vary sentence structure, remove unnecessary markers

### Mistake 5: Generic Examples

**AI writes:** "For example, Company XYZ..."

**Fix:** Replace with real, specific examples

## Tool-Specific Tips

### ChatGPT/Claude

- Use system prompts to set context
- Iterate with follow-up requests
- Ask for multiple versions
- Request specific changes, not vague "improve"

### Grammarly

- Good for grammar, punctuation, clarity
- Be skeptical of tone suggestions
- Disable suggestions that change your voice
- Use as final polish, not primary drafting

### GitHub Copilot (for Documentation)

- Good for code comments and docstrings
- Verify technical accuracy carefully
- Adjust for your team's style
- Use for boilerplate, edit the substance

## References

For detailed guidance, see:

- `references/ai-prompt-patterns.md` - Comprehensive prompt templates by use case
- `references/voice-preservation.md` - Techniques for maintaining authenticity

## Related Commands

- `/soft-skills:draft-email` - Generate professional emails
- `/soft-skills:craft-linkedin-post` - Create engaging posts
- `/soft-skills:prepare-feedback` - Structure feedback using SBI

## Anti-Patterns to Avoid

- Sending AI output without reading it
- Using AI for highly sensitive content
- Trusting AI facts without verification
- Letting AI override your authentic voice
- Over-relying on AI for tasks you should learn
- Copy-pasting prompts without customization

## Version History

- **v1.0.0** (2025-12-26): Initial release

---

## Last Updated

**Date:** 2025-12-26
**Model:** claude-opus-4-5-20251101
