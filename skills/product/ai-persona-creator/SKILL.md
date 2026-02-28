---
name: ai-persona-creator
description: Use when analyzing stakeholder psychology for negotiations, proposals, or persuasion. Creates research-backed personas revealing hidden motivations.
---

# AI Persona Creator

## 🎯 When to Use

Activate this skill when you need to:
- **Persuade stakeholders**: Understand CISO, CFO, CIO psychology for approvals
- **Prepare negotiations**: Analyze counterparty before critical meetings
- **Write winning proposals**: Identify what decision-makers really care about
- **Enable sales teams**: Create deep buyer personas beyond demographics

**Trigger**: "Create persona for [Role] on [Project]"

---

## ⚡ Quick Start (30 seconds)

**Copy and run this prompt**:

```
Create an AI persona for:
- Job/Role: [e.g., E-commerce CTO]
- Project: [What you're proposing, e.g., Headless commerce platform migration]
- Goal: [What you want from them, e.g., Get technical approval]
```

**What happens automatically**:
1. 32-48 web searches across 4 categories
2. Cross-verification of 3+ independent sources
3. 10-framework psychological analysis (Tier 1-3 foundation system)
4. Complete persona with research citations
5. Self-refinement through 3 quality rounds

**Time**: 60-90 minutes automated

---

## 📋 Step-by-Step Execution

### Phase 1: Research (30-45 min)

**STEP 1 → Run Web Search Strategy**

Execute this prompt:
```
Run comprehensive web research for [Role] persona:

Search Categories (8-12 queries each):
1. Job Understanding: "[Role] responsibilities", "[Role] daily challenges"
2. Decision Psychology: "[Role] decision-making process", "how [Role] evaluates [Project type]"
3. Org Culture: "[Role] reporting structure", "[Role] KPIs and metrics"
4. Field Testimony: "[Role] forum discussions", "[Role] Reddit/community quotes"

Save all results to: research/[role-name]/web-search-results.md
```

**STEP 2 → Synthesize Research**

Execute this prompt:
```
Synthesize web research with cross-verification:

Requirements:
- Minimum 3 independent sources per insight
- 5+ direct practitioner quotes required
- Cross-verify conflicting information
- Calculate confidence levels (%)

Save findings to: research/[role-name]/research-findings.md
```

---

### Phase 2: Analysis (20-30 min)

**STEP 3 → Apply 4-Layer Analysis**

Execute this prompt:
```
Analyze [Role] using Divide & Conquer method:

Layer 1 (Surface): What do they say publicly?
Layer 2 (Hidden): What are they really thinking?
Layer 3 (Org Pressure): What organizational forces drive behavior?
Layer 4 (Primal): What evolutionary instincts are triggered?

Use research findings from STEP 2.
```

**STEP 4 → Run 5 Why Technique**

Execute this prompt:
```
Trace [Role]'s behavior to root psychological cause:

Start with: [Observed behavior, e.g., "Rejects cloud proposals"]
Why 1: [First-level reason]
Why 2: [Deeper reason]
Why 3: [Organizational reason]
Why 4: [Psychological reason]
Why 5: [Root evolutionary/primal reason]

Save analysis to psychology profile.
```

---

### Phase 3: Psychology (20 min)

**STEP 5 → Apply 10 Frameworks (Tier 1 Foundation + Core Frameworks)**

Execute this prompt:
```
Apply all 10 psychological frameworks to [Role]:

TIER 1 FOUNDATION (Critical base layer - read first):
- frameworks/08_motivational_psychology.md → SDT, McClelland's needs, goal orientation
- frameworks/09_personality_psychology.md → Big Five/OCEAN, regulatory focus, Type A/B
- frameworks/10_trust_risk_psychology.md → Trust equation, loss aversion, betrayal sensitivity

Then apply CORE FRAMEWORKS with Tier 1 foundation context:
- frameworks/01_kahneman.md → System 1/2, biases (contextualized by personality)
- frameworks/02_cialdini.md → 6 persuasion principles, ranked by motivation profile
- frameworks/03_voss.md → Hidden needs, black swans (aligned with trust baseline)
- frameworks/04_navarro.md → Stress triggers, comfort zones (type A/B based)
- frameworks/05_ariely.md → Irrationality patterns (personality-driven)
- frameworks/06_evolution.md → Primal drives (survival, status, tribal)
- frameworks/07_organization.md → Power structure, stakeholders

Integration approach:
- First establish WHO they are (Personality), WHAT drives them (Motivation), SAFETY needs (Trust)
- Then apply remaining frameworks contextualized by Tier 1 foundation
- Use frameworks/00_integration.md for weighted importance and integration logic

Save to: research/[role-name]/psychology-profile.md
```

**STEP 6 → Create Comprehensive Profile**

Execute this prompt:
```
Generate comprehensive profile with Tier 1 foundation + analysis:

TIER 1 FOUNDATION (WHO they are fundamentally):
- Personality (Big Five): Openness, Conscientiousness, Extraversion, Agreeableness, Neuroticism
- Motivation (McClelland): Achievement (nAch), Power (nPow), Affiliation (nAff)
- Trust Baseline: Current trust level, credibility, reliability, intimacy, self-orientation

Core Fears (Top 3 with % confidence, grounded in Tier 1):
1. [Fear] (X%) - Evidence: "[Quote]" (Source) - Tier 1 drivers: [personality/motivation/trust factors]
2. [Fear] (Y%) - Evidence: "[Quote]" (Source)
3. [Fear] (Z%) - Evidence: "[Quote]" (Source)

Core Motivations (Top 3 with % confidence, aligned with motivation profile):
1. [Motivation] (X%) - Evidence + Implication
2. [Motivation] (Y%) - Evidence + Implication
3. [Motivation] (Z%) - Evidence + Implication

Decision Style Prediction (contextualized by personality):
- System 1 (Emotion): X%
- System 2 (Logic): Y%
- Primary bias: [Bias name]
- Personality drivers: [How Big Five and regulatory focus shape decision-making]

Top 3 Persuasion Levers (ranked by motivation + personality fit):
1. [Cialdini principle] (X%) - Why effective for [personality/motivation profile]
2. [Cialdini principle] (Y%) - Why effective for this role
3. [Cialdini principle] (Z%) - Why effective for this role
```

---

### Phase 4: Generation (10-15 min)

**STEP 7 → Generate Persona Prompt**

Execute this prompt:
```
Create complete persona prompt using template:

Read template: templates/persona-prompt.md

Fill in all sections:
- Background (with past trauma)
- Psychological profile (all 7 frameworks)
- Feedback generation framework (3 steps)
- Red flag detection (4 categories)
- Research sources (all citations)

Save to: research/[role-name]/persona-prompt.md
```

**STEP 8 → Self-Refine (3 Rounds)**

Execute this prompt:
```
Self-refine persona through 3 rounds:

Round 1 - Evidence Check:
[ ] 3+ sources per major claim?
[ ] 5+ field testimonials?
[ ] All 10 frameworks applied (Tier 1 + 7 core)?
[ ] Tier 1 foundation explicitly established (personality/motivation/trust)?

Round 2 - Quality Rubric (50 points):
[ ] Research quality (10 pts)
[ ] Analysis depth (10 pts)
[ ] Tier 1 foundation integration (5 pts) - NEW
[ ] Psychological insight (15 pts) - includes Tier 1 contextualization
[ ] Actionability (10 pts) - aligned with psychological profile
[ ] Citations (5 pts)

Round 3 - Final Polish:
[ ] Confidence levels accurate?
[ ] Tier 1 foundation (personality/motivation/trust) clearly articulated?
[ ] Framework integration shows how Tier 1 shapes other frameworks?
[ ] Tactical applications clear and grounded in psychology?
[ ] Alternative solutions provided?

Target: 45+/50 points (90%+) with full Tier 1 foundation integration
```

---

## 💡 Usage Examples

### Example 1: E-commerce CTO (Platform Migration Approval)

**Input**:
```
Job/Role: E-commerce CTO
Project: Headless commerce platform migration
Goal: Get technical approval
```

**Key Output** (60-min automated):
- Core fear: Platform downtime during peak sales (95%)
- Decision: 75% data-driven (performance metrics)
- Top levers: Authority (90%), Scarcity (85%)
- Strategy: Performance benchmarks + competitor adoption + ROI projections

**Persona Quote**:
> "Your architecture looks solid. But what keeps me up: explaining to the CEO
> why we had outages during Black Friday. I need: (1) Load testing results to prove
> scalability, (2) 3 similar-scale e-commerce platforms using this, (3) Migration roadmap
> with zero-downtime plan."

---

### Example 2: VC Partner (Series A Funding)

**Input**:
```
Job/Role: VC Investment Partner
Project: $5M Series A fundraising
Goal: Win investment
```

**Key Output**:
- Core fear: Missing next unicorn (90%)
- Decision: 60% social proof (peer validation)
- Top levers: Scarcity (90%), Social Proof (95%)
- Strategy: FOMO tactics + "closing soon" + other VC interest

---

## 📚 Reference Documentation

**Framework Details** (when you need deep dives):

**TIER 1 FOUNDATION (Read first)**:
- `frameworks/08_motivational_psychology.md` - Motivation drivers (SDT, McClelland, goal orientation)
- `frameworks/09_personality_psychology.md` - Personality profile (Big Five, regulatory focus, Type A/B)
- `frameworks/10_trust_risk_psychology.md` - Trust foundation (trust equation, loss aversion, betrayal sensitivity)

**CORE FRAMEWORKS (Read with Tier 1 context)**:
- `frameworks/01_kahneman.md` - Cognitive biases, System 1/2
- `frameworks/02_cialdini.md` - 6 persuasion principles
- `frameworks/03_voss.md` - Tactical empathy, hidden needs
- `frameworks/04_navarro.md` - Behavior analysis
- `frameworks/05_ariely.md` - Predictable irrationality
- `frameworks/06_evolution.md` - Primal drives
- `frameworks/07_organization.md` - Power structures

**INTEGRATION GUIDE**:
- `frameworks/00_integration.md` - How to apply Tier 1 + all 7 core frameworks together

**Templates**:
- `templates/persona-prompt.md` - Complete prompt structure

**Additional Resources**:
- `EXAMPLES.md` - 3 complete end-to-end examples
- `README.md` - Installation and setup guide

---

## ⚙️ Integration with Other Skills

**Seamless workflows**:

1. **Proposal Writing** → `docx` skill
   - Create persona → Customize proposal → Generate professional document

2. **Market Research** → `web-research` skill
   - Industry trends → Stakeholder psychology → Market entry strategy

3. **Strategic Planning** → `thinking-framework` skill
   - Analyze problem → Understand reactions → Implementation plan

4. **ROI Analysis** → `roi-analyzer` skill
   - Calculate benefits → Understand CFO/CIO → Frame data appropriately

---

## 🚨 Limitations & Validation

**AI Limitations**:
- Web info may be outdated (re-search every 3 months)
- Cultural nuances may need local validation
- Individual variation exists (this is an average profile)

**Recommended Validation**:
- Interview real stakeholders when possible
- A/B test: Compare predictions vs actual reactions
- Track success rate: % of accurate predictions
