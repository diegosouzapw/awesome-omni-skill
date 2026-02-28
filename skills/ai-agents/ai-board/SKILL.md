---
name: ai-board
description: Advanced multi-agent reasoning system for complex questions requiring maximum accuracy. Uses adaptive technique selection, adversarial validation, multi-agent debate, and deep analysis. Trigger phrases "ai board", "expert panel", "deep analysis", "maximum accuracy", "thorough reasoning". Use for critical medical decisions, theological questions, philosophical analysis, complex technical problems, high-stakes decisions, or any question requiring 95%+ confidence.
---

# AI Board - Advanced Reasoning System

## Overview

The AI Board is a sophisticated reasoning system that dynamically selects and combines advanced LLM techniques to achieve maximum accuracy on complex questions. It uses multi-agent debate, adversarial validation, domain adaptation, and structured reasoning to deliver expert-level analysis.

**When to use**: Critical decisions, complex medical cases, theological questions, philosophical analysis, technical problems requiring deep exploration, or any query where accuracy is paramount.

## Core Workflow

The AI Board follows a systematic 5-phase approach that scales with question complexity:

### Phase 1: Question Analysis & Technique Selection

1. **Classify the question**:
   - Domain (medical, theological, philosophical, technical, general)
   - Complexity (simple, moderate, complex, expert-level)
   - Reasoning type (sequential, exploratory, verification, creative)
   - Stakes (routine, important, critical, life-impacting)

2. **Select reasoning techniques** based on classification:
   - Simple questions: Standard CoT + self-consistency (3-5 samples)
   - Moderate: Multi-agent debate (3 agents, 2 rounds)
   - Complex: Full board with adversarial validation (5-7 agents)
   - Expert-level: Extended board + external validation + reflection

3. **Estimate required depth**:
   - Routine: 500-1000 tokens
   - Important: 2000-5000 tokens
   - Critical: 5000-15000 tokens
   - Maximum: Unlimited depth with iterative refinement

### Phase 2: Multi-Agent Analysis

Deploy specialized agents based on question domain:

**Medical questions**:
- Primary clinician (main analysis)
- Specialist (domain expert)
- Devil's advocate (challenges assumptions)
- Evidence reviewer (literature/guidelines)
- Risk assessor (evaluates outcomes)

**Theological questions**:
- Biblical scholar (textual analysis)
- Historical theologian (historical context)
- Systematic theologian (doctrinal coherence)
- Practical theologian (application)
- Devil's advocate (alternative interpretations)

**Philosophical questions**:
- Ethicist (moral dimensions)
- Epistemologist (knowledge/certainty)
- Metaphysician (nature of reality)
- Logician (argument structure)
- Devil's advocate (counterarguments)

**Technical questions**:
- Domain expert (subject matter)
- Systems thinker (interactions)
- Pragmatist (implementation)
- Security/safety reviewer (risks)
- Devil's advocate (edge cases)

**General questions**:
- Generalist (broad analysis)
- Specialist (relevant expertise)
- Critical thinker (logic/reasoning)
- Empiricist (evidence/data)
- Devil's advocate (challenges)

### Phase 3: Adversarial Validation

1. **Initial analysis** by primary agents
2. **Devil's advocate critique**:
   - Identify weak assumptions
   - Challenge reasoning steps
   - Propose alternative interpretations
   - Test edge cases
   - Evaluate confidence levels

3. **Rebuttal and refinement**:
   - Primary agents respond to critiques
   - Revise analyses based on valid objections
   - Strengthen weak arguments
   - Acknowledge genuine uncertainties

4. **Synthesis round**:
   - Integrate validated insights
   - Resolve disagreements
   - Build consensus on high-confidence points
   - Flag remaining uncertainties

### Phase 4: Evidence Grounding & Verification

1. **External validation** (when applicable):
   - Literature search for medical questions
   - Biblical cross-references for theological questions
   - Logical consistency checks for philosophical questions
   - Technical documentation for implementation questions

2. **Self-consistency verification**:
   - Generate 3-5 independent reasoning paths
   - Check for agreement on key conclusions
   - Investigate discrepancies
   - Update confidence based on consistency

3. **Constitutional alignment**:
   - Verify adherence to principles (medical ethics, biblical fidelity, logical rigor)
   - Check for bias or assumptions
   - Ensure balanced consideration of alternatives

### Phase 5: Structured Output with Confidence Levels

Present results in this format:

```markdown
## Analysis Summary

[1-2 paragraph executive summary of the conclusion]

## Key Findings

[Numbered list of main insights with confidence levels]

1. **[Finding 1]** - Confidence: [95%/85%/70%/50%/<50%]
   - Supporting evidence: [brief rationale]
   - Limitations: [what could change this]

2. **[Finding 2]** - Confidence: [level]
   - Supporting evidence: [rationale]
   - Limitations: [uncertainties]

## Reasoning Process

[Detailed analysis showing the reasoning path, including:
- Key decision points
- Alternative hypotheses considered
- Why certain paths were pursued/rejected
- Critical evidence that shaped conclusions]

## Areas of Disagreement/Uncertainty

[Explicitly call out where agents disagreed or evidence is ambiguous:
- What remains uncertain
- What additional information would help
- Edge cases or scenarios where conclusion might not hold]

## Confidence Assessment

Overall confidence in main conclusion: [X%]

Factors increasing confidence:
- [Factor 1]
- [Factor 2]

Factors decreasing confidence:
- [Factor 1]
- [Factor 2]

## Recommendations

[Actionable next steps, further questions to explore, or implementation guidance]
```

## Domain-Specific Guidance

### Medical Questions (for Jordan)

Use the enhanced clinical reasoning framework:

1. **Initial assessment** - Present the case systematically
2. **Differential diagnosis** - Consider alternatives with devil's advocate
3. **Evidence review** - Current literature and guidelines
4. **Risk-benefit analysis** - Evaluate treatment options
5. **Recommendation** - Clear guidance with confidence levels

See `references/medical-reasoning.md` for detailed clinical protocols.

### Theological Questions (for Eric and family)

Use the biblical-theological framework:

1. **Textual analysis** - What does Scripture say?
2. **Historical context** - Original meaning and setting
3. **Systematic integration** - How does it fit with whole Bible?
4. **Application** - What does this mean for us today?
5. **Practical wisdom** - How to live this out

See `references/theological-reasoning.md` for detailed protocols.

### Philosophical Questions (for Eric)

Use the analytical philosophy framework:

1. **Clarify the question** - Define terms precisely
2. **Map positions** - Survey major views
3. **Evaluate arguments** - Assess logical validity
4. **Consider objections** - Devil's advocate critique
5. **Reasoned conclusion** - Tentative position with humility

See `references/philosophical-reasoning.md` for detailed protocols.

## Computational Efficiency Guidelines

Balance thoroughness with token efficiency:

- **Simple questions** (<90% baseline): Use direct answer + offer deeper analysis
- **Moderate questions**: Standard CoT + self-consistency (3 samples) ≈ 2-3K tokens
- **Complex questions**: Multi-agent (5 agents, 2 rounds) ≈ 5-10K tokens
- **Critical questions**: Full board + adversarial validation ≈ 10-20K tokens
- **Maximum depth**: Unlimited for life-impacting decisions

## Key Principles

1. **Confidence calibration**: Always state confidence levels explicitly
2. **Epistemic humility**: Acknowledge uncertainties and limitations
3. **Evidence grounding**: Base conclusions on verifiable evidence
4. **Alternative consideration**: Seriously engage with counterarguments
5. **Practical wisdom**: Balance theoretical rigor with practical application
6. **Domain expertise**: Use domain-specific reasoning for specialized questions
7. **Iterative refinement**: Continue until confidence is justified or uncertainty is irreducible

## Scripts

- `scripts/multi_agent_orchestrator.py` - Coordinates multi-agent analysis
- `scripts/confidence_calculator.py` - Computes calibrated confidence scores
- `scripts/evidence_validator.py` - Validates claims against evidence

## References

- `references/medical-reasoning.md` - Clinical decision support protocols
- `references/theological-reasoning.md` - Biblical-theological analysis framework
- `references/philosophical-reasoning.md` - Analytical philosophy methods
- `references/reasoning-techniques.md` - Comprehensive guide to all techniques
- `references/domain-adaptation.md` - How to adapt reasoning by domain

## Notes

- The AI Board automatically activates when trigger phrases are used or when question complexity warrants deep analysis
- For routine questions, use standard Claude responses; reserve the board for truly complex cases
- The system balances thoroughness with efficiency, scaling approach to question importance
- All analyses include explicit confidence levels and acknowledge Eric's priority of avoiding confident wrongness
