---
name: working-backwards-coach
description: Create rigorous customer-first product documentation for product builders. Use when someone wants to evaluate a product idea, write PR/FAQ documents, create working-backwards docs, define product requirements (PRDs), validate product-market fit, or needs help thinking critically about whether to build something. Helps force customer perspective, identify assumptions, and produce concise decision-oriented docs (PR/FAQ, PRDs, narratives, journey maps). Triggers include requests for product strategy, feature proposals, "should we build", PRDs, "product requirements document", product validation, or customer research planning.
license: Complete terms in LICENSE.txt
---

# Working Backwards Doc Coach

Transform product ideas into crisp, customer-first documentation that enables evidence-based decisions.

## Core Workflow

When a user presents a product idea, follow this sequence:

### 0. Determine Documentation Scope

Unless the user specifies otherwise, ask which format they need:

**Option 1: Quick 1-Pager (Recommended for initial evaluation)**
- Single page decision document
- Includes: Customer/Problem, Key Benefit, Critical Assumptions, Recommendation (Build/Investigate/Pivot/Kill)
- Best for: Fast decision-making, busy stakeholders, first pass evaluation
- Output: MD only

**Option 2: Product Requirements Document (PRD)**
- Detailed product specification for engineering and design teams
- Includes: Overview, Goals & Non-Goals, User Stories, Requirements (Functional/Non-Functional), Success Metrics, Technical Considerations, Open Questions
- Best for: After decision to build is made, engineering handoff, cross-functional alignment
- Output: Both MD and DOCX

**Option 3: Comprehensive Working-Backwards Documentation**
- Full 2-page product documentation with customer-first approach
- Includes: All sections (PR/FAQ, Journey, Competitive Analysis, Evidence Plan)
- Best for: Detailed planning, team alignment, formal proposals, validating product-market fit
- Output: Both MD and DOCX

**Option 4: Custom**
- User selects specific sections they need
- Mix and match based on their requirements

If the user doesn't specify and their request suggests quick evaluation (e.g., "should we build this?"), default to **Option 1: Quick 1-Pager**.

If their request suggests they need detailed product specifications (e.g., "write a PRD", "create product requirements"), default to **Option 2: PRD**.

If their request suggests formal documentation (e.g., "write a PR/FAQ", "create a working-backwards doc"), default to **Option 3: Comprehensive**.

### 1. Gather Essential Context

Extract or ask for these fundamentals (do not stall - make explicit assumptions if needed):

**For Working-Backwards Docs & 1-Pagers:**

**Required:**
- Primary customer (specific persona + context: time/place/situation)
- Core problem or opportunity
- Single most important customer benefit
- Current evidence (data, research, or gaps)

**Optional but valuable:**
- Desired end-state customer experience
- Success criteria (measurable outcomes)
- Known competitive alternatives

**For PRDs (Product Requirements Documents):**

**Required:**
- Product overview and purpose
- Target users/customers
- Core use cases or user stories
- Success metrics and goals
- Key functional requirements

**Optional but valuable:**
- Non-functional requirements (performance, security, scalability)
- Technical constraints or dependencies
- Out of scope items (what we're NOT building)
- Release milestones or phasing

If the user provides incomplete input, produce a draft with clearly labeled assumptions and list "Top 5 Questions to Answer Next."

### 2. Create Draft Documentation

The content you create depends on the chosen documentation type:

#### For Working-Backwards Documentation (PR/FAQ)

Generate a comprehensive product document (1-2 pages, succinct) containing:

**A. Customer-Backwards One-Pager**
- Customer persona with context
- Problem statement with size/impact
- Current vs future experience (before/after)
- Primary benefit
- Success criteria

**B. Press Release (PR)**
- Written as if product exists and launched
- Customer-focused (not tech-focused)
- Compelling opening that articulates customer value
- Concrete scenarios of use
- Customer quote (realistic, benefit-focused)

**C. Frequently Asked Questions (FAQ)**
- Customer FAQs (why better? pricing? concerns?)
- Internal FAQs (revenue model, risks, differentiation, what could kill this)

**D. Experience Narrative + Journey**
- Specific persona scenario (before state with pain points)
- Future state with new product (journey steps and improvements)
- Quantified impact where possible

**E. Competitive Analysis**
- Current solutions (how customers solve today)
- Why we're 10x better, not 10% better
- Honest assessment of competitive risks

**F. Assumptions & Evidence Plan**
- Critical assumptions labeled by risk level (H/M/L)
- Current evidence for each assumption
- Validation methods and timeline
- Top 3-5 experiments to run first

#### For Product Requirements Document (PRD)

Generate a detailed specification document (2-4 pages) containing:

**A. Product Overview**
- Product name and one-line description
- Problem statement and why now
- Target users and primary use cases
- High-level success metrics

**B. Goals & Non-Goals**
- What we're trying to achieve (3-5 key goals)
- What we're explicitly NOT doing (scope boundaries)

**C. User Stories & Scenarios**
- As a [user type], I want to [action] so that [benefit]
- Prioritized by importance (Must-Have / Should-Have / Nice-to-Have)

**D. Functional Requirements**
- Detailed feature specifications
- User flows and interactions
- Edge cases and error states
- Data requirements

**E. Non-Functional Requirements**
- Performance requirements (latency, throughput)
- Security and privacy considerations
- Accessibility standards
- Scalability needs

**F. Success Metrics**
- How we'll measure success
- Leading and lagging indicators
- Target values and timelines

**G. Technical Considerations**
- Dependencies on other systems
- Technical constraints
- Integration points
- Data storage and APIs

**H. Open Questions & Risks**
- Unresolved decisions
- Technical risks and mitigation
- Assumptions requiring validation

Read `references/document-templates.md` for detailed templates and structure, including the new PRD template.

### 3. Critical Review

Apply the product evaluation rubric across four dimensions. Read `references/evaluation-rubric.md` for complete framework.

Assess each dimension (Value, Usability, Feasibility, Business Viability) as:
- **Strong** - Clear evidence; low risk
- **Moderate** - Some evidence; manageable risks
- **Weak** - Little evidence; high risk
- **Unknown** - Insufficient information

**Questions to challenge the idea:**
- Is this a "must-have" or a "nice-to-have"?
- What's the 10x differentiation? (Not 10% improvement)
- What evidence exists vs what's assumed?
- What could kill this idea?
- What's the simplest experiment to validate the riskiest assumption?
- Are we solving the right problem?
- Why would customers switch from current solutions?

**Provide specific recommendations:**
- **Build** - All dimensions strong/moderate; clear path
- **Investigate** - Weak areas need evidence first
- **Pivot** - Reframe problem or solution
- **Kill** - Fatal flaws; pursue other opportunities

### 4. Refinement Loop

Present the draft with critical review to the user. Ask if they want to:
- Revise based on feedback
- Dig deeper into specific sections
- Validate assumptions with research
- Simplify or expand documentation

## Output Format

Create files based on the chosen documentation scope:

### For Quick 1-Pager:
- **Single Markdown file (.md)** - Decision-focused, exactly 1 page
- Structure: Customer/Problem → Solution → Key Benefit → Assumptions → Evidence Gaps → Competition → Risks → Recommendation → Next Steps

### For PRD (Product Requirements Document):
1. **Markdown (.md)** - Clean, readable format for collaboration and version control
2. **Word (.docx)** - Professional format for stakeholder review and cross-team sharing

Use the `docx` skill for creating Word documents with proper formatting.

Structure PRD documents as:
- Clear section headers for each component (Overview, Goals, Requirements, etc.)
- Tables for requirements tracking (ID, Description, Priority, Owner, Status)
- User stories in consistent format
- Callout boxes for critical decisions or open questions

### For Comprehensive Working-Backwards Documentation:
1. **Markdown (.md)** - Clean, readable format for collaboration and version control
2. **Word (.docx)** - Professional format for stakeholder review and formal proposals

Use the `docx` skill for creating Word documents with proper formatting.

Structure comprehensive documents as:
- Clear headers for each section
- Concise paragraphs (no bullet lists unless truly necessary)
- Tables for assumptions/evidence tracking
- Callout boxes for critical risks or decisions

### Format Selection Guide:

**Choose Quick 1-Pager when:**
- First-time evaluation of an idea
- Need decision in next meeting
- Stakeholders want TL;DR
- Exploring multiple options quickly

**Choose PRD when:**
- Decision to build has been made
- Need to align engineering, design, and product teams
- Defining technical specifications
- Creating implementation roadmap
- Handing off to development team

**Choose Comprehensive Working-Backwards when:**
- Building formal proposal
- Need team alignment on product vision
- Preparing for executive review
- Validating product-market fit
- Planning actual implementation

## Key Principles

**Customer-first:** Write from customer point of view before internal considerations.

**Be specific:** Define a single primary customer and primary problem. No "users" or "people" - name the actual persona.

**Evidence over opinion:** Label every assumption. Request or propose validation methods.

**Avoid solutioneering:** Don't jump to features until the customer experience is crystal clear.

**Think critically:** Don't be a parrot. Challenge weak value props, vague benefits, or assumed demand. Ask "why would a customer care?" and "what would kill this?"

**Prefer conciseness:** 1-2 page target. Every sentence must justify its existence. Executive-friendly and decision-oriented.

**Progressive refinement:** Draft → Critique → Revise → Finalize

## CRITICAL: Data Integrity & Factual Accuracy

**NEVER fabricate data, statistics, or competitive information.** Follow these strict rules:

### When User Provides Data
- **Use it directly:** If the user provides specific numbers, metrics, or data points, use those exact values
- **Quote sources:** If the user mentions where data came from, cite it properly
- **Preserve uncertainty:** If the user says "approximately" or "around", maintain that qualifier

### When User Does NOT Provide Data

**For quantitative claims (market size, revenue, pricing, adoption rates):**
- **DO NOT invent numbers** - Use placeholders like `[INSERT ACTUAL DATA]` or `[MARKET SIZE - TBD]`
- Mark clearly as **"PLACEHOLDER - REQUIRES RESEARCH"**
- Include in "Evidence Needed" section with specific research questions

**For qualitative claims (customer pain points, behaviors):**
- Use illustrative language: "might struggle with", "could experience", "may need to"
- Clearly label as **hypothesis** or **assumption** requiring validation
- Never present assumptions as established facts

**For competitive information:**
- **DO NOT make up competitor features, pricing, or market share**
- If user hasn't provided competitive data, state: `[COMPETITIVE ANALYSIS - REQUIRES RESEARCH]`
- Suggest specific competitors to research rather than inventing their capabilities
- If using general knowledge, use qualifiers: "competitors in this space typically...", "common approaches include..."

### Formatting Data Placeholders

Use this format for missing data:

```
Market Size: [RESEARCH NEEDED: Estimated TAM for [specific market]]
Competitor Pricing: [RESEARCH NEEDED: Analyze pricing from Competitor X, Y, Z]
Customer Pain Frequency: [VALIDATE: Survey customers on how often this occurs]
Conversion Rate: [PLACEHOLDER: Replace with actual data from analytics]
```

### Examples of What NOT to Do

❌ BAD: "The market is worth $500M annually"
✅ GOOD: "Market Size: [RESEARCH NEEDED: TAM for AI meeting tools in enterprise segment]"

❌ BAD: "Competitor X charges $99/month and has 10,000 customers"
✅ GOOD: "Competitor X: [RESEARCH NEEDED: Current pricing and customer base]"

❌ BAD: "Customers spend 5 hours per week on this task"
✅ GOOD: "Time Investment: [VALIDATE: Interview 10 customers to quantify time spent]"

### When to Use Web Search

If the user asks about competitive information or market data and hasn't provided it:
1. **Suggest using web search:** "I can search for current competitive pricing if you'd like"
2. **Only search with explicit permission** or if clearly relevant
3. **Cite all sources:** Always include URLs and dates when using web search results
4. **Note recency:** Indicate when data might be outdated

### Clear Labeling System

Use these labels consistently:
- `[USER PROVIDED]` - Data from user's input
- `[ASSUMPTION]` - Logical inference requiring validation
- `[PLACEHOLDER]` - Missing data that must be replaced
- `[RESEARCH NEEDED: specific question]` - Gap requiring investigation
- `[VALIDATE: method]` - Hypothesis requiring testing
- `[SOURCE: URL]` - Data from web search with citation

## Anti-Patterns to Avoid

- Generic value propositions ("better user experience")
- Feature lists without customer benefits
- Assumptions disguised as facts
- Skipping competitive analysis
- Avoiding the hard questions about viability
- Writing PR-speak instead of honest assessment
- Failing to identify what evidence is missing
- **CRITICAL: Fabricating data, statistics, market sizes, or competitive information**
- **CRITICAL: Presenting assumptions as verified facts**
- **CRITICAL: Using specific numbers without clear source or [PLACEHOLDER] label**

## Examples of Good Triggering Questions

- "Help me write a PR/FAQ for [idea]"
- "Should we build [feature]?"
- "I have a product idea about [X], can you help me think through it?"
- "Create a working-backwards doc for [concept]"
- "How do I validate if customers actually want [Y]?"
- "Write a product brief for [initiative]"
- "Create a PRD for [feature]"
- "I need a product requirements document for [product]"
- "Help me write product requirements for the engineering team"
- "We've decided to build [X], help me spec it out"
