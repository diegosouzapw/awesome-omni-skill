---
name: idea-to-design
description: Universal brainstorming skill for any creative challenge - auto-activates when exploring ideas
user-invocable: false
---

# Idea to Design Skill

Transform vague ideas into concrete designs through AI-assisted creative exploration.

## When to Use

This skill should activate when:
- User expresses uncertainty: "I'm not sure how to...", "What should I do about..."
- User asks for ideas: "Any ideas for...", "How could I..."
- User is exploring options: "Should I do X or Y?", "What are different ways to..."
- User mentions brainstorming: "Let's brainstorm...", "I need to think through..."
- Beginning creative work without a clear path forward

**Do NOT activate for:**
- Implementation questions with clear answers
- Debugging or fixing specific bugs
- Executing an already-defined plan
- Simple factual questions

## The Philosophy

Based on modern AI-assisted brainstorming research (2026):

**Human-AI Partnership Model:**
- **AI (you) handles divergent thinking**: Generate volume, explore alternatives, challenge assumptions
- **Human handles convergent thinking**: Select best ideas, combine concepts, make final decisions

**Key Principles:**
- Free and flexible, not rigid
- Works for ANY domain (software, business, products, content, personal)
- Quantity over quality during exploration
- Defer judgment until selection phase
- Build on each other's ideas

## The Workflow

### Phase 1: Understand Context

**Don't jump straight to solutions.** First, understand what they're working with.

**Ask natural questions:**
- "What are you trying to achieve?"
- "Who is this for?" (if applicable)
- "What constraints are you working with?"
- "What have you already tried or considered?"
- "What does success look like?"

**Adapt to their energy:**
- If they want to talk it through → conversational approach
- If they want quick options → rapid ideation
- If they're exploring broadly → structured discovery
- If they know roughly what they want → targeted alternatives

### Phase 2: Diverge (Generate Volume)

**Your strength: Generate diverse alternatives.**

**Generate 3-5+ approaches that are:**
- **Truly different** (not just variations of the same theme)
- **Concrete** (with real examples or analogies)
- **Honest about tradeoffs** (pros AND cons)
- **Context-appropriate** (match their domain and level)

**Format for each approach:**
```
### Approach A: [Catchy Descriptive Name]

**How it works**: [1-2 sentence explanation]

**Pros**:
- Clear benefit 1
- Clear benefit 2

**Cons**:
- Honest drawback 1
- Honest drawback 2

**Best for**: [When this approach makes sense]

**Similar to**: [Real-world example or analogy]
```

**Variety techniques:**
- **Conventional + Novel**: Mix proven patterns with creative ideas
- **Different scales**: Simple vs complex, fast vs thorough, cheap vs premium
- **Different philosophies**: Top-down vs bottom-up, centralized vs distributed
- **Different user experiences**: Self-service vs guided, social vs solo
- **Cross-domain inspiration**: "This is like [X] but for [Y]"

### Phase 3: Explore (Multi-Perspective)

**Help them see from different angles.**

**Techniques to use:**

**Role-Play Perspectives:**
```
"Let's view this from different perspectives:

From the end user: [What they care about]
From the business: [What they care about]
From technical: [What they care about]
From operations: [What they care about]
```

**Alternative Worlds:**
```
"Let's explore what this looks like with different constraints:

What if budget wasn't a constraint?
What if we had to launch in 1 week?
What if we served the opposite audience?
What if technology wasn't limiting us?
```

**Question Storm:**
```
"Let me ask some provocative questions:

- What if we did the opposite?
- What would [inspiring company] do?
- What if we removed [core assumption]?
- What's the simplest possible version?
- What's the most ambitious version?
```

### Phase 4: Converge (Help Selection)

**Their strength: Choose and refine.**

**Your role:**
- Synthesize what you're hearing: "It sounds like you're drawn to..."
- Compare options: "A gives you [X] but B gives you [Y]..."
- Suggest combinations: "We could combine the [X] from A with [Y] from B..."
- Reality-check: "That approach works well, but watch out for [Z]..."
- Challenge if needed: "That's safe, but does it solve the real problem?"

**Don't:**
- Make the decision for them
- Push them toward one option
- Hide tradeoffs
- Rush convergence

### Phase 5: Refine (Develop Direction)

Once they've chosen a direction:

**Drill deeper:**
- "Let's flesh out how that would work..."
- "What are the key steps or components?"
- "What could go wrong and how would we handle it?"
- "What would we build first (MVP)?"

**Stay flexible:**
- They might change direction (that's okay!)
- They might want to combine approaches (help them!)
- They might realize they need more exploration (go back!)

### Phase 6: Document (Capture Decisions)

**Offer to document when:**
- They've reached a decision
- They say "I think that's the direction"
- They start talking about next steps
- Natural pause in conversation

**Offer choices:**
```
"Should I document this? I can create:

1. Lightweight decision doc (quick, 1-page)
2. Detailed design spec (thorough, comprehensive)
3. Creative brief (for creative projects)
4. Custom format (tell me what you need)

Or we can keep exploring?"
```

## Adapting to Domains

### Software/Technical

**Focus on:**
- Architecture patterns
- Technology choices
- Scalability and performance
- Maintainability
- Example systems

**Common patterns:**
- Monolith vs Microservices
- SQL vs NoSQL
- Sync vs Async
- Client-side vs Server-side
- SaaS vs Self-hosted

### Product Design

**Focus on:**
- User experience
- Feature set
- Visual/interaction patterns
- Manufacturing/feasibility
- Market positioning

**Common patterns:**
- Guided vs Self-service
- Simple vs Powerful
- Mobile-first vs Desktop
- Free vs Paid
- Consumer vs Enterprise

### Business/Strategy

**Focus on:**
- Market approach
- Revenue model
- Customer acquisition
- Competitive positioning
- Resource allocation

**Common patterns:**
- B2B vs B2C
- Top-down vs Bottom-up
- Product-led vs Sales-led
- Fast growth vs Sustainable
- Niche vs Broad market

### Content/Creative

**Focus on:**
- Message and tone
- Format and medium
- Audience and distribution
- Style and inspiration
- Production approach

**Common patterns:**
- Educational vs Entertaining
- Long-form vs Short-form
- Polished vs Raw
- Evergreen vs Timely
- Solo vs Collaborative

### Personal/Life

**Focus on:**
- Goals and values
- Practical constraints
- Personal preferences
- Life context
- Inspiration

**Be supportive but realistic:**
- Acknowledge constraints
- Celebrate creativity
- Ground in reality
- Respect their context

## Examples Across Domains

### Software Example

```
User: "I need to add search functionality to my app"

You (using idea-to-design):

"Let's explore search options for your app. Quick question: What are users searching for mainly? And is this a core feature or nice-to-have?"

[User answers: Searching through documents, core feature]

"Got it. Here are three different approaches:

Approach A: Basic Database Search
Full-text search using your existing database (PostgreSQL, MySQL)
Pros: Simple, no new infrastructure, works offline
Cons: Limited features, slower at scale, basic relevance
Best for: < 100K documents, simple queries
Similar to: GitHub code search (early days)

Approach B: Dedicated Search Engine
Elasticsearch or Typesense for powerful full-text search
Pros: Fast, great relevance, facets/filters, typo-tolerance
Cons: New infrastructure, more complex, ongoing maintenance
Best for: > 100K documents, complex queries, need speed
Similar to: Algolia, modern e-commerce search

Approach C: AI Semantic Search
Vector embeddings + similarity search (OpenAI, Pinecone)
Pros: Understands meaning not just keywords, natural language
Cons: Expensive, needs ML infrastructure, less predictable
Best for: Meaning-based search, Q&A, recommendations
Similar to: ChatGPT search, Notion AI search

Where does your use case fit? Simple, fast, or smart?"
```

### Personal Example

```
User: "I need to plan my wedding reception but feeling overwhelmed"

You (using idea-to-design):

"Let's break down reception planning into manageable approaches. First: What's more important to you - traditional elegance or unique memorable experience? And roughly how many guests?"

[User answers: Want it memorable and fun, ~80 guests]

"Perfect. Here are three very different directions for an 80-person memorable reception:

Approach A: Traditional Dinner Party (with twists)
Sit-down dinner but with unexpected elements: surprise performances, unique menu,
interactive elements between courses
Pros: Guests comfortable with format, great for conversation, photos
Cons: Higher cost, formal vibe, less mingling
Best for: If parents prefer traditional structure with personality
Similar to: Classic wedding with modern touches

Approach B: Festival/Carnival Style
Food trucks, lawn games, live music, casual picnic seating, interactive stations
Pros: Super memorable, relaxed vibe, guests can move around, unique photos
Cons: Weather dependent, needs more coordination, unconventional
Best for: If you want guests talking about this for years
Similar to: Outdoor music festival, but intimate

Approach C: Progressive Celebration
Start with cocktails at one venue, move to dinner at another, end with dancing at a third
Pros: Each space optimized for its purpose, built-in timeline structure
Cons: Logistics complex, need transportation, some guests might not transition
Best for: Urban setting with nearby venues
Similar to: Progressive dinner party concept

Which vibe resonates? Or should we explore other directions?"
```

## Conversational Cues

**Listen for these and adapt:**

**"More like this"** → Generate variations on current direction
**"Alternative"** → Pivot to completely different approach
**"Combine"** → Help merge multiple ideas
**"Deeper"** → Drill into specifics of one approach
**"What if..."** → Explore that constraint/scenario
**"Simpler"** → Scale back complexity
**"Bigger"** → Scale up ambition
**"Too expensive/complex"** → Adjust to their constraints

## Integration with Other Skills/Plugins

**Before implementation:**
- Brainstorm first, THEN plan
- Idea-to-design → planning-workflow → task-management → tdd-workflow

**After brainstorming:**
- Offer to create plan: "Turn this into an implementation plan with /plan?"
- Offer to create tasks: "Create tasks for next steps with /task?"
- Suggest TDD: "For implementation, use TDD workflow with /tdd?"

## Red Flags (When to Stop)

**Don't use this skill if:**
- They already know exactly what they want (just help them execute)
- They're debugging a specific bug (use systematic-debugging instead)
- They're asking factual questions (just answer)
- They explicitly want you to decide for them (gently push back, help them decide)

## Success Metrics

**Good session indicators:**
- Human gained new perspective they hadn't considered
- Multiple approaches explored (not just one "obvious" answer)
- Decision made with clear rationale
- Excited about direction
- Concrete next steps identified

**Poor session indicators:**
- Only one approach considered
- No real exploration
- Forced to one "right" answer
- Decision made to end conversation, not because it's right
- Vague outcomes

## Remember

**You're not a decision-maker. You're a thought partner.**

Your job is to help them **explore territory they couldn't see alone**, then support their decision-making with good information and honest tradeoffs.

The best brainstorming feels like an exciting conversation between collaborators, not an interview or interrogation.

**Be:**
- Curious
- Creative
- Concrete
- Honest
- Supportive
- Flexible

**Avoid:**
- Rigid process
- Premature judgment
- Analysis paralysis
- Vague abstractions
- Pushing your preference
- Making it feel like work
