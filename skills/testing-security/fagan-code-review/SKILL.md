---
name: fagan-code-review
description: Systematic code inspection methodology for finding errors through structured team review. Based on Michael Fagan's formal inspection process (1976). Use for code reviews, design reviews, and quality audits.
license: Apache-2.0
version: 1.0.0
tags: [code-review, quality-assurance, inspection, methodology, error-detection]
---

# Fagan Code Review

A systematic code inspection methodology based on Michael Fagan's formal inspection process (1976). This skill teaches structured error detection through team-based review with separated objectives, defined roles, and measurable outcomes.

## Quick Start

**For AI-assisted inspection:**
```
"Conduct a Fagan inspection on src/auth/login.js"
```

**For detailed usage instructions:** See [HOW-TO-USE.md](HOW-TO-USE.md) for step-by-step guides and examples.

**For human teams:** Follow the 5-phase process below with your inspection team.

## When to Use This Skill

- Conducting formal code inspections (design or implementation)
- Reviewing pull requests with systematic rigor
- Performing quality audits on critical code
- Training teams in structured review processes
- Establishing inspection checkpoints in development workflow
- Analyzing error patterns and improving code quality

## Core Principles

**Separation of Objectives**: Never mix error finding with error fixing, or education with inspection.

**Systematic Coverage**: All code inspected, all branches examined, all logic verified.

**Role-Based Process**: Moderator, designer, implementor, and tester each have specific responsibilities.

**Measurable Outcomes**: Track errors by type, severity, and density for continuous improvement.

**Early Detection**: Find errors as close to origin as possible (10-100× cost savings).

## The 5-Phase Process

### 1. Overview (Optional - Team Meeting)
- **When**: New material or new team members
- **Duration**: As needed to understand context
- **Who**: Designer presents to entire team
- **Output**: Shared understanding of what's being reviewed
- **Skip if**: Same team reviewed the design

### 2. Preparation (Individual)
- **Duration**: Study material individually
- **Rate**: 100-125 LOC/hour
- **Activities**:
  - Study code/design and documentation
  - Review error checklists (load `references/checklists.md`)
  - Note questions and potential errors
  - Do NOT fix errors during preparation
- **Output**: Prepared participants ready for inspection

### 3. Inspection Meeting (Team - MOST CRITICAL)
- **Duration**: Maximum 2 hours per session
- **Rate**: 130-150 LOC/hour
- **Focus**: **FIND ERRORS ONLY** - no fixing, no design discussions
- **Process**:
  - Reader paraphrases how they will implement
  - Cover every piece of logic at least once
  - Take every branch at least once
  - Questions pursued only to point of error recognition
  - Moderator notes errors with type and severity
  - NO solution hunting
- **Output**: List of errors with classifications

### 4. Rework (Individual)
- **Who**: Original author
- **Duration**: Until all errors resolved
- **Activities**: Fix all errors identified in inspection
- **Output**: Corrected code/design

### 5. Follow-Up (Moderator-Led)
- **Critical Rule**: ALL errors must be resolved before proceeding
- **Reinspection Required If**: >5% of material reworked
- **Moderator Verifies**: All issues from inspection are correctly fixed
- **Output**: Signed-off inspection report

## Conducting an Inspection

### Before the Meeting

1. **Moderator schedules** meeting with all roles filled
2. **Distribute materials** to all participants 1-2 days before
3. **All participants prepare** individually (critical for success)
4. **Review checklists** for error types (load `references/checklists.md`)

### During the Meeting (2 Hours Max)

```
✅ DO:
- Focus solely on finding errors
- Let the reader paraphrase the logic
- Cover every line and every branch
- Note error type and severity immediately
- Pursue questions only until error is identified
- Keep moving to cover all material

❌ DON'T:
- Discuss how to fix errors
- Debate design alternatives
- Go down solution rabbit holes
- Let meetings exceed 2 hours (efficiency drops)
- Skip "obvious" or "simple" sections
- Use inspection results for performance reviews
```

### After the Meeting

1. **Moderator produces report within 24 hours**
2. **Author fixes all identified errors**
3. **Moderator verifies all fixes in follow-up**
4. **Reinspect if >5% reworked**
5. **Update error metrics and distributions**

## Roles and Responsibilities

**Load `references/roles-responsibilities.md` for complete details.**

Quick summary:
- **Moderator**: Manages process, notes errors, ensures follow-up (most critical role)
- **Designer**: Presents the design or architecture
- **Implementor/Coder**: Acts as reader, paraphrases implementation
- **Tester**: Reviews testability and test coverage

**Team Size**: 4 people optimal

## Error Classification

Classify each error by three dimensions:

**Type**: LO (Logic), IC (Interconnection), TB (Test/Branch), DE (Design Error), etc.
**Category**: M (Missing), W (Wrong), E (Extra)
**Severity**: Major (causes malfunction) or Minor (lesser impact)

Example: `LO/M/MAJ` = Logic error, Missing, Major severity

**Load `references/error-classification.md` for complete taxonomy.**

## Metrics and Tracking

### Key Metrics
- **Errors per K.LOC**: Primary quality metric
- **Error detection efficiency**: (Errors found / Total errors) × 100
- **Inspection rate**: LOC per hour
- **Rework effort**: Hours per K.LOC

### Target Results (from Fagan 1976 research)
- 60-82% error detection efficiency
- 20%+ productivity improvement vs. no inspection
- 38% fewer errors vs. informal reviews

### Use Metrics For
- Identifying error-prone modules
- Allocating testing effort
- Process improvement
- Training focus areas

**Use the inspection report tool to track metrics over time. Aggregate data manually or via custom tooling.**

## Inspection Types

**I₁ - Design Inspection**: Review design before coding
- Entry: Design complete to level of 3-10 code instructions per statement
- Focus: Logic, design decisions, interfaces
- Use design checklists

**I₂ - Code Inspection**: Review implementation
- Entry: First clean compilation
- Focus: Implementation correctness, test/branch logic, interconnections
- Use code checklists

**Modified Code Inspection**: Review changes/fixes
- ALL modifications must be inspected
- Error rate in modified code is higher than new code
- Group small changes for batch inspection

## Checklists

**Load `references/checklists.md` when conducting inspections.**

The checklists provide systematic prompts for finding common error types:
- Design inspection: Logic missing/wrong, interface issues
- Code inspection: Test/branch errors, interconnection errors
- Modified code: Change impact, regression concerns

## Process Details

**Load `references/inspection-phases.md` for complete phase documentation.**

Key insights:
- Preparation is individual, inspection is team
- 2-hour time limit is critical for efficiency
- Moderator training is essential
- Follow-up is mandatory, not optional
- Reinspection threshold: >5% rework

## Best Practices

### Critical Success Factors
1. **ONE objective at a time** - never mix finding and fixing
2. **Moderator training** - brief but essential
3. **2-hour time limit** - efficiency drops after this
4. **100% coverage** - no exceptions for "simple" code
5. **Scheduling discipline** - make time for inspection or it won't happen
6. **No performance reviews** - data is for programmer benefit only

### Common Pitfalls to Avoid
- Skipping preparation phase
- Letting meetings become debugging sessions
- Not following up on rework
- Using inspection data against programmers
- Exceeding 2-hour meeting duration
- Skipping "trivial" modifications

### Inspection vs. Walkthrough

| Aspect | Fagan Inspection | Walkthrough |
|--------|-----------------|-------------|
| Moderator training | Required | No |
| Defined roles | Yes | No |
| Who drives | Moderator | Code author |
| Checklists | Yes | No |
| Formal follow-up | Yes | No |
| Metrics tracked | Yes | No |
| Process improvement | Yes | No |

## Generating Reports

**Run `scripts/inspection_report.py --help` for report generation.**

### Report Types

The script generates four types of formal reports:

1. **Error List** (`error-list`) - Detailed list of all errors found
   - Each error with location, classification, description
   - Possible solutions if identified
   - Use for: Rework phase, tracking specific issues

2. **Module Detail** (`module-detail`) - Statistical analysis
   - Error counts by type, category, severity
   - Reinspection decision with rationale
   - Use for: Quality metrics, process improvement

3. **Inspection Summary** (`summary`) - Executive overview
   - Participants and roles
   - Size estimates and effort metrics
   - Error totals and quality metrics
   - Sign-off section for approval
   - Use for: Management reporting, audit trail

4. **JSON Data** (`json`) - Machine-readable export
   - Complete inspection data in structured format
   - Use for: Metrics aggregation, tool integration

### Usage Examples

```bash
# Interactive mode (prompts for all data)
python3 scripts/inspection_report.py --interactive

# Generate from JSON file
python3 scripts/inspection_report.py --from-file inspection_data.json

# Generate specific report type
python3 scripts/inspection_report.py --from-file data.json --report-type summary

# All reports to custom directory
python3 scripts/inspection_report.py --from-file data.json --output-dir ./reports
```

Reports must be completed within 24 hours of inspection meeting.

## Adapting for Modern Development

### For Pull Requests
- Author = Designer + Implementor
- Reviewer(s) = Moderator + team roles
- Use async preparation, sync discussion (or structured async)
- Apply checklists in review comments
- Track error types in review analytics

### For Solo Development
- Conduct self-inspection with checklists
- Focus on systematic coverage
- Track personal error patterns
- Use metrics to improve over time
- Consider pair programming as inspection variant

### For Agile Teams
- Inspection as definition of done
- Quick I₁ for design spikes
- I₂ for critical features
- Modified code inspection for bug fixes
- Sprint retrospective reviews error patterns

## Reference Materials

Load these on-demand during inspections:

- **references/checklists.md** - Systematic error-finding prompts
- **references/error-classification.md** - Complete error taxonomy
- **references/roles-responsibilities.md** - Detailed role descriptions
- **references/inspection-phases.md** - Complete 5-phase process guide
- **HOW-TO-USE.md** - Step-by-step usage guide with examples
- **QUICK-REFERENCE.md** - One-page quick reference card
- **examples/sample-inspection-session.md** - Complete example inspection with AI

## Scripts and Tools

- **scripts/inspection_report.py** - Generate formal inspection reports with error tracking and metrics

## References

Based on: Fagan, M.E. (1976). "Design and Code Inspections to Reduce Errors in Program Development." IBM Systems Journal, 15(3), 182-211.
