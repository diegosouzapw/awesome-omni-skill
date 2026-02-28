---
name: agent-creator
description: Comprehensive guide for creating high-quality specialized agents following v2 architecture patterns. Use this skill when users need to design and implement new agents, understand agent architecture, or learn best practices for agent creation.
license: MIT
---

# Agent Creator

**Purpose**: Teach the principles, patterns, and practices for creating high-quality specialized agents that follow v2 architecture standards.

**Critical Use Case**: This skill provides structured guidance for creating agents from requirements through deployment, preventing common mistakes and ensuring quality through automated validation.

**Differentiation from agent-hr-manager**:
- **agent-creator** (this skill) = Teaching guide, knowledge resource, passive reference 📖
- **agent-hr-manager** (agent) = Autonomous executor, active creator, can use this skill 👨‍🏫

Use agent-creator when learning how to create agents. Use agent-hr-manager when you want an agent automatically created.

---

## When to Use This Skill

Use agent-creator when:
- Creating a new specialized agent from scratch
- Learning agent architecture and design patterns
- Understanding quality validation (0-80 rubric)
- Troubleshooting agent quality issues
- Migrating agents to v2 architecture
- Training others on agent creation

Do NOT use for:
- Creating skills (use skill-creator skill instead)
- Quick agent modifications (just edit directly)
- General Claude usage questions

---

## 6-Step Agent Creation Workflow

### Step 0: Research Existing Patterns (BEFORE DESIGN)

**Objective**: Understand what already exists before creating something new. This prevents duplicate agents and ensures you leverage proven patterns.

**Why this matters**: Creating an agent without research leads to:
- Duplicating existing agent functionality
- Missing reusable patterns from similar agents
- Not discovering skills that solve part of the problem
- Reinventing methodology that already exists

**Actions**:

1. **Search for Similar Agents**:
   ```bash
   # List all available agents
   ls ~/.claude/agents/ | head -20

   # Search for agents in similar domain
   grep -l "[domain-keyword]" ~/.claude/agents/*.md 2>/dev/null
   ```

2. **Review Relevant Agent Examples**:
   - Read `references/agent-examples.md` for quality patterns
   - Study agents with high quality scores (60+/80)
   - Note phase structures that work for similar domains

3. **Check Skill Inventory**:
   ```bash
   # List available skills
   ls ~/.claude/skills/

   # Search for domain-relevant skills
   grep -r "[domain-keyword]" ~/.claude/skills/*/SKILL.md 2>/dev/null | head -10
   ```

4. **Decision Checkpoint** (REQUIRED):
   ```markdown
   | Question | Answer |
   |----------|--------|
   | Similar agent exists? | [yes/no - if yes, consider tuning instead] |
   | Relevant skills found? | [list skills to integrate] |
   | Reusable patterns identified? | [list patterns to follow] |
   | Proceed with new agent? | [yes with justification] |
   ```

5. **Research Novel Domains** (if unfamiliar):
   - Use WebSearch for domain best practices
   - Find authoritative sources and frameworks
   - Document key methodologies the agent should follow

**Deliverable**: Research summary documenting similar agents, skills to integrate, and justification for new agent.

---

### Step 1: Temporal Awareness & Requirements Gathering (CRITICAL)

**Objective**: Establish current date context and understand what the agent needs to do.

#### 1.1 Establish Temporal Context (REQUIRED)

**Why this matters**: Legal documents, contracts, compliance reports, and project documentation with incorrect dates create serious risks. The pizza baker contract bug (January 2025 vs November 2025) demonstrated this - wrong dates in legal documents can affect validity and compliance.

**Implementation**:
```markdown
## Phase 1: [Phase Name] & Temporal Awareness

**Objective**: [Phase goal]

**Actions**:
1. **Establish Temporal Context** (REQUIRED):
   ```bash
   CURRENT_DATE=$(date '+%Y-%m-%d')          # ISO 8601: 2025-11-06
   READABLE_DATE=$(date '+%B %d, %Y')        # Human: November 06, 2025
   TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S %Z') # Full: 2025-11-06 12:34:56 EET
   ```
   - Use CURRENT_DATE for document metadata, version numbers
   - Use READABLE_DATE for human-readable headers
   - Use TIMESTAMP for detailed audit trails

2. [Other Phase 1 actions...]

**Deliverable**: [Concrete output]
```

**Validation**: The validate_agent.py script checks for temporal awareness pattern in Phase 1.

#### 1.2 Gather Requirements

**Key Questions**:
1. **Problem Definition**: What problem does this agent solve?
2. **Domain Expertise**: What specialized knowledge is needed?
3. **Tool Requirements**: Which tools will it need? (Read, Write, Edit, Bash, Grep, Glob, etc.)
4. **Typical Workflow**: What is the step-by-step process?
5. **Success Metrics**: How do we know it worked?
6. **Edge Cases**: What unusual situations must it handle?

**Techniques**:
- **Example-Based**: Ask for 2-3 concrete usage examples
- **Anti-Pattern Analysis**: What should it NOT do?
- **Boundary Testing**: What are the limits (file size, complexity, scope)?

**Output**: Requirements document or clear mental model before proceeding.

---

### Step 1.5: Skill Discovery & Integration Planning

**Objective**: Identify which existing skills to integrate into the agent and how.

**Why this matters**: This skill moves beyond "prompt engineering" into "cognitive architecture" — ensuring the agent doesn't use a hammer for a screw. Proper skill integration gives agents specialized capabilities without reinventing them.

**Actions**:

1. **Map Requirements to Skill Categories**:
   ```markdown
   | Agent Requirement | Skill Category | Candidate Skills |
   |-------------------|----------------|------------------|
   | Debugging logic | Reasoning | hypothesis-elimination, self-reflecting-chain |
   | Security review | Development | security-analysis-skills, adversarial-reasoning |
   | Documentation | Documentation | document-writing-skills |
   | Database ops | Integration | chromadb-integration-skills |
   | Testing | Development | testing-methodology-skills |
   | Error handling | Development | error-handling-skills |
   ```

2. **Evaluate Each Candidate Skill**:
   ```markdown
   | Skill | Size | Active? | Integrate or Inline? |
   |-------|------|---------|---------------------|
   | [skill-name] | [lines] | [yes/no] | [integrate/inline/skip] |
   ```

   **Decision Criteria**:
   - **Integrate** if: Skill >100 lines, actively maintained, reusable
   - **Inline** if: Simple pattern <20 lines, agent-specific variant needed
   - **Skip** if: Not relevant after review

3. **Document Skills Integration**:
   ```markdown
   **Skills Integration**: skill-1, skill-2, skill-3
   ```
   This goes in the agent's header metadata.

4. **Plan Skill Invocation Points**:
   ```markdown
   | Phase | When to Invoke | Skill |
   |-------|----------------|-------|
   | Phase 2 | Complex decision | integrated-reasoning-v2 |
   | Phase 3 | Design validation | adversarial-reasoning |
   | Phase 4 | Error recovery | hypothesis-elimination |
   ```

5. **Check for Handover/Parallelism Needs**:
   - Will the agent need multi-pattern reasoning? → Add reasoning-handover-protocol
   - Will tasks run in parallel? → Add parallel-execution skill
   - See `cognitive-skills/INTEGRATION_GUIDE.md` for patterns

**Deliverable**: Skill integration plan with invocation points documented.

---

### Step 2: Architecture Design

**Objective**: Design the agent's phase structure, tool selection, and quality criteria.

#### 2.1 Determine Agent Complexity

**Decision Tree: Simple vs Complex Agent**

**Simple Agent** (3 phases, <200 lines):
- Single domain focus (e.g., PDF manipulation, CSV parsing)
- Linear workflow (no branching)
- Minimal state management
- Examples: pdf-creator-agent, code-formatter

**Complex Agent** (4-5 phases, 200-250 lines):
- Multiple operation modes (e.g., create, read, update)
- Conditional branching or decision trees
- State tracking across phases
- Examples: legal-agent, ceo-orchestrator, agent-hr-manager

**When to use integrated-reasoning-v2**: 8+ decision dimensions, strategic importance, >90% confidence required
- **9 patterns available**: ToT, BoT, SRC, HE, AR, DR, AT, RTR, NDF
- **11 scoring dimensions** for pattern selection
- See `cognitive-skills/INTEGRATION_GUIDE.md` for full integration patterns

#### 2.2 Design Phase Structure

**Guidelines** (from agent-design-patterns.md):
- **3-5 phases optimal** (2 too simple, 6+ too complex)
- Each phase has ONE clear objective
- Actions are SPECIFIC, not generic
- Deliverables are CONCRETE artifacts

**Phase Structure Template**:
```markdown
## Phase N: [Descriptive Name]

**Objective**: [One sentence describing the goal]

**Actions**:
1. [Specific action with tool: "Use Grep to search for X pattern in Y files"]
2. [Specific action with tool: "Use Edit to modify lines 45-52 in config.yml"]
3. [Specific action with condition: "If errors found, use TodoWrite to track fixes"]

**Deliverable**: [Concrete output: "List of 5 validated regex patterns with test cases"]
```

**Example from kaggle-leak-auditor**:
- Phase 1: Static Code Analysis → List of violations
- Phase 2: Runtime Validation → Validation results
- Phase 3: Report Generation → Audit report with recommendations

#### 2.3 Select Tools

**Common Tool Combinations**:
- **File analysis**: Read, Grep, Glob
- **Code modification**: Read, Edit, Write
- **Research**: WebSearch, WebFetch, Read
- **Execution**: Bash, TodoWrite, Read
- **Complex tasks**: Task (invoke other agents)

**Tool Selection Criteria**:
1. **Minimal set**: Only include tools actually used in phases
2. **Specific over general**: Edit > Write for modifications
3. **Composed workflows**: Grep to find, Read to analyze, Edit to modify

#### 2.4 Define Success Criteria (10-16 items)

**Categories**:
1. **Phase Deliverables** (3-5 items): "✅ Phase 1 violations list complete with severity scores"
2. **Quality Gates** (2-3 items): "✅ All findings validated with evidence"
3. **Confidence** (1 item): "✅ Confidence level >85% with clear reasoning"
4. **Documentation** (2-3 items): "✅ Report includes examples and references"
5. **Edge Cases** (2-3 items): "✅ Handled missing files gracefully"
6. **Temporal** (1 item): "✅ Document dated with current date"

**Format**:
```markdown
## Success Criteria

- ✅ Temporal awareness established in Phase 1
- ✅ Phase 1 deliverable: [specific output]
- ✅ Phase 2 deliverable: [specific output]
- ✅ All files created/modified successfully
- ✅ Quality validation passed with score ≥70/80
- ✅ Confidence level >85% with supporting evidence
- ✅ Edge cases documented and handled
- ✅ Reference documentation created (if using progressive disclosure)
[10-16 total items]
```

#### 2.5 Design Self-Critique (6-10 questions)

**Question Categories**:
1. **Completeness**: "Did I check all [domain-specific items]?"
2. **Confidence**: "What is my confidence level? Why?"
3. **Assumptions**: "What assumptions did I make?"
4. **False Positives**: "Could [finding X] be wrong? How?"
5. **False Negatives**: "What might I have missed?"
6. **Verification**: "How can user verify this?"
7. **Temporal**: "Did I use current date correctly?"

**Format**:
```markdown
## Self-Critique

1. **Domain Accuracy**: Did I correctly apply [domain] expertise?
2. **Tool Selection**: Did I use optimal tools for each task?
3. **Edge Cases**: Did I handle errors and failures gracefully?
4. **Temporal Accuracy**: Did I establish current date in Phase 1?
5. **Confidence Basis**: What evidence supports my confidence level?
6. **Assumptions**: What assumptions should the user validate?
[6-10 total questions]
```

#### 2.6 Define Confidence Thresholds

**Three-Tier System**:
```markdown
## Confidence Thresholds

- **High (85-95%)**: [Specific conditions: "All criteria met, deliverables complete, tests passed"]
- **Medium (70-84%)**: [Conditions: "Most criteria met, minor issues present, acceptable quality"]
- **Low (<70%)**: [Conditions: "Significant issues, incomplete work - continue working"]
```

**Domain-Specific Examples**:
- **Code analysis**: Based on test coverage, execution traces
- **Legal**: Based on citation verification, precedent alignment
- **Research**: Based on source quality, corroboration
- **Debugging**: Based on reproduction success, log evidence

---

### Step 3: Implementation

**Objective**: Write the agent definition file following v2 architecture.

#### 3.1 Create Agent Frontmatter

**Template**:
```yaml
---
name: agent-name
description: Clear one-sentence description. Use when [specific trigger conditions]. Examples: [concrete user questions].
tools: Read, Write, Edit, Bash, Grep, Glob, TodoWrite
model: claude-sonnet-4-5
color: blue
---
```

**Guidelines**:
- **name**: Hyphen-case (my-agent-name), <40 chars
- **description**: Include WHEN to use + example questions
- **tools**: Only list tools actually used in phases
- **model**: Usually claude-sonnet-4-5 (use opus for complex reasoning)
- **color**: blue/green/purple/gold/red for visual grouping

#### 3.2 Write Agent Opening

**Structure**:
```markdown
# Agent Name

**Purpose**: [1-2 sentences on what this agent does]

**Core Responsibilities**:
1. [Responsibility 1 with domain context]
2. [Responsibility 2 with domain context]
3. [Responsibility 3 with domain context]
[3-7 items total]

**Specialized Knowledge** (if applicable):
- Domain-specific terminology
- Technical constraints
- Industry standards
```

#### 3.3 Add Decision Tree (if multi-mode)

**When to include**: Agent operates in different modes or scenarios

**Template**:
```markdown
## Decision Tree: [What to Decide]

When tasked with [type of request], first determine the appropriate [mode/type]:

**Mode A** - Use when:
- [Condition 1]
- [Condition 2]
- User asks "[example question]"
→ Follow Phase 1A-2A workflow

**Mode B** - Use when:
- [Condition 1]
- [Condition 2]
- User asks "[example question]"
→ Follow Phase 1B-2B workflow
```

#### 3.4 Implement Phases (from Step 2.2)

**Critical**: First phase MUST include temporal awareness pattern.

#### 3.5 Add Success Criteria, Self-Critique, Confidence (from Step 2.4-2.6)

#### 3.6 Consider Progressive Disclosure

**When to extract to references**:
- Agent would exceed 250 lines with inline details
- Has extensive pattern catalogs (3+ detailed patterns)
- Includes large lookup tables or reference data
- Contains detailed code examples (>30 lines)

**What to extract**:
- Detailed code examples
- Technical deep-dives
- Edge case handling details
- Reference lookup tables

**Reference in main agent**:
```markdown
## Pattern Detection

**Reference Documentation**: `~/.claude/agents-library/refs/[agent]-patterns.md`

**Key patterns** (see reference for details):
1. Pattern A (CRITICAL)
2. Pattern B (WARNING)
3. Pattern C (INFO)
```

**Line Count Targets**:
- Main agent: 150-250 lines (ideal: 200)
- Reference docs: 200+ lines (no limit)

---

### Step 4: Quality Validation

**Objective**: Score agent quality using 0-80 rubric and iterate if needed.

#### 4.1 Use Automated Validation

**Run validate_agent.py**:
```bash
~/.claude/skills/agent-creator/scripts/validate_agent.py /path/to/agent.md
```

**Output**:
```
Quality Score: 72/80 (Excellent)

Phase Structure: 15/15 ✅
Success Criteria: 14/15 ⚠️  (Missing 1 criterion)
Self-Critique: 10/10 ✅
Progressive Disclosure: 8/10 ⚠️  (232 lines, close to limit)
Tool Usage: 10/10 ✅
Documentation: 5/10 ❌ (Missing examples)
Edge Case Handling: 10/10 ✅

Recommendations:
- Add 1 more success criterion (target: 10-16)
- Add usage examples for better documentation
```

**Scoring Rubric**:
- **70-80**: Excellent - production ready
- **60-69**: Good - minor improvements needed
- **50-59**: Fair - significant improvements needed
- **<50**: Poor - major refactoring required

See `references/quality-rubric-explained.md` for detailed breakdown.

#### 4.2 Manual Review Checklist

Even with automated scoring, manually verify:

- [ ] Temporal awareness in Phase 1 with REQUIRED label
- [ ] All tools in frontmatter are actually used in phases
- [ ] Success criteria are specific and measurable (not vague)
- [ ] Self-critique questions are domain-specific (not generic)
- [ ] Confidence thresholds have concrete conditions
- [ ] Examples demonstrate real usage (if included)
- [ ] No spelling errors in critical sections
- [ ] Markdown formatting is valid

#### 4.3 Iterate if Score <70

**Common improvements**:
- **Add edge case handling** (+10 pts): Document error conditions
- **Improve documentation** (+5-10 pts): Add examples, clarify instructions
- **Refine success criteria** (+3-5 pts): Make more specific and measurable
- **Progressive disclosure** (+5-10 pts): Extract details to references if >250 lines

**Iterate until score ≥70** or diminishing returns.

---

### Step 5: Deployment

**Objective**: Deploy agent to appropriate location(s) and verify availability.

#### 5.1 Determine Deployment Target(s)

**Global Library** (`~/.claude/agents-library/`):
- Persistent across all projects
- Available to all Claude Code instances
- Use for: Reusable agents (research, code formatting, validation)

**Local Project** (`.claude/agents/`):
- Project-specific
- Version controlled with project
- Use for: Domain-specific agents (this project's business logic)

**Both**: Deploy to global first, copy to local if project needs it

#### 5.2 Deploy Agent

**To Global Library**:
```bash
cp /path/to/my-agent.md ~/.claude/agents-library/my-agent.md
```

**To Local Project**:
```bash
cp /path/to/my-agent.md ./.claude/agents/my-agent.md
```

**With References**:
```bash
# Deploy agent
cp my-agent.md ~/.claude/agents-library/

# Deploy reference doc
cp my-agent-patterns.md ~/.claude/agents-library/refs/
```

#### 5.3 Verify Availability

**Restart Claude Code** to load new agent.

**Test invocation**:
```
"[Agent Name], help me with [typical task]"
```

**Check agent registry** (if using CEO orchestrator):
- Update CEO's worker agent registry if this is a new operational agent
- Add estimated duration based on similar agents

---

## Decision Trees

### Decision Tree 1: Create New Agent vs Extend Existing

**Create New Agent** when:
- New domain/expertise area (e.g., adding legal agent when only have code agents)
- Different tool requirements (e.g., new agent needs Bash, existing only uses Read/Write)
- Different phase structure (e.g., new agent has 5 phases, existing has 3)
- User explicitly requests new agent

**Extend Existing Agent** when:
- Same domain, just adding capabilities (e.g., PDF agent adding form-filling)
- Same tool set, similar workflow
- Agent currently <200 lines (room to grow)
- Change is backward compatible

**Create New + Deprecate Old** when:
- Fundamental architecture change (v1 → v2)
- Existing agent has quality score <40
- Existing agent >300 lines and unmaintainable

### Decision Tree 2: When to Use Cognitive Reasoning Patterns

**Use integrated-reasoning-v2** (meta-orchestrator) when:
- **8+ decision dimensions** (architecture, tools, phases, quality, deployment, etc.)
- **Strategic importance** (affects multiple projects, long-term impact)
- **Uncertain which reasoning pattern** is best for the problem

**Direct pattern selection** (skip meta-orchestrator):
- **Diagnosis/debugging** → Use hypothesis-elimination (HE)
- **Security review** → Use adversarial-reasoning (AR)
- **Trade-off resolution** → Use dialectical-reasoning (DR)
- **Novel problem** → Use analogical-transfer (AT)
- **Time pressure** → Use rapid-triage-reasoning (RTR)
- **Stakeholder coordination** → Use negotiated-decision-framework (NDF)
- **High confidence required** (>90%, mission-critical)
- **Complex trade-offs** (performance vs accuracy, simplicity vs power)

**Use tree-of-thoughts** when:
- Clear evaluation criteria exist
- Need single best solution
- Medium complexity (4-7 dimensions)

**Use breadth-of-thought** when:
- Solution space unknown
- Need to explore all options
- Multiple valid approaches

**Use self-reflecting-chain** when:
- Sequential dependencies
- Need step-by-step validation
- Logical reasoning with backtracking

**Use direct implementation** when:
- Simple agent (<3 phases)
- Well-understood domain
- Similar agents exist as templates

---

## Common Mistakes to Avoid

See `references/common-mistakes.md` for detailed analysis. Top 5 pitfalls:

### 1. Missing Temporal Awareness ❌
**Mistake**: Forgetting to check current date in Phase 1
**Impact**: Documents with wrong dates (legal/compliance risk)
**Fix**: Always include temporal awareness with REQUIRED label in Phase 1

### 2. Vague Success Criteria ❌
**Mistake**: "✅ Agent works correctly" (not measurable)
**Impact**: Can't validate agent actually succeeded
**Fix**: "✅ Generated report includes 5 sections: summary, findings, evidence, recommendations, confidence score"

### 3. Generic Self-Critique ❌
**Mistake**: "Did I do a good job?" (applies to everything)
**Impact**: Doesn't catch domain-specific errors
**Fix**: "Did I validate all legal citations against Finlex API?" (domain-specific)

### 4. Tool Overload ❌
**Mistake**: Listing 10+ tools in frontmatter when only 3 are used
**Impact**: Confusing, suggests agent does more than it does
**Fix**: Only list tools actually referenced in phase actions

### 5. No Edge Case Handling ❌
**Mistake**: Only implementing "happy path"
**Impact**: Agent fails on unexpected inputs, errors not handled gracefully
**Fix**: Add "Edge Cases" section, document what to do when things go wrong

---

## Using validate_agent.py

The validation script provides automated quality scoring:

**Basic Usage**:
```bash
~/.claude/skills/agent-creator/scripts/validate_agent.py ~/.claude/agents-library/my-agent.md
```

**Output Interpretation**:
- **70-80**: Ship it! Excellent quality
- **60-69**: Almost there, minor fixes
- **50-59**: Needs work, iterate
- **<50**: Major refactoring required

**What it checks**:
- Phase structure (3-5 phases, clear objectives, deliverables)
- Success criteria (10-16 items, specific)
- Self-critique (6-10 questions, domain-specific)
- Progressive disclosure (150-250 line target)
- Tool usage (tools in frontmatter match phase usage)
- Documentation (examples, references)
- Edge case handling (documented error scenarios)
- Temporal awareness (REQUIRED in Phase 1)

See `references/quality-rubric-explained.md` for scoring details.

---

## Reference Documentation

This skill includes detailed reference documentation:

**`references/agent-examples.md`**: Annotated examples of high-quality agents
- legal-agent (264 lines, progressive disclosure, 68/80 quality)
- ceo-orchestrator (244 lines, integrated-reasoning integration)
- agent-hr-manager (748 lines, meta-agent patterns)

**`references/quality-rubric-explained.md`**: Deep-dive on 0-80 scoring system
- Detailed breakdown of each category
- Examples of excellent vs poor implementations
- How to improve scores in each area

**`references/common-mistakes.md`**: Anti-pattern catalog
- 10 most common agent creation mistakes
- Real examples from production agents
- How to detect and fix each mistake

**`references/temporal-awareness-deep.md`**: Why temporal awareness matters
- Legal/compliance risks of wrong dates
- The pizza baker contract bug case study
- Implementation patterns and validation

---

## Quick Start Examples

### Example 1: Simple Agent (CSV to Markdown Converter)

**Requirements**: Convert CSV files to markdown tables

**Architecture**:
- 3 phases (Parse CSV → Format Table → Output Markdown)
- Tools: Read, Write, Bash
- <200 lines, no progressive disclosure needed

**Key Decisions**:
- Simple agent (linear workflow)
- No decision tree (single mode)
- Success criteria: 10 items
- Self-critique: 6 questions

**Implementation time**: ~20 minutes
**Expected quality score**: 63-70/80

### Example 2: Complex Agent (Multi-Language Legal Compliance Checker)

**Requirements**: Check code/documents for GDPR, Finnish, and EU law compliance

**Architecture**:
- 5 phases (Temporal + Scan → Finnish Law → EU Law → Cross-Reference → Report)
- Tools: Read, Bash, Grep, WebFetch, Task (for legal-agent)
- 220 lines with references/legal-patterns.md (150 lines)

**Key Decisions**:
- Complex agent (multi-jurisdiction)
- Decision tree (document type: code vs contracts vs policies)
- Success criteria: 14 items
- Self-critique: 8 questions
- Uses integrated-reasoning for cross-jurisdiction conflicts

**Implementation time**: ~2 hours
**Expected quality score**: 72-80/80

---

## Summary: 5-Step Workflow

1. **Temporal Awareness & Requirements** → Current date + clear problem definition
2. **Architecture Design** → Phases, tools, success criteria, self-critique, confidence
3. **Implementation** → Write agent following v2 patterns (150-250 lines)
4. **Quality Validation** → Score with validate_agent.py (target: ≥70/80)
5. **Deployment** → Copy to global library and/or local project

**Validation checkpoint**: Run validate_agent.py before deploying!

---

**Meta**: This skill was designed using integrated-reasoning (94% confidence) to synthesize patterns from agent-design-patterns.md and 17 production v2 agents.
