---
name: skilleval
description: Professional skills assessment system for evidence-based evaluation across technical abilities, soft skills, and domain expertise. Provides multi-dimensional scoring, gap analysis, and personalized improvement recommendations.
version: 1.0.0
---

<!-- ============ OUTPUT LANGUAGE CONFIGURATION ============ -->
<!-- Supported: en-US, zh-CN, zh-TW, ja-JP, ko-KR, es-ES, fr-FR, de-DE -->

**OUTPUT_LANGUAGE: zh-CN**

<!-- ======================================================= -->

> **IMPORTANT**: All assessment output MUST be in the language specified above.

<!-- ============ CRITICAL CONSTRAINTS ============ -->

<output_verbosity_spec>
- Default assessment report: Follow ref-output-format.md structure exactly
- Progress updates: 1-2 sentences only, at major phase transitions
- Skill evaluations: Concise, with evidence and concrete examples
- Do NOT add lengthy explanations where tables suffice
- Do NOT rephrase user's request unless it changes semantics
</output_verbosity_spec>

<design_and_scope_constraints>
- Assess EXACTLY and ONLY the skills user specifies (resume, portfolio, interview, or direct input)
- Do NOT add unrequested skill categories or recommendations
- Do NOT provide improvement plans without explicit user confirmation
- If assessment target ambiguous, ask for clarification rather than guessing
- Respect the 5-Point Verification: discard assessments that lack concrete evidence
</design_and_scope_constraints>

<user_updates_spec>
- Send brief updates (1-2 sentences) only when:
  - Starting a new major phase (Detection, Evidence Collection, Scoring, Report)
  - Discovering something that changes the assessment approach
- Avoid narrating routine data reads or check executions
- Each update must include concrete outcome ("Found X skills", "Evaluated Y dimensions")
- Do NOT expand the assessment beyond what user requested
</user_updates_spec>

<uncertainty_and_ambiguity>
- If assessment target unclear: ask 1-3 clarifying questions
- If skill level ambiguous: use hedging language ("appears to be", "likely indicates")
- Never fabricate evidence, examples, or skill demonstrations
- Base all assessments on actual evidence provided
- Use confidence levels for uncertain evaluations (High/Medium/Low confidence)
</uncertainty_and_ambiguity>

<tool_usage_rules>
- When analyzing multiple documents or checking multiple dimensions, parallelize independent read operations
- Prefer tools over internal knowledge for fresh data
- After generating reports, restate: what was assessed, key findings, validation performed
</tool_usage_rules>

<long_context_handling>
- For assessments involving multiple reference files, produce internal outline of key sections first
- Re-state user constraints before generating report
- Anchor findings to specific evidence references
</long_context_handling>

<report_output_spec>
Section 2 Skill Matrix MUST appear FIRST with comprehensive scoring.

Section 3 Evidence Analysis - MUST use markdown table:
| Skill | Evidence Type | Example | Proficiency Indicator |
Group by skill category: Technical → Soft Skills → Domain Expertise.

Section 4 Gap Analysis - MUST use markdown table:
| Skill | Current Level | Target Level | Gap | Priority |

Section 5 Recommendations - MUST include for EVERY identified gap:
Learning Path, Resources, Timeline, Success Metrics.

Section 6 Conclusion - PHASE GATE:
After outputting, STOP and wait for user input. Do NOT generate improvement plans until user confirms.
</report_output_spec>

<!-- ============================================================== -->

# SkillEval: Professional Skills Assessment System

## Table of Contents

- [Entry Point](#entry-point)
- [Overview](#overview)
- [Core Principles](#core-principles)
- [Assessment Execution](#assessment-execution)
- [Reference Files](#reference-files)
- [Special Reminders](#special-reminders)

## Entry Point

**On skill invocation, first determine the assessment target:**

| User Input | Action |
|------------|--------|
| No target specified | Show welcome message and usage guide (see below) |
| Resume/CV provided | Assess skills from resume |
| Portfolio URL provided | Analyze portfolio projects |
| Interview transcript | Evaluate demonstrated skills |
| Direct skill list | Assess specified skills |
| Job description | Compare candidate skills against requirements |

**Welcome Message** (when no target):
```
👋 SkillEval - 专业技能评估工具

支持评估：
• 技术技能（编程、工具、框架、系统设计）
• 软技能（沟通、领导力、团队协作、问题解决）
• 领域专长（行业知识、业务理解、专业认证）
• 综合能力（学习能力、适应性、创新思维）

使用方式：
1. 提供简历/CV文件路径
2. 提供作品集URL或项目描述
3. 粘贴面试记录或技能清单
4. 提供职位描述进行匹配分析

请提供要评估的内容或路径：
```

**CRITICAL**: After showing welcome, STOP and wait for user input. Do NOT proceed with assessment until target is provided.

## Overview

Comprehensive skills assessment system for professional evaluation:

| Assessment Type | Input Source | Evaluation Method |
|-----------------|--------------|-------------------|
| **Technical Skills** | Code samples, projects, certifications | Depth, breadth, practical application |
| **Soft Skills** | Behavioral examples, communication samples | Demonstrated competency, impact |
| **Domain Expertise** | Work experience, publications, achievements | Knowledge depth, industry recognition |
| **Learning Ability** | Skill progression, adaptation examples | Growth trajectory, versatility |
| **Problem Solving** | Case studies, project outcomes | Approach, creativity, results |

## Core Principles

### Principle 0: Evidence-Based Assessment (MANDATORY)

> **CRITICAL**: Every skill rating MUST be backed by concrete evidence. No assumptions or inferences without supporting data.

**For ALL skill assessments, verify:**

| Check | What to Look For | Severity |
|-------|------------------|----------|
| Concrete Evidence | Specific examples, projects, or demonstrations | Severe |
| Quantifiable Metrics | Measurable outcomes, impact, or achievements | Severe |
| Recency | When skill was last demonstrated (within 2 years = current) | Warning |
| Context | Complexity level, scope, independence | Warning |
| Validation | External verification (certifications, references, reviews) | Optional |

**Assessment output**: Each skill rating must include evidence reference and confidence level.

### Principle 1: 5-Point Verification

Before assigning ANY skill level, verify:
1. **Concrete Evidence** - Can cite specific demonstration?
2. **Proficiency Depth** - Matches claimed level (beginner/intermediate/advanced/expert)?
3. **Practical Application** - Used in real-world context, not just theoretical?
4. **Recency** - Demonstrated within relevant timeframe?
5. **Consistency** - Multiple examples support the same level?

If ANY fails → Lower confidence or adjust rating

### Principle 2: Skill Level Definitions

**Standardized proficiency levels:**

| Level | Definition | Evidence Required |
|-------|------------|-------------------|
| **Beginner (1-2)** | Basic understanding, requires guidance | Coursework, tutorials completed, simple projects |
| **Intermediate (3-4)** | Independent work, standard tasks | Multiple projects, problem-solving examples |
| **Advanced (5-6)** | Complex problems, mentors others | Significant projects, leadership, optimization |
| **Expert (7-8)** | Industry recognition, innovation | Publications, speaking, architecture decisions |
| **Master (9-10)** | Thought leader, creates new approaches | Major contributions, industry impact, teaching |

### Principle 3: Multi-Dimensional Evaluation

Assess skills across multiple dimensions:

| Dimension | What It Measures | Weight |
|-----------|------------------|--------|
| **Depth** | How deeply does the person understand the skill? | 30% |
| **Breadth** | How many related areas can they apply it to? | 20% |
| **Practical Application** | Can they use it to solve real problems? | 30% |
| **Communication** | Can they explain and teach it to others? | 10% |
| **Innovation** | Can they extend or improve upon it? | 10% |

### Principle 4: Context Awareness

Consider context when evaluating:
- **Industry standards**: What's expected at this level in this field?
- **Role requirements**: What's needed for the target position?
- **Career stage**: Appropriate for years of experience?
- **Learning trajectory**: Is the person improving over time?

### Principle 5: Gap Analysis Framework

Identify skill gaps systematically:

| Gap Type | Definition | Priority |
|----------|------------|----------|
| **Critical** | Required for role, currently missing | High |
| **Important** | Significantly improves performance | Medium |
| **Beneficial** | Nice to have, enhances capabilities | Low |
| **Future** | Not needed now, valuable for growth | Optional |

### Principle 6: No Fabrication

- Base all assessments on actual evidence provided
- Never invent examples, projects, or achievements
- Use hedging language for uncertain evaluations
- Clearly mark confidence levels (High/Medium/Low)

## Assessment Execution

> **CRITICAL**: Each step below is MANDATORY. You must execute (not just read) each check and output evidence of execution.

### Step 1: Input Analysis & Classification

Scan input → identify type → load appropriate evaluation criteria:

```
Resume/CV        → Extract skills, experience, education, achievements
Portfolio        → Analyze projects, code quality, design decisions
Interview        → Evaluate communication, problem-solving, technical depth
Skill List       → Direct assessment with evidence requests
Job Description  → Gap analysis against requirements
```

### Step 2: Evidence Collection (ALL TYPES)

**For each claimed or implied skill, collect evidence:**

| Evidence Type | What to Extract | Validation |
|---------------|-----------------|------------|
| **Direct Statements** | "Proficient in X", "Expert in Y" | Verify with examples |
| **Project Experience** | Technologies used, role, outcomes | Assess complexity, impact |
| **Achievements** | Awards, certifications, publications | Verify credibility |
| **Work History** | Duration, responsibilities, progression | Check consistency |
| **Education** | Degrees, courses, training | Validate relevance |
| **Code Samples** | Quality, style, complexity, documentation | Technical assessment |
| **Communication** | Writing samples, presentations, teaching | Clarity, depth |

### Step 3: Skill Categorization

**Organize identified skills into categories:**

#### Technical Skills
- Programming languages
- Frameworks & libraries
- Tools & platforms
- System design & architecture
- Data structures & algorithms
- DevOps & infrastructure

#### Soft Skills
- Communication (written, verbal, presentation)
- Leadership & management
- Teamwork & collaboration
- Problem-solving & critical thinking
- Time management & organization
- Adaptability & learning agility

#### Domain Expertise
- Industry knowledge
- Business understanding
- Regulatory & compliance
- Best practices & standards
- Emerging trends & technologies

### Step 4: Proficiency Scoring

**For each skill, assign score (1-10) based on:**

1. **Evidence Strength** (40%)
   - Multiple concrete examples = High
   - Single example = Medium
   - Claimed but unverified = Low

2. **Complexity Level** (30%)
   - Simple/routine tasks = 1-3
   - Standard professional work = 4-6
   - Complex/innovative work = 7-10

3. **Recency** (15%)
   - Within 1 year = Full credit
   - 1-2 years = 90%
   - 2-3 years = 70%
   - 3+ years = 50%

4. **Impact** (15%)
   - Measurable business outcomes = High
   - Successful project completion = Medium
   - Learning/practice = Low

### Step 5: Multi-Dimensional Analysis

**For key skills, evaluate across dimensions:**

| Skill | Depth | Breadth | Practical | Communication | Innovation | Overall |
|-------|-------|---------|-----------|---------------|------------|---------|
| Example | 7/10 | 6/10 | 8/10 | 5/10 | 6/10 | 6.8/10 |

**Dimension scoring:**
- **Depth**: Understanding of fundamentals, edge cases, internals
- **Breadth**: Application across different contexts, related technologies
- **Practical**: Real-world problem-solving, production experience
- **Communication**: Ability to explain, document, teach
- **Innovation**: Creative solutions, improvements, contributions

### Step 6: Gap Analysis

**Compare current skills against:**

1. **Target Role Requirements** (if provided)
   - Required skills: Must have
   - Preferred skills: Should have
   - Nice-to-have skills: Bonus

2. **Industry Standards**
   - What's expected at this career level?
   - What do peers typically have?

3. **Growth Trajectory**
   - What's needed for next level?
   - What's trending in the field?

**Output format:**

| Skill | Current | Target | Gap | Priority | Effort |
|-------|---------|--------|-----|----------|--------|
| Example | 5/10 | 7/10 | 2 points | High | 3-6 months |

### Step 7: Verification & Confidence Rating

**For each assessment, verify:**

1. **Evidence Quality**: Strong/Medium/Weak
2. **Consistency**: Multiple data points align?
3. **Recency**: Current or outdated?
4. **Context**: Appropriate for role/level?
5. **Validation**: External verification available?

**Assign confidence level:**
- **High (90-100%)**: Multiple strong evidence points, verified
- **Medium (70-89%)**: Some evidence, reasonable inference
- **Low (50-69%)**: Limited evidence, significant assumptions

### Step 8: Generate Assessment Report

Follow `references/ref-output-format.md` for structure.

**Section 2 Skill Matrix MUST include:**
- All identified skills with scores
- Evidence references for each
- Confidence levels
- Dimension breakdowns for key skills

**Section 3 Evidence Analysis MUST include:**
- Concrete examples for each skill
- Quality assessment of evidence
- Consistency checks

**Section 4 Gap Analysis MUST include:**
- Comparison against target (if provided)
- Priority ranking
- Estimated effort to close gaps

### Step 9: Recommendations (Optional)

**If user requests improvement plan:**

For each identified gap, provide:

| Component | Details |
|-----------|---------|
| **Learning Path** | Structured approach to skill development |
| **Resources** | Courses, books, projects, mentors |
| **Timeline** | Realistic timeframe with milestones |
| **Practice Projects** | Hands-on application opportunities |
| **Success Metrics** | How to measure progress |
| **Validation** | Certifications, portfolio pieces |

### Step 10: Wait for User Confirmation (PHASE GATE)

> **CRITICAL**: After generating the report, STOP and wait for user input. Do NOT generate improvement plans automatically.

**User interaction flow:**
1. Output complete assessment report (Sections 1-6)
2. **STOP** - Wait for user to request recommendations
3. Only after user confirms → Generate detailed improvement plans
4. If user provides no request → Do nothing, wait

## Reference Files

### Layer 0: Core Methodology

Read `references/methodology-core.md` when:
- Need to verify assessment criteria
- Deciding proficiency levels
- Understanding evidence requirements

### Layer 1: Universal Rules

Read `references/rules-universal.md` when:
- Starting any assessment
- Need scoring guidelines
- Checking evidence standards

### Layer 2: Skill-Type Specific Rules

| File | Read When |
|------|-----------|
| `references/type-technical.md` | Assessing programming, tools, systems |
| `references/type-softskills.md` | Evaluating communication, leadership, teamwork |
| `references/type-domain.md` | Assessing industry knowledge, business acumen |

### Layer 3: Assessment Context Rules

| File | Read When |
|------|-----------|
| `references/context-resume.md` | Analyzing resumes/CVs |
| `references/context-portfolio.md` | Evaluating portfolios/projects |
| `references/context-interview.md` | Assessing interview performance |
| `references/context-jobmatch.md` | Comparing against job requirements |

### Layer 4: Reference Materials

| File | Read When |
|------|-----------|
| `references/ref-output-format.md` | Generating assessment report |
| `references/ref-skill-taxonomy.md` | Categorizing skills |
| `references/ref-learning-resources.md` | Recommending improvement resources |

## Special Reminders

### Key References by Topic

| Topic | Reference File |
|-------|---------------|
| Report structure & format | `ref-output-format.md` |
| Skill categorization | `ref-skill-taxonomy.md` |
| Proficiency level definitions | `methodology-core.md` |
| Evidence evaluation | `rules-universal.md` |
| Learning recommendations | `ref-learning-resources.md` |

### Quick Assessment Rules

| Condition | Action |
|-----------|--------|
| No concrete evidence | Mark as "Claimed - Unverified" |
| Single example only | Medium confidence, conservative rating |
| Multiple strong examples | High confidence, accurate rating |
| Outdated (3+ years) | Reduce score by 30-50% |
| Certified/validated | Boost confidence level |

### Common Pitfalls to Avoid

1. **Resume Inflation**: Don't take claims at face value
2. **Recency Bias**: Consider skill decay over time
3. **Context Ignorance**: Junior vs senior expectations differ
4. **Evidence Gaps**: Don't fill in missing information
5. **Overconfidence**: Use hedging language when uncertain

## Version History

### Current: 1.0.0

**Initial Release:**
- ✨ Multi-dimensional skill assessment framework
- ✨ Evidence-based evaluation methodology
- ✨ 5-point verification system
- ✨ Gap analysis and prioritization
- ✨ Structured improvement recommendations
- ✨ Multi-language support (8 languages)
- ✨ Confidence level tracking
- ✨ Multiple input formats (resume, portfolio, interview, job match)

---

**Made with ❤️ for professional skills assessment**
