---
name: architecture-readiness
description: Use this skill to create Product Owner Specifications documenting business requirements before architecture design
---

# Architecture Readiness Skill

## Description

This skill helps Product Owners document business requirements and context before architecture design begins. It provides templates and guidance for creating Product Owner Specifications that feed into technical ARCHITECTURE.md documents.

The skill includes two primary functions:
1. **PO Spec Creation**: Templates and guidance for documenting business requirements
2. **PO Spec Evaluation**: Scoring methodology to assess if a PO Spec is ready for architecture team handoff

## When to Use This Skill

Invoke this skill when:
- User asks to create a Product Owner Specification
- User asks about documenting business requirements for architecture
- User mentions "business context", "product requirements", or "requirements gathering" in relation to architecture
- User wants to prepare business documentation before technical architecture design
- User asks to evaluate or score a Product Owner Specification
- User wants to know if their PO Spec is ready for the architecture team

## Files in This Skill

- **PRODUCT_OWNER_SPEC_GUIDE.md**: Comprehensive guide with 8-section template, examples, and best practices
- **templates/PO_SPEC_TEMPLATE.md**: Quick-start template for creating a new PO Specification
- **PO_SPEC_SCORING_GUIDE.md**: Weighted scoring methodology to evaluate PO Spec readiness (0-10 scale)

## How to Use This Skill

### For PO Spec Creation

When this skill is activated for document creation:

1. **Read the guide**: Load PRODUCT_OWNER_SPEC_GUIDE.md to understand the 8-section structure
2. **Understand user context**: Ask clarifying questions about their product/feature
3. **Provide appropriate template**:
   - For guidance and understanding: Reference PRODUCT_OWNER_SPEC_GUIDE.md
   - For quick start: Provide templates/PO_SPEC_TEMPLATE.md
4. **Guide document creation**: Help user fill out each section with business context
5. **Reference mapping**: Explain how PO Spec maps to ARCHITECTURE.md (see guide Section "Mapping to ARCHITECTURE.md")

### For PO Spec Evaluation

When this skill is activated to evaluate a PO Spec:

1. **Read the scoring guide**: Load PO_SPEC_SCORING_GUIDE.md to understand the weighted scoring methodology
2. **Read the PO Spec**: Load the user's Product Owner Specification document
3. **Evaluate each section**: Assess completeness of all 8 sections using the evaluation criteria
4. **Calculate weighted score**: Apply section weights and compute total score (0-10 scale)
5. **Provide detailed feedback**:
   - Overall score and readiness interpretation
   - Section-by-section breakdown showing completeness %
   - Identify gaps in critical sections (Use Cases, Business Constraints, Business Objectives)
   - Provide actionable recommendations for improvement
6. **Determine readiness**: Score ≥7.5/10 indicates ready for architecture team handoff

## Key Principles

- **Business-focused**: No technical details or architecture decisions in PO Spec
- **User-centric**: Emphasize user needs, personas, and pain points
- **Measurable**: All goals and success criteria must be quantifiable
- **Constraint-aware**: Document all business constraints (budget, timeline, compliance)

## Integration with Other Skills

**Relationship to architecture-docs skill:**
- architecture-readiness (this skill) → Creates **business** requirements (PO Spec)
- architecture-docs skill → Creates **technical** architecture (ARCHITECTURE.md)

**Workflow:**
1. Product Owner uses architecture-readiness skill → Creates PO Spec
2. PO Spec handed off to architecture team
3. Architecture team uses architecture-docs skill → Creates ARCHITECTURE.md

## Example Invocations

### Example 1: Creating a PO Spec

```
User: "I want to create a Product Owner Specification for a new mobile payment feature"

Claude (with architecture-readiness skill activated):
1. Load PRODUCT_OWNER_SPEC_GUIDE.md to understand structure
2. Ask clarifying questions:
   - What business problem does this solve?
   - Who are the target users?
   - What are the business objectives and success metrics?
   - Are there regulatory/compliance requirements?
3. Guide user through 8 sections:
   - Business Context
   - Stakeholders & Users
   - Business Objectives
   - Use Cases
   - User Stories
   - User Experience Requirements
   - Business Constraints
   - Success Metrics & KPIs
4. Provide template sections as needed
5. Explain how completed PO Spec feeds into ARCHITECTURE.md
```

### Example 2: Evaluating a PO Spec

```
User: "Can you evaluate my Product Owner Spec and tell me if it's ready for the architecture team?"

Claude (with architecture-readiness skill activated):
1. Load PO_SPEC_SCORING_GUIDE.md to understand scoring methodology
2. Read the user's PRODUCT_OWNER_SPEC.md file
3. Evaluate each section:
   - Section 1 (Business Context): 80% complete
   - Section 2 (Stakeholders): 60% complete
   - Section 3 (Business Objectives): 90% complete
   - Section 4 (Use Cases): 100% complete
   - Section 5 (User Stories): 70% complete
   - Section 6 (UX Requirements): 85% complete
   - Section 7 (Business Constraints): 95% complete
   - Section 8 (Success Metrics): 80% complete
4. Calculate weighted score: 8.85/10
5. Provide feedback:
   - "Your PO Spec scores 8.85/10 - Good, ready for architecture design"
   - "Excellent: Use Cases, Business Constraints, Business Objectives"
   - "Minor gaps: Stakeholders & Users section needs more detailed personas"
   - "Recommendation: Proceed with architecture design; clarify user personas during kickoff"
```

## Common User Requests

**PO Spec Creation:**
- "Help me document business requirements" → Use PRODUCT_OWNER_SPEC_GUIDE.md
- "I need a template for product specifications" → Provide templates/PO_SPEC_TEMPLATE.md
- "How does this relate to the architecture document?" → Reference "Mapping to ARCHITECTURE.md" section in guide
- "What should I include in use cases vs user stories?" → Reference Sections 4 and 5 of guide

**PO Spec Evaluation:**
- "Evaluate my PO Spec" → Use PO_SPEC_SCORING_GUIDE.md to score the document
- "Is my PO Spec ready for the architecture team?" → Score and assess readiness (≥7.5/10)
- "What's missing from my business requirements?" → Identify gaps using evaluation criteria
- "How can I improve my PO Spec score?" → Provide actionable recommendations based on section weights

## Success Criteria

### For PO Spec Creation

A well-formed Product Owner Specification should:
- ✅ Have all 8 sections completed
- ✅ Include specific, measurable success criteria
- ✅ Define user personas with real pain points
- ✅ Document all business constraints (budget, timeline, compliance)
- ✅ Use business language (avoid technical jargon)
- ✅ Map clearly to ARCHITECTURE.md inputs
- ✅ Score ≥7.5/10 on the weighted scoring methodology

### For PO Spec Evaluation

A quality evaluation should:
- ✅ Assess all 8 sections using the evaluation criteria from PO_SPEC_SCORING_GUIDE.md
- ✅ Calculate weighted score correctly (applying section weights)
- ✅ Provide clear readiness determination (Ready ≥7.5/10, Not Ready <7.5/10)
- ✅ Give specific, actionable feedback for each gap identified
- ✅ Prioritize feedback on high-weight sections (Use Cases, Business Constraints, Business Objectives)
- ✅ Explain what needs to be added/improved to reach readiness threshold