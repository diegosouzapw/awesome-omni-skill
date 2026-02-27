---
name: "governance"
description: "Shared governance framework across all phases. Decision-making authority, RACI matrix, approval workflows, change management, and escalation procedures. Warm, story-driven Expert Mentor style for 2026."
type: "shared"
used_by: ["all_phases"]
---

#  Governance Framework

---

##  What You'll Learn

By the time you finish this guide, you'll understand:

 How to make decisions fast (without bypassing necessary review)
 Who decides what (clear authority, no ambiguity)
 How to document decisions (traceability, audit trail)
 How to handle changes (without chaos or scope creep)
 How to escalate (when and how, without politics)

**Time Investment:** 35 minutes to read, a career to master
**Difficulty Level:** Medium (we explain everything in plain English)
**Emotional Difficulty:** Low (we replace frustration with clarity)

---

##  Quick Navigation

**New to Governance?** Start here:
- [Governance Principles](#-governance-principles-the-foundation) - The mindset that enables fast decisions
- [RACI Matrix](#-raci-matrix-who-does-what) - Who decides what

**Ready to Implement?** Jump to:
- [Approval Workflows](#-approval-workflows-how-decisions-are-made) - Your decision-making processes
- [Change Management](#-change-management-handling-change-without-chaos) - Handle changes without chaos

**Need Specifics?** Go to:
- [Decision Authority](#-decision-making-authority-who-decides-what) - Clear authority for every decision
- [Escalation](#-escalation-procedures-when-and-how-to-escalate) - When and how to escalate
- [Templates](#-templates-and-tools) - Tools to implement governance

---

##  Governance Principles: The Foundation

### Delegated Authority (Decisions Made at the Right Level)

**The Principle:** Decisions are pushed to the appropriate level (not all to executives).

**Why It Matters:**
- Faster decisions (no executive bottlenecks)
- Better decisions (decisions made by people with expertise)
- Empowered teams (ownership and accountability)

**Real-World Analogy:**
Think about a restaurant. The head chef decides what's on the menu. The line cooks decide how to prepare each dish. The servers decide how to serve customers. The restaurant owner doesn't decide every little detail—they hire experts and trust them to decide.

**Anti-Pattern:** 🚫
> "Every decision needs executive approval."

**Why It's a Problem:** Executives become bottlenecks. Decisions take weeks. Good people leave (they're not empowered).

**Better:** 
> "Technical decisions are made by technical leadership. Business decisions are made by product leadership. Executive decisions are made by executives."

---

### Parallel Approvals (Don't Serialize Everything)

**The Principle:** Risk-based approval tracks (faster for low-risk items, thorough for high-risk items).

**Why It Matters:**
- Fast approvals for low-risk items (don't over-govern)
- Thorough approvals for high-risk items (appropriate review)
- Emergency process for critical items (when speed is essential)

**Approval Tiers:**

| Tier | Risk Level | Approvers | Timeline |
|------|------------|-----------|----------|
| **Tier 1** | Low impact, low risk | Tech Lead / PM | 24 hours |
| **Tier 2** | Medium impact, medium risk | Phase Owner | 1 week |
| **Tier 3** | High impact, high risk | Executive Sponsor | 2 weeks |
| **Emergency** | Critical (system down, security breach) | On-call engineer → Escalate as needed | Immediate |

---

### Transparency and Traceability (Every Decision Documented)

**The Principle:** All decisions are documented with clear audit trail.

**Why It Matters:**
- Audit trail (who decided what and why)
- Learning (we can look back and understand decisions)
- Reversibility (we can reverse decisions if needed)

**Decision Record Format:**

<details>
<summary><strong>📖 Decision Record Template</strong></summary>

```
Decision ID: DEC-2026-001
Date: 2026-01-15
Decision Maker: CTA (Jane Doe)
Status: Approved

**Decision Description:**
Use PostgreSQL as the primary database for the e-commerce platform.

**Rationale:**
- PostgreSQL has excellent JSON support (needed for product catalog)
- Strong ACID compliance (needed for transactions)
- Open-source (no licensing costs)
- Team has PostgreSQL experience (lower risk)
- Cloud provider has managed PostgreSQL (operational benefits)

**Alternatives Considered:**
1. MySQL: Rejected due to weaker JSON support
2. MongoDB: Rejected due to weaker transaction support
3. DynamoDB: Rejected due to cost and learning curve

**Impact:**
- Positive: Lower risk, faster development, lower cost
- Negative: Vendor lock-in to PostgreSQL (mitigated by using standard SQL)
- Effort: Medium (team knows PostgreSQL, but needs to learn advanced features)

**Approvals:**
- CTA: Jane Doe (Approved)
- Tech Lead: John Smith (Consulted)
- Security Architect: Sarah Johnson (Consulted - approved from security perspective)

**Review Date:** 2026-07-15 (6 months)
```

</details>

---

##  RACI Matrix: Who Does What

### RACI Definitions

| Letter | Meaning | Responsibility | Key Points |
|--------|----------|----------------|------------|
| **R** | Responsible | Does the work | Can be multiple people |
| **A** | Accountable | Owns the decision | ONLY ONE PERSON (this is critical) |
| **C** | Consulted | Provides input | Two-way communication |
| **I** | Informed | Notified of results | One-way communication |

**Critical Rule:** Only ONE "A" (Accountable) per activity. If two people are accountable, no one is accountable.

---

### RACI by Phase and Activity

<details>
<summary><strong>📖 Full RACI Matrix (All Phases and Activities)</strong></summary>

| Phase | Activity | Product Owner | PM | CTA | Tech Lead | Security | Compliance | Exec Sponsor |
|-------|----------|--------------|----|-----|-----------|----------|------------|--------------|
| **P1** | Business Case | A | C | C | I | I | I | A |
| **P1** | PRD | A | C | C | I | I | I | I |
| **P1** | Go/No-Go | R | C | C | I | I | I | A |
| **P2** | Requirements | A | C | I | C | I | C | I |
| **P2** | NFRs | I | I | A | C | I | I | I |
| **P2** | Security Reqs | I | I | C | I | A | C | I |
| **P2** | Compliance Reqs | I | I | I | I | C | A | I |
| **P2** | Go/No-Go | R | C | C | I | I | I | A |
| **P3** | Architecture | I | C | A | C | C | I | I |
| **P3** | Security Arch | I | C | C | I | A | C | I |
| **P3** | Compliance Arch | I | C | I | I | C | A | I |
| **P3** | Go/No-Go | R | C | A | I | I | I | I |
| **P4** | WBS | C | A | C | C | I | I | I |
| **P4** | Resources | C | A | C | C | I | I | A |
| **P4** | Go/No-Go | R | A | C | C | I | I | I |
| **P5** | Code | I | I | I | A | C | I | I |
| **P5** | Code Review | I | I | I | A | C | I | I |
| **P6** | Testing | C | I | I | C | I | C | I |
| **P6** | UAT Sign-off | A | I | I | I | I | I | A |
| **P7** | Deploy | I | I | I | C | C | I | I |
| **P8** | Operations | I | I | I | C | C | C | I |

**Legend:**
- **A:** Accountable (only ONE person owns the decision)
- **R:** Responsible (does the work)
- **C:** Consulted (provides input, two-way communication)
- **I:** Informed (notified of results, one-way communication)

</details>

---

## 🔄 Approval Workflows: How Decisions Are Made

### Go/No-Go Decisions (End of Each Phase)

**Approval Chain:**

1. **Phase Owner prepares recommendation** (R - Responsible)
2. **Secondary Owner reviews** (C - Consulted)
3. **Key stakeholders review** (C - Consulted)
4. **Executive Sponsor decides** (A - Accountable - ONLY ONE)

**Timeline:** 1-2 weeks for standard process
**Emergency:** 24-hour approval for critical issues

**Go/No-Go Criteria:**

<details>
<summary><strong>📖 Go/No-Go Decision Criteria</strong></summary>

**Phase 1 Go/No-Go:**
- ☐ Business case approved
- ☐ PRD complete and reviewed
- ☐ Technical feasibility confirmed
- ☐ Regulatory requirements identified
- ☐ Budget approved

**Phase 2 Go/No-Go:**
- ☐ All requirements approved
- ☐ Traceability matrix created
- ☐ Stakeholders sign off on requirements
- ☐ Security and compliance requirements defined
- ☐ Scope agreed upon

**Phase 3 Go/No-Go:**
- ☐ Architecture approved
- ☐ Threat models completed
- ☐ Security architecture approved
- ☐ Compliance architecture approved
- ☐ Performance targets defined
- ☐ Technology stack decisions finalized

**Phase 4 Go/No-Go:**
- ☐ WBS approved
- ☐ Resources allocated
- ☐ Budget finalized
- ☐ Sprint plan created
- ☐ Team onboarding complete

**Phase 5 Go/No-Go:**
- ☐ All features complete
- ☐ Unit tests passing (70-80% coverage)
- ☐ Code reviews completed
- ☐ CI/CD tests passing
- ☐ Technical debt addressed

**Phase 6 Go/No-Go:**
- ☐ All tests passing
- ☐ Performance meets SLAs
- ☐ Security validated
- ☐ Compliance validated
- ☐ UAT signed off
- ☐ No critical bugs outstanding

**Phase 7 Go/No-Go:**
- ☐ Pre-deployment tests passing
- ☐ Staging validation passing
- ☐ Rollback plan tested
- ☐ Monitoring configured
- ☐ On-call team ready

**Phase 8 Go/No-Go:**
- ☐ Production monitoring active
- ☐ SLAs being met
- ☐ Handover to operations complete
- ☐ Documentation complete
- ☐ Support team trained

</details>

---

### Code Approvals

**Approval Chain:**

1. **Developer self-review** (R - Responsible)
2. **Peers review** (C - Consulted)
3. **Tech Lead approves** (A - Accountable - ONLY ONE)

**Timeline:** Within 24-48 hours of submission

**Code Review Criteria:**
- Functionality (code does what it's supposed to do)
- Testing (tests included and passing)
- Code quality (readable, maintainable)
- Security (no obvious vulnerabilities)
- Performance (no obvious performance issues)
- Documentation (complex logic explained)

---

### Architecture Approvals

**Approval Chain:**

1. **CTA proposes architecture** (R - Responsible)
2. **Security Architect reviews** (C - Consulted)
3. **Tech Lead reviews** (C - Consulted)
4. **CTA approves** (A - Accountable - ONLY ONE)

**Timeline:** 1-2 weeks for major architectures

**Architecture Review Criteria:**
- Meets requirements (functional and non-functional)
- Scalability (can handle expected load)
- Security (includes necessary security controls)
- Compliance (includes necessary compliance controls)
- Operability (can be operated and maintained)
- Cost (within budget)
- Risk (acceptable risk level)

---

### Security Approvals

**Approval Chain:**

1. **Security Architect proposes** (R - Responsible)
2. **CISO reviews** (C - Consulted)
3. **CTA validates technical** (C - Consulted)
4. **CISO approves** (A - Accountable - ONLY ONE)

**Timeline:** 1 week for security designs

**Security Review Criteria:**
- Threat models completed
- Security controls defined
- Compliance requirements met
- Risk acceptable (or documented)
- Implementable (within team capability)

---

### Compliance Approvals

**Approval Chain:**

1. **Compliance Officer proposes** (R - Responsible)
2. **Legal reviews** (C - Consulted)
3. **CISO reviews security** (C - Consulted)
4. **Executive Sponsor approves** (A - Accountable - ONLY ONE)

**Timeline:** 2 weeks for compliance frameworks

**Compliance Review Criteria:**
- Regulatory requirements identified
- Compliance controls defined
- Gap analysis completed
- Remediation plan defined
- Risk acceptable (or documented)

---

## 🔄 Change Management: Handling Change Without Chaos

### Change Request Process

**Step 1: Submit Change Request**
- Document change (what is being requested)
- Document rationale (why is change needed)
- Document impact (how will this affect scope, schedule, budget, risk)

**Step 2: Impact Assessment**
- Analyze impact on scope (will this expand or shrink scope?)
- Analyze impact on schedule (will this delay or accelerate timeline?)
- Analyze impact on budget (will this increase or decrease cost?)
- Analyze impact on risk (will this increase or decrease risk?)

**Step 3: Review**
- Review by appropriate authority (based on impact tier)
- Stakeholder consultation (get input from affected parties)
- Risk assessment (understand the risks)

**Step 4: Approve/Reject**
- Decision on change request (approve, reject, defer)
- Document decision (why was this decision made?)
- Communicate decision (notify all stakeholders)

**Step 5: Update**
- Update plans (reflect approved change)
- Update documentation (keep everything in sync)
- Update traceability (link change to affected items)

**Step 6: Communicate**
- Notify stakeholders (everyone affected by the change)
- Update team (ensure everyone understands the change)
- Update stakeholders (keep leadership informed)

---

### Change Tiers

| Tier | Description | Approval | Timeline | Example |
|------|-------------|----------|----------|---------|
| **Tier 1** | Low impact, low risk | Tech Lead | 24 hours | Change variable name, update comment |
| **Tier 2** | Medium impact, medium risk | Project Manager | 1 week | Add new field to form, modify validation logic |
| **Tier 3** | High impact, high risk | Executive Sponsor | 2 weeks | Add new feature, change architecture, change timeline |

**Tier Determination:**

<details>
<summary><strong>📖 How to Determine Change Tier</strong></summary>

**Tier 1 (Low Impact, Low Risk):**
- Affects single component or feature
- No impact on timeline or budget
- No impact on other teams or stakeholders
- Reversible (can easily undo if needed)
- Low risk (won't cause significant issues if it goes wrong)

**Examples:**
- Bug fix (non-critical)
- UI tweak (doesn't affect functionality)
- Documentation update
- Performance optimization (doesn't change behavior)

**Tier 2 (Medium Impact, Medium Risk):**
- Affects multiple components or features
- Minor impact on timeline or budget (<10%)
- Some impact on other teams or stakeholders
- Reversible but with effort
- Medium risk (would cause noticeable issues if it goes wrong)

**Examples:**
- New feature (small to medium complexity)
- Change to API contract
- Database schema change
- Integration with new system

**Tier 3 (High Impact, High Risk):**
- Affects entire system or multiple systems
- Major impact on timeline or budget (>10%)
- Significant impact on other teams or stakeholders
- Difficult or impossible to reverse
- High risk (would cause significant issues if it goes wrong)

**Examples:**
- Architecture change
- Major feature (high complexity)
- Timeline change (delay or accelerate)
- Budget change (increase or decrease)
- Cancellation of project

</details>

---

##  Escalation Procedures: When and How to Escalate

### Escalation Levels

| Level | Trigger | Escalate To | Timeline | Example |
|-------|---------|---------------|----------|---------|
| **L1** | Issue blocking work | Tech Lead / PM | Immediate | Can't deploy due to bug |
| **L2** | Issue unresolved for 24h | Phase Owner | Within 24h | Architecture decision needed |
| **L3** | Issue unresolved for 1 week | Executive Sponsor | Within 1 week | Budget overrun, timeline slip |
| **L4** | Critical issue (system down, security breach) | Executive Sponsor + Crisis Team | Immediate | Production outage, data breach |

---

### Escalation Process

**Step 1: Identify Issue**
- Clearly define the issue (what is the problem?)
- Document impact (how is this blocking work?)
- Document attempts made (what have you tried already?)

**Step 2: Attempt Resolution**
- Try to resolve at current level first
- Document your attempts (what did you try?)
- Don't spend more time than appropriate (know when to escalate)

**Step 3: Document**
- Document issue (clear description)
- Document attempts (what you tried)
- Document impact (why this needs escalation)
- Document requested action (what do you need?)

**Step 4: Escalate**
- Escalate to next level with documentation
- Include all context (don't make them ask for details)
- Be clear about urgency (is this blocking everything?)

**Step 5: Follow Up**
- Ensure escalation is addressed (don't let it drop)
- Provide updates (keep everyone informed)
- Close the loop (confirm resolution)

**Escalation Template:**

<details>
<summary><strong>📖 Escalation Template</strong></summary>

```
ESCALATION: [Issue Title]

**Escalation Level:** L2 (Phase Owner)
**Date/Time:** 2026-01-15 14:30
**Escalated By:** Tech Lead (John Smith)
**Escalated To:** CTA (Jane Doe)

**Issue Description:**
Team cannot proceed with API integration due to unclear authentication requirements. The current requirements document doesn't specify whether to use OAuth 2.0 or API keys.

**Impact:**
- Blocking: 3 developers
- Timeline impact: 2 days delay so far
- Risk: Medium (will delay sprint completion)

**Attempts Made:**
1. Reviewed requirements document - no specification
2. Consulted with Product Owner - uncertain, deferred to CTA
3. Consulted with Security Architect - either option is acceptable from security perspective

**Requested Action:**
Decision on authentication approach (OAuth 2.0 vs API keys) by end of day 2026-01-15.

**Urgency:** High - blocking 3 developers, will cause sprint slip if not resolved today

**Follow-up:** Will follow up at 17:00 if no response

**Related Items:**
- Requirement: P2-REQ-045 (API Authentication)
- Architecture: P3-ARCH-012 (Integration Architecture)
```

</details>

---

##  Decision-Making Authority: Who Decides What

### Decision Categories and Owners

| Decision Category | Owner | Consulted | Informed | Example |
|-------------------|-------|-----------|----------|---------|
| **Business Case** | Executive Sponsor | PM, Finance | All | Should we build this? |
| **Product Requirements** | Product Owner | Users, Tech Lead | All | What should we build? |
| **Architecture** | CTA | Tech Lead, Security | All | How should we build it? |
| **Security** | CISO | Security Architect, Compliance | All | Is this secure enough? |
| **Compliance** | Compliance Officer | Legal, Security | All | Are we compliant? |
| **Technical** | Tech Lead | CTA, Developers | All | How do we implement this? |
| **Budget** | Executive Sponsor | PM, Finance | All | How much can we spend? |
| **Timeline** | Project Manager | Tech Lead, Product Owner | All | When will this be done? |
| **Quality** | QA Lead | Tech Lead, Product Owner | All | Is this good enough? |
| **Resources** | Project Manager | Tech Lead, HR | All | Who do we need? |
| **Go/No-Go (Phase)** | Executive Sponsor | Phase Owner, PM | All | Should we proceed? |

**Key Principles:**
- **Single Accountable:** Only one person is "A" (Accountable)
- **Appropriate Consultation:** Consult the right people (not everyone)
- **Timely Notification:** Inform the right people (not everyone)
- **Clear Documentation:** Document the decision (for traceability)

---

##  Documentation and Traceability

### Decision Records

**All significant decisions must be documented:**

**Format:**
- Decision ID (unique identifier)
- Date (when was decision made?)
- Decision Maker (who made the decision?)
- Decision Description (what was decided?)
- Rationale (why was this decision made?)
- Alternatives Considered (what other options were considered?)
- Impact (what is the impact of this decision?)
- Approvals (who approved this decision?)

**Storage:** Version controlled, accessible to all stakeholders

**Traceability Format:**

All artifacts follow the format: `P{N}-{SECTION}-###`

**Examples:**
- `P1-VISION-001`: Phase 1, Vision document, item 1
- `P3-ARCH-015`: Phase 3, Architecture, item 15
- `P5-CODE-427`: Phase 5, Code commit/story, item 427

**Traceability Chain:**
```
Epic → Feature → Story → Commit → Build → Artifact → Release → Test → Result
```

**Why This Matters:**
- **Traceability:** Can trace from requirement to implementation to test
- **Audit Trail:** Can see what was decided and why
- **Reversibility:** Can reverse decisions if needed
- **Learning:** Can learn from past decisions

---

## 💬 Stakeholder Communication

### Communication Cadence

| Stakeholder | Frequency | Format | Owner | Purpose |
|-------------|-----------|--------|-------|---------|
| **Executive Sponsor** | Monthly | Executive Summary | PM | High-level progress, risks, decisions |
| **Product Owner** | Weekly | Status Report | PM | Detailed progress, blockers, next steps |
| **Tech Team** | Daily | Standup | Tech Lead | What did you do, what will you do, blockers |
| **Users** | Per Sprint | Demo | Product Owner | See what's being built, provide feedback |
| **Security** | Weekly | Security Report | Security Lead | Security status, vulnerabilities, incidents |
| **Compliance** | Monthly | Compliance Report | Compliance Officer | Compliance status, gaps, remediation |

**Status Report Content:**
- Progress against plan (are we on track?)
- Milestones achieved (what did we accomplish?)
- Risks and issues (what's blocking us?)
- Next steps (what are we working on?)
- Decisions required (what decisions do you need to make?)

**Format:** Standard template, distributed via email/dashboard

---

##  Governance Tools

### Tool Recommendations by Category

| Category | Tools | Cost | Best For |
|----------|-------|------|----------|
| **Project Management** | Jira, Azure DevOps, Asana | $10-$20/user/month | Tracking work, decisions, issues |
| **Documentation** | Confluence, Notion, SharePoint | $5-$15/user/month | Documentation, decision records |
| **Decision Tracking** | Decision log, ADRs (Architecture Decision Records) | Free | Tracking decisions over time |
| **Approvals** | Jira workflows, email, sign-off sheets | Variable | Managing approval workflows |
| **Reporting** | Dashboards, status reports, exec summaries | Variable | Communicating status |
| **Auditing** | Audit logs, access logs, change logs | Variable | Audit trail for all activities |

---

##  Expected Outcomes

By following this governance framework, you will:

 **Make decisions fast** (clear authority, no bottlenecks)
 **Make good decisions** (right input from right people)
 **Document decisions** (traceability, audit trail)
 **Handle changes** (without chaos or scope creep)
 **Escalate effectively** (when and how, without politics)
 **Communicate clearly** (everyone knows what's happening)

**Governance is not about bureaucracy.** It's about clarity. When everyone knows who decides what, decisions happen fast and rework disappears.

---

## 💬 Final Thoughts

**Governance is an enabler, not a blocker.**

Good governance:
- Enables fast decisions (clear authority)
- Enables good decisions (right input, right people)
- Enables traceability (audit trail, learning)
- Enables agility (handle change without chaos)

Bad governance (bureaucracy):
- Slows decisions (too many approvals)
- Obscures accountability (no clear owner)
- Creates bottlenecks (decisions pile up)
- Encourages workarounds (people bypass governance)

**You don't need heavy governance.** You need smart governance.

**Start with RACI.** Clarify who decides what. Everything else builds on that.

**Remember:** Governance is about clarity. Clarity enables speed. Speed enables value.

---

##  Resources and Further Learning

### Free Resources

**Learning:**
- **RACI:** [wikipedia.org/wiki/RACI_matrix](https://en.wikipedia.org/wiki/RACI_matrix) - RACI explained
- **Architecture Decision Records:** [adr.github.io](https://adr.github.io) - ADR format and examples
- **Project Governance:** [pmi.org](https://www.pmi.org) - Project Management Institute resources

**Tools:**
- **Jira:** [atlassian.com/jira](https://www.atlassian.com/jira) - Project management
- **Confluence:** [atlassian.com/confluence](https://www.atlassian.com/confluence) - Documentation
- **Notion:** [notion.so](https://notion.so) - All-in-one workspace

**Communities:**
- **Project Management LinkedIn Groups:** Active communities
- **PMI:** Professional association for project management
- **Local PM meetups:** Networking and learning

---

##  Templates and Tools

See `./templates/` for:
- **Change Request Template** - Document change requests consistently
- **Decision Record Template** - Document decisions consistently
- **Escalation Template** - Escalate effectively
- **Status Report Template** - Communicate status consistently

---

##  Troubleshooting Governance Issues

| Issue | Symptom | Resolution |
|-------|----------|------------|
| **Bottleneck** | Decisions delayed | Escalate to executive, adjust authority (push decisions down) |
| **Unclear Authority** | Who decides? | Consult RACI matrix (clarify who's accountable) |
| **Poor Communication** | Stakeholders unaware | Improve communication cadence (more frequent updates) |
| **Scope Creep** | Requirements expanding | Enforce change management (all changes go through process) |
| **Decision Reversal** | Same decision made multiple times | Improve decision documentation (decision records, ADRs) |
| **Slow Approvals** | Approvals take too long | Streamline approval workflows (remove unnecessary approvers) |

---

**This shared skill is referenced by all phase skills.**

---

**Transformed by:** OCTALUME EXPERT MENTOR
**Transformation:** Complete rewrite to Expert Mentor style (warm, story-driven, emotionally intelligent, progressive disclosure, plain language, 2026 trends)

---

**Version 1.0.0 | OCTALUME Enterprise Lifecycle Framework**
