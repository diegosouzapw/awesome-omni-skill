---
name: blog-experiential
description: Write experiential articles for marvinzhang.dev following 3-stage workflow (Outline → Writing → Refine). Use for personal insights, lessons learned, project retrospectives, and reflective pieces. Produces narrative-driven bilingual content with honest, practical takeaways. Requires blog-common skill.
metadata:
  author: marvinzhang
  version: "1.0"
  workflow: "3-stage"
  style: "experiential"
---

# Blog Writing: Experiential Style

Write personal, reflective articles that share lessons learned and practical insights from real experiences.

## Prerequisites

**Load blog-common skill** for formatting, localization, quality standards, and writing principles.

## When to Use

- Lessons learned from projects or experiences
- Personal insights and reflections
- Project retrospectives and case studies
- Career reflections and growth stories
- Problem-solving journeys

**Example topics**: "What I Learned from 5 Years in Open Source", "Building Crawlab: A Technical Retrospective", "My Journey from Vibe Coding to SDD"

## Questionnaire-First Workflow

**Start every experiential article with the questionnaire** to gather structured input:

```bash
lean-spec create "article-slug" --template=experiential
```

This creates a spec with embedded questionnaire covering:
- Context (project, timeline, role, scale)
- The challenge and what made it difficult
- Your approach and pivots
- Lessons learned (3-5 key takeaways)
- Advice for others

**Fill out the questionnaire**, then tell AI "questionnaire complete". This helps capture the full story before writing begins.

[See questionnaire guide in references/questionnaire.md]

## Workflow: 3-Stage Process

### Stage 1: Outline (1 interaction)

**Objective**: Create narrative arc and identify key lessons

**Activities**:
1. **Story structure**: Context → Challenge → Approach → Outcome → Lessons
2. **Key moments**: Pivotal decisions, turning points, realizations
3. **Lesson extraction**: Actionable insights from experience
4. **Tone setting**: Balance humility and authority

**Structure Template**:
```markdown
## Introduction (300-500 words)
- Hook: Compelling moment or question
- Context: What you were trying to do
- Why it matters: Relevance to readers
- Roadmap: Journey preview

## The Context (400-600 words)
- Situation and background
- Initial approach/assumptions
- What you expected to happen

## The Challenge (500-700 words)
- What went wrong or surprised you
- Key turning points
- Decisions made and why

## The Approach (500-700 words)
- How you tackled it
- What worked and what didn't
- Iterations and adaptations

## The Outcome (400-600 words)
- Final result
- Unexpected learnings
- What you'd do differently

## Lessons & Takeaways (400-600 words)
- 3-5 concrete lessons
- Actionable advice for readers
- Broader implications
```

**Deliverables**:
- Complete `specs/{spec-number}-{slug}/outline.md`
- Narrative arc defined
- Key lessons identified
- Honest tone established

**Quality Gates**:
- [ ] Clear narrative arc (context → challenge → resolution)
- [ ] Specific stories, not vague generalities
- [ ] Lessons are actionable and concrete
- [ ] Tone is humble and honest

### Stage 2: Writing (3-5 interactions)

**Objective**: Create engaging narrative with practical insights

**Approach**: **Write ONE section per interaction** directly to final files:
- English: `blog/YYYY-MM-DD-slug.mdx` (with `unlisted: true`)
- Track progress in spec folder

**Section Standards**:
- **Specific stories**: Real examples with details, not abstractions
- **Honest reflection**: Include failures and mistakes
- **Visual elements**: Use diagrams for process, tables for comparisons
- **Actionable lessons**: Readers can apply to their own situations
- **Balance**: Technical depth + personal reflection

**Section Completion Checklist**:
- [ ] Specific examples with details
- [ ] Honest about challenges and failures
- [ ] Visual element where helpful (diagram/table)
- [ ] Lessons clearly articulated
- [ ] Actionable takeaways
- [ ] Natural story flow

[See complete writing guidelines in blog-common/SKILL.md]

### Stage 3: Refine (1 interaction)

**Objective**: Polish narrative and finalize

**Activities**:
1. **Story review**: Ensure narrative cohesion and flow
2. **Lesson validation**: Ensure insights are clear and actionable
3. **Tone check**: Balance humility with authority
4. **Chinese translation**: Apply 形不同而意同 principle
5. **Final validation**: Build, format, links

**Quality Assessment**:
- [ ] Narrative flows naturally
- [ ] Lessons are concrete and actionable
- [ ] Honest about failures/mistakes
- [ ] Balance of story and technical depth
- [ ] Tone is humble but authoritative
- [ ] Chinese reads naturally
- [ ] Build succeeds: `pnpm run build`

**Publication Files**:
- English: `blog/YYYY-MM-DD-slug.mdx` (remove `unlisted: true`)
- Chinese: `i18n/zh/docusaurus-plugin-content-blog/YYYY-MM-DD-slug.mdx`

## Experiential-Specific Principles

### Specificity Over Generality
Share concrete stories, not platitudes:
```markdown
❌ "Communication is important in teams"
✅ "When our team missed a critical deadline because I assumed others understood 
   the database migration requirements, I learned to over-communicate technical 
   dependencies"
```

### Honesty and Vulnerability
Share failures and mistakes openly:
- What went wrong and why
- What you misunderstood or overlooked
- How you recovered or adapted
- What you'd do differently

### Actionable Lessons
Extract practical insights readers can apply:
```markdown
❌ "Be more careful with technical debt"
✅ "Set a 'debt ceiling': if fixing takes >2 hours, create a spec and schedule it. 
    We reduced emergency fixes by 60% after implementing this rule."
```

### Balance Technical and Personal
Weave technical depth with personal reflection:
- Technical challenges and solutions
- Personal growth and realizations
- Team dynamics and collaboration
- Industry patterns and trends

## Quality Gates (Experiential-Specific)

Beyond blog-common standards:

- [ ] Specific stories with details, not vague generalities
- [ ] Lessons clearly articulated and actionable
- [ ] Honest about failures and mistakes
- [ ] Balance of narrative and technical content
- [ ] Takeaways applicable to readers
- [ ] Natural story flow throughout
- [ ] Humility and authenticity in tone

## References

- [blog-common/SKILL.md](../../blog-common/SKILL.md) - Formatting, localization, quality, style
- [references/questionnaire.md](references/questionnaire.md) - Questionnaire guide
- [LeanSpec template](../../../.lean-spec/templates/articles/experiential.md) - Full spec template with questionnaire
