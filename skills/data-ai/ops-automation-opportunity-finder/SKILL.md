---
name: ops-automation-opportunity-finder
description: Identify and evaluate automation opportunities in banking operations using structured assessment frameworks. Use when analyzing processes for RPA, intelligent automation, AI/ML, or straight-through processing potential across payments, lending, account servicing, compliance, and back-office functions.
---

# Ops Automation Opportunity Finder

## Overview

This skill produces structured automation opportunity assessments for banking operations. It evaluates processes against automation readiness criteria including volume, standardization, rule-based logic, error rates, and ROI potential. It covers RPA (Robotic Process Automation), intelligent document processing (IDP), AI/ML-driven decisioning, straight-through processing (STP), and workflow automation. Output supports business cases, technology roadmaps, and operational transformation programs.

## When to Use

- Conducting automation opportunity assessments across banking operations
- Evaluating specific processes for RPA, AI, or intelligent automation suitability
- Building business cases for automation investments with ROI projections
- Prioritizing an automation pipeline based on value, feasibility, and risk
- Assessing automation readiness (data quality, process maturity, system landscape)
- Supporting digital transformation and operations modernization programs
- Identifying quick wins vs. strategic automation investments

## Required Inputs

| Input | Description | Format |
|-------|-------------|--------|
| Process inventory | List of operational processes with descriptions | Process catalog |
| Volume data | Transaction/task volumes by process | Operations metrics |
| Effort data | FTE effort, time per task, manual steps | Time study/workforce data |
| Error data | Error rates, rework rates, exception frequencies | Quality metrics |
| System landscape | Applications used, integration capabilities, APIs | IT architecture |
| Cost data | Labor costs, error costs, processing costs | Finance data |
| Compliance constraints | Regulatory requirements affecting automation | Compliance mapping |

## Methodology

### Step 1: Catalog Candidate Processes

Build a comprehensive process inventory across operations:

| Domain | Process | Volume/Month | FTEs | Manual Steps | Systems | Error Rate |
|--------|---------|-------------|------|-------------|---------|------------|
| Payments | Wire initiation and release | [N] | [N] | [N] | [List] | [X%] |
| Payments | ACH return processing | [N] | [N] | [N] | [List] | [X%] |
| Payments | Check exception handling | [N] | [N] | [N] | [List] | [X%] |
| Lending | Loan document review | [N] | [N] | [N] | [List] | [X%] |
| Lending | Condition clearing | [N] | [N] | [N] | [List] | [X%] |
| Account services | Account opening data entry | [N] | [N] | [N] | [List] | [X%] |
| Account services | Address change processing | [N] | [N] | [N] | [List] | [X%] |
| Compliance | SAR narrative preparation | [N] | [N] | [N] | [List] | [X%] |
| Compliance | KYC document verification | [N] | [N] | [N] | [List] | [X%] |
| Reconciliation | GL reconciliation | [N] | [N] | [N] | [List] | [X%] |
| Reconciliation | Nostro/Vostro reconciliation | [N] | [N] | [N] | [List] | [X%] |

### Step 2: Assess Automation Suitability

Score each process against automation readiness criteria:

| Criterion | Weight | Score 1 (Low) | Score 3 (Medium) | Score 5 (High) |
|-----------|--------|--------------|------------------|----------------|
| **Volume** | 20% | <100/month | 100-1,000/month | >1,000/month |
| **Standardization** | 20% | Highly variable, many exceptions | Mostly standard, some exceptions | Highly standardized, few exceptions |
| **Rule-based logic** | 20% | Requires significant judgment | Mix of rules and judgment | Clearly defined business rules |
| **Digital inputs** | 15% | Paper-based, unstructured | Mix of digital and paper | Fully digital, structured data |
| **System stability** | 10% | Frequent changes, unstable | Occasional changes | Stable, well-documented |
| **Error impact** | 15% | Low impact errors | Moderate financial/customer impact | High financial/regulatory impact |

**Automation suitability score** = Σ (Weight × Score)

| Score Range | Suitability | Recommended Approach |
|------------|-------------|---------------------|
| 4.0-5.0 | **High** — Immediate candidate | RPA or STP; fast implementation |
| 3.0-3.9 | **Medium** — Good candidate with preparation | Intelligent automation; process redesign first |
| 2.0-2.9 | **Low-Medium** — Requires significant investment | AI/ML for unstructured; phased approach |
| 1.0-1.9 | **Low** — Not ready for automation | Process maturation needed before automation |

### Step 3: Select the Right Automation Technology

Match process characteristics to automation technology:

| Technology | Best For | Characteristics | Typical ROI Timeline |
|-----------|---------|-----------------|---------------------|
| **RPA** | High-volume, rule-based, multi-system data entry | Structured data, defined steps, stable UI | 6-12 months |
| **Intelligent Document Processing (IDP)** | Document-heavy processes (loans, KYC, correspondence) | Unstructured/semi-structured documents | 9-18 months |
| **Workflow automation** | Multi-step processes with approvals and routing | Sequential/parallel tasks, rule-based routing | 3-9 months |
| **AI/ML decisioning** | Pattern recognition, prediction, classification | Historical data, probabilistic outcomes | 12-24 months |
| **Straight-through processing (STP)** | End-to-end elimination of manual intervention | API integration, event-driven architecture | 12-24 months |
| **Chatbot/Virtual assistant** | Customer and employee inquiry resolution | FAQ, guided workflows, NLP | 6-12 months |
| **Process mining** | Process discovery, conformance checking, optimization | Event logs, process execution data | 3-6 months |

### Step 4: Calculate ROI and Business Case

For each automation candidate, quantify the business case:

**Cost savings calculation**:
| Component | Current State | Automated State | Savings |
|-----------|-------------|-----------------|---------|
| Labor (FTE equivalent) | [N FTEs × $X] | [N FTEs × $X] | [$X/yr] |
| Error/rework costs | [$X/yr] | [$X/yr] | [$X/yr] |
| Processing time | [X hrs/item] | [X hrs/item] | [X hrs saved] |
| Overtime/temp staff | [$X/yr] | [$X/yr] | [$X/yr] |
| **Total annual savings** | | | **[$X/yr]** |

**Investment required**:
| Component | Cost |
|-----------|------|
| Software licensing | [$X/yr] |
| Implementation (partner/internal) | [$X one-time] |
| Integration development | [$X one-time] |
| Change management/training | [$X one-time] |
| Ongoing maintenance | [$X/yr] |
| **Total first-year cost** | **[$X]** |
| **Total ongoing annual cost** | **[$X/yr]** |

**ROI metrics**:
- Net annual benefit: [Annual savings - ongoing cost]
- Payback period: [Total investment / net annual benefit] months
- 3-year NPV: [Calculated at institution's hurdle rate]
- IRR: [Internal rate of return]
- FTE capacity freed: [N FTEs redeployed to higher-value activities]

### Step 5: Assess Risk and Compliance Considerations

Evaluate automation risks specific to financial services:

| Risk Category | Considerations | Mitigation |
|--------------|---------------|------------|
| **Regulatory** | Does the process have regulatory requirements for human review? | Identify required human-in-the-loop checkpoints |
| **Model risk** | Does AI/ML automation create SR 11-7 model risk obligations? | Assess model risk classification, validation requirements |
| **Operational** | What happens when the automation fails? | Design fallback procedures, monitoring, alerting |
| **Data privacy** | Does the automation process PII or restricted data? | Apply data handling controls, encryption, access limits |
| **Audit trail** | Can automated decisions be explained and audited? | Ensure logging, explainability, record retention |
| **Change management** | How will staff and processes adapt? | Training, role redesign, communication plan |
| **Vendor risk** | Does the automation depend on third-party platforms? | Vendor due diligence, contractual protections, exit strategy |

### Step 6: Prioritize the Automation Pipeline

Rank opportunities using a value-feasibility matrix:

| Process | Value Score | Feasibility Score | Combined | Priority |
|---------|-----------|-------------------|----------|----------|
| [Process 1] | [1-5] | [1-5] | [Average] | [Rank] |
| [Process 2] | [1-5] | [1-5] | [Average] | [Rank] |

**Value score factors**: Annual savings, error reduction, customer experience improvement, strategic alignment
**Feasibility score factors**: Technical complexity, process maturity, data availability, organizational readiness, regulatory constraints

**Pipeline categorization**:
- **Quick wins** (High feasibility, moderate value): Implement in 0-6 months
- **Strategic bets** (High value, moderate feasibility): Plan and implement in 6-18 months
- **Low-hanging fruit** (Moderate both): Implement as capacity allows
- **Long-term vision** (High value, low feasibility): Requires process maturation first

### Step 7: Design the Implementation Roadmap

Structure the automation program in waves:

**Wave 1 — Foundation (0-6 months)**:
- Quick wins with proven RPA technology
- Process documentation and standardization
- Center of Excellence (CoE) establishment
- Governance framework and change management

**Wave 2 — Expansion (6-18 months)**:
- Intelligent automation (IDP, workflow)
- Cross-functional process automation
- Analytics and process mining integration
- Scaling infrastructure and monitoring

**Wave 3 — Transformation (18-36 months)**:
- AI/ML-driven decisioning and prediction
- End-to-end STP for target processes
- Customer-facing automation (onboarding, servicing)
- Continuous improvement and optimization

## Output Specification

```markdown
# Automation Opportunity Assessment: [Scope]

## Executive Summary
[Key findings: number of opportunities, total savings potential, recommended priorities]

## Process Inventory
[Catalog of evaluated processes with volumes, effort, and error rates]

## Automation Suitability Scores
| Process | Volume | Standardization | Rule-Based | Digital Input | System Stability | Error Impact | Total | Suitability |
|---------|--------|-----------------|------------|---------------|------------------|-------------|-------|-------------|
| [Process] | [1-5] | [1-5] | [1-5] | [1-5] | [1-5] | [1-5] | [X.X] | [High/Med/Low] |

## Top Opportunities
### [Opportunity 1]
- **Process**: [Name]
- **Technology**: [RPA/IDP/AI/STP]
- **Annual Savings**: [$X]
- **Investment**: [$X]
- **Payback**: [X months]
- **FTEs Freed**: [N]
- **Risk Level**: [Low/Medium/High]

## Prioritized Pipeline
[Value-feasibility matrix with categorization]

## Implementation Roadmap
[Three-wave implementation plan with milestones]

## Risk Assessment
[Regulatory, operational, and technology risks with mitigations]

## Recommendations
[Top 3-5 recommendations with supporting rationale]
```

## Analysis Framework

### Automation Maturity Assessment

Evaluate the institution's automation maturity:
- **Level 1 — Ad hoc**: Individual macros and scripts, no governance
- **Level 2 — Opportunistic**: Piloting RPA, initial CoE formation
- **Level 3 — Systematic**: Established CoE, pipeline management, scaling RPA
- **Level 4 — Intelligent**: Integrated intelligent automation, AI/ML in production
- **Level 5 — Autonomous**: Self-optimizing processes, minimal human intervention

### Process Mining Application

Before automating, mine the actual process execution:
- Discover the true process (not the documented process) from system event logs
- Identify process variants, rework loops, and bottlenecks
- Quantify the proportion of straight-through vs. exception processing
- Use conformance checking to identify deviations from standard process
- Prioritize automation of the dominant process variant (80% path)

### Human-in-the-Loop Design

For regulated processes requiring human oversight:
- Define which steps can be fully automated vs. human-reviewed
- Design exception routing for items outside automation confidence thresholds
- Implement sampling-based quality assurance of automated decisions
- Ensure explainability for AI/ML-driven decisions
- Maintain regulatory audit trail with clear attribution (human vs. automated)

## Examples

**Example 1 — RPA Quick Win**:
"Account maintenance address change process: 2,400 requests/month, currently requiring manual data entry across 3 systems (core banking, CRM, card system) taking an average of 8 minutes per request. 3.2 FTEs dedicated to this task. Error rate: 4.5% (wrong field, incomplete update). Automation suitability score: 4.6/5.0 (high volume, highly standardized, rule-based, digital input from online banking). Recommended technology: RPA bot with structured data extraction from the online banking request form. Expected results: 95% straight-through processing (2,280 automated/month), 0.1% error rate, 2.8 FTE capacity freed. Investment: $85K implementation + $24K/year licensing. Annual savings: $196K labor + $18K error remediation = $214K. Payback: 5.2 months."

**Example 2 — Intelligent Automation**:
"Loan document review and condition clearing: 800 loans/month, average 12 documents per loan, 45 minutes per loan for initial review. 8 FTEs. Error rate: 6% (missed conditions, incorrect classification). Automation suitability: 3.2/5.0 (moderate — semi-structured documents, some judgment required). Recommended technology: Intelligent Document Processing (IDP) with ML-based document classification and data extraction, combined with rules-based condition matching. Human-in-the-loop for low-confidence extractions (<85% confidence score). Expected results: 60% of documents auto-classified and extracted, reducing average review time to 18 minutes. 3.6 FTE capacity freed. Investment: $350K implementation + $120K/year platform. Annual savings: $295K labor + $42K error reduction = $337K. Payback: 14 months. Regulatory note: final loan approval decision remains with human underwriter per SR 11-7 model risk requirements."

## Guidelines

- Automate the process as-is only if it's well-designed; redesign before automating when the process is fundamentally flawed
- Start with high-volume, rule-based processes for initial automation to build confidence and capability
- Always design for exceptions; no process is 100% automatable, and exception handling must be planned
- Quantify both hard savings (FTE, error reduction) and soft benefits (speed, consistency, scalability)
- Regulatory requirements may mandate human oversight for certain decisions; identify these constraints early
- RPA is not a substitute for system integration; use APIs and STP for long-term architecture
- Monitor bot performance continuously; automation can fail silently and accumulate errors
- Consider the impact on staff (redeployment, upskilling, morale) in the business case
- AI/ML automations may trigger SR 11-7 model risk management requirements
- Maintain a centralized automation inventory with ownership, monitoring, and lifecycle management

## Validation Checklist

- [ ] Process inventory is comprehensive across all banking operations domains
- [ ] Automation suitability scoring uses consistent, weighted criteria
- [ ] Technology recommendation matches process characteristics
- [ ] ROI calculation includes all cost components (labor, error, investment, ongoing)
- [ ] Payback period and NPV are calculated at the institution's hurdle rate
- [ ] Regulatory and compliance constraints are identified for each opportunity
- [ ] Human-in-the-loop requirements are designed for regulated processes
- [ ] Pipeline is prioritized using value-feasibility framework
- [ ] Implementation roadmap is phased with realistic timelines
- [ ] Risk assessment covers regulatory, operational, vendor, and change management dimensions
