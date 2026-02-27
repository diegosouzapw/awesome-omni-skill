---
name: capture-content
description: Journalist interview that extracts publishable content from your work through structured questions
---

# /capture-content - Journalist Interview for Content Creation

Claude Code skill that extracts publishable content from your work through structured interview.

## Purpose

You've just done interesting work but don't know how to talk about it yet. This skill acts as a journalist interviewing you to extract:
- What you built/discovered/solved
- Why it matters
- What to screenshot or video record
- How to frame it for your audience

## When to Use

**Session-based triggers**:
- Solved a significant problem
- Built or updated a system
- Discovered an interesting tool/workflow
- Had a meaningful realization
- Completed interesting analysis or synthesis

**Synthesis-based triggers**:
- Identified connections across multiple frameworks
- Recognized patterns from bookmark clusters
- Developed new mental model from expert synthesis

## Interview Process

The skill will ask 8-12 strategic questions designed to extract content:

### Phase 1: What Happened (3-4 questions)
- What were you working on when [interesting thing] happened?
- What specific tools/approaches did you use?
- What was the problem you were trying to solve?
- Walk me through the key moment - what exactly did you do?

### Phase 2: Why It Matters (2-3 questions)
- What surprised you about this?
- Why is this different from how it usually works?
- What does this say about [broader trend/pattern]?
- Who else struggles with this problem?

### Phase 3: Content Strategy (2-3 questions)
- Is this a "behind-the-scenes reveal" or "framework teaching" moment?
- What's the single most important insight here?
- What would make someone else care about this?
- What writing style cluster fits this best? (Technical Observer, Enthusiastic Builder, Framework Teacher)

### Phase 4: Visual Capture (1-2 questions)
- What should be screenshotted to show this?
- Would a 30-second screen recording add value?
- What's the visual "proof" that this worked?

## Output Format Options

After interview, the skill offers to generate:

1. **Twitter/X Thread** (7-12 tweets)
   - Hook using identified style cluster
   - Process breakdown with specific tools
   - Key insight delivery
   - Call to action or implication

2. **LinkedIn Post** (1,800-2,100 characters)
   - Concrete opening (what happened)
   - Process breakdown
   - Insight delivery (understated but profound)
   - Broader implication

3. **Quote-Tweet Addition** (for synthesis content)
   - Technical Observer commentary on bookmarked tool/framework
   - Your implementation experience
   - What you discovered that others miss

4. **Screenshot/Video Script**
   - What to capture visually
   - How to annotate
   - What to highlight

## Style Integration

The interview automatically applies your writing style:
- **Concrete Specificity**: Extracts exact tools, timeframes, examples
- **Observational Authority**: Frames as "here's what I witnessed"
- **Technical Accessibility**: Helps you explain without jargon
- **Confident Understatement**: Natural, authentic voice

Blends appropriate clusters based on content type:
- **Technical Observer**: Default for most content
- **Enthusiastic Builder**: When something surprisingly worked
- **Framework Teacher**: For synthesis/system content
- **Behind-the-Scenes**: For workflow reveals

## Example Interview Flow

```
Assistant: I noticed this session involved [interesting work]. Let's capture this as content.

**Phase 1: What Happened**

1. What were you trying to accomplish when you started?
User: [Answer]

2. What specific tools did you use? (Names, not "AI tools")
User: [Answer]

3. Walk me through the key moment - what exactly happened?
User: [Answer]

**Phase 2: Why It Matters**

4. What surprised you about this?
User: [Answer]

5. Why is this different from the usual way this works?
User: [Answer]

6. Who else struggles with this problem?
User: [Answer]

**Phase 3: Content Strategy**

7. What's the single most important insight from this?
User: [Answer]

8. Is this more "here's what I built" or "here's what I learned"?
User: [Answer]

9. Which style fits: Technical Observer (nuanced analysis), Enthusiastic Builder (it actually worked!), or Framework Teacher (here's the system)?
User: [Answer]

**Phase 4: Visual Capture**

10. What should I screenshot to prove this worked?
User: [Answer]

11. Would a 30-second screen recording add value? What would it show?
User: [Answer]

**Draft Generation**

Based on your answers, I'll draft:
- [Twitter thread / LinkedIn post / Quote-tweet addition]
- Screenshot/video capture instructions
- Posting guidance (timing, hashtags, mentions)
```

## Usage

```bash
# Manual invocation
/capture-content

# With context hint
/capture-content "just solved interesting problem"

# For synthesis
/capture-content synthesis

# Quick mode (fewer questions)
/capture-content quick
```

## Integration with Existing Strategy

**Resolves these strategic tensions**:
- **Evaluation vs. Publication**: Captures tool evaluations in-the-moment
- **Implementation Mode**: Extracts insights while doing the work
- **Detailed Breakdowns vs. Execution**: Reduces steps through guided interview
- **Hook Effectiveness**: Automatically applies proven patterns

**Feeds into**:
- Quote-tweet strategy (generates substantive additions)
- Framework synthesis (documents thinking for later synthesis)
- Thought leadership positioning (consistent content flow)

## Technical Implementation

The skill uses AskUserQuestion tool to conduct interview:
- Questions adapt based on previous answers
- Content type detected from responses
- Style cluster selected based on content nature
- Visual guidance tailored to platform

## Success Metrics

Track over time:
- Capture rate: % of interesting sessions that become content
- Publication rate: % of captured content that gets published
- Engagement: Which interview-generated content performs best
- Efficiency: Time from "interesting work" to "published post"

## Continuous Improvement

The system learns from:
- Which questions extract the best content
- Which style clusters drive engagement
- What visual formats work best
- When you skip questions (signals friction)

---

**Installation**: Save as `~/.claude/skills/capture-content/skill.md`