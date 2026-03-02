---
name: argument-analysis
description: Analyze argument structure, identify logical gaps, suggest evidence needs, generate counterarguments, apply claim-evidence-warrant framework. Use when strengthening arguments, analyzing persuasive writing, checking logical validity, or when user asks to improve reasoning or logic.
---

# Argument Analysis

This skill provides systematic analysis of arguments to strengthen logic, identify gaps, and improve persuasiveness.

## Core Framework: Claim-Evidence-Warrant (CEW)

Every strong argument contains three elements:

### 1. Claim
The assertion you're making - what you want the reader to believe.

**Characteristics of strong claims**:
- Specific and falsifiable
- Not obviously true or universally accepted
- Worth arguing about
- Connected to evidence

**Weak claim**: "AI is important"
**Strong claim**: "Foundation models will consolidate around three major providers within 18 months"

### 2. Evidence
The data, examples, or facts that support your claim.

**Types of evidence** (strongest to weakest):
1. **Empirical data** - Studies, statistics, measurements
2. **Expert testimony** - Authoritative sources
3. **Case studies** - Specific examples with details
4. **Analogies** - Comparisons to similar situations
5. **Anecdotes** - Personal stories (weakest, but engaging)

**Evidence quality checklist**:
- [ ] Recent and relevant
- [ ] From credible source
- [ ] Specific (not vague generalities)
- [ ] Sufficient quantity
- [ ] Directly supports the claim

### 3. Warrant
The logical connection between evidence and claim - why the evidence proves the claim.

**Common warrant failures**:
- Assuming the connection is obvious when it isn't
- Jumping from evidence to claim without explanation
- Unstated assumptions that reader may not share

**Example with warrant**:
- **Claim**: "Remote work increases productivity"
- **Evidence**: "Microsoft's 2024 study showed 15% output increase"
- **Warrant**: "When employees control their environment and eliminate commute time, they can focus for longer uninterrupted periods, leading to measurable output gains"

## Analysis Process

When analyzing an argument, work through these steps:

### Step 1: Map the Argument Structure

Identify all claims in the piece:
1. Main thesis (central claim)
2. Supporting claims (sub-arguments)
3. Assumptions (unstated claims)

**Output format**:
```
Main Thesis: [statement]

Supporting Claims:
1. [claim 1]
2. [claim 2]
3. [claim 3]

Assumptions:
- [assumption 1]
- [assumption 2]
```

### Step 2: Check Each Claim for CEW Completeness

For each claim, verify:
- ✅ Claim is stated clearly
- ✅ Evidence is provided
- ✅ Warrant connects evidence to claim

**Flag gaps**:
- 🚩 Claim without evidence
- 🚩 Evidence without warrant
- 🚩 Weak or inappropriate evidence type
- 🚩 Warrant requires unstated assumptions

### Step 3: Identify Logical Gaps

Common gaps to look for:

#### Missing Evidence
- Claims asserted without support
- Vague references ("studies show", "experts say")
- Insufficient quantity of evidence

#### Weak Warrants
- Leap from evidence to claim without explanation
- Assumes reader shares unstated beliefs
- Connection is tenuous or requires multiple steps

#### Unstated Assumptions
- Premises taken for granted
- Cultural or contextual assumptions
- Value judgments presented as facts

#### Logical Fallacies
See [fallacies.md](fallacies.md) for complete list.

Most common:
- **False cause**: Correlation ≠ causation
- **Cherry-picking**: Selective evidence, ignoring counter-examples
- **Strawman**: Misrepresenting opposing view
- **Slippery slope**: Unwarranted chain of consequences
- **Appeal to authority**: Expert opinion outside their expertise
- **Hasty generalization**: Conclusion from too few examples

### Step 4: Generate Counterarguments (Steel-manning)

For the main thesis, construct the strongest possible counterargument:

1. **State the counter-claim** clearly
2. **Provide counter-evidence** (what would opposing side cite?)
3. **Identify unaddressed weaknesses** in original argument

**Purpose**: Not to defeat the argument, but to:
- Expose vulnerabilities that need addressing
- Strengthen the argument by anticipating objections
- Ensure claims are defensible

### Step 5: Suggest Improvements

For each identified gap, suggest specific fixes:

**Gap**: Claim without evidence
**Fix**: "Add [specific type of evidence needed]"

**Gap**: Weak warrant
**Fix**: "Explain why [evidence] supports [claim] by addressing [assumption]"

**Gap**: Logical fallacy
**Fix**: "Replace [fallacy] with [correct reasoning]"

## Output Format for Analysis

When analyzing a piece, use this structure:

```markdown
## Argument Structure Map

**Main Thesis**: [statement]

**Supporting Claims**:
1. [claim 1]
2. [claim 2]
3. [claim 3]

**Key Assumptions**:
- [assumption 1]
- [assumption 2]

---

## CEW Analysis

### Claim 1: [statement]
- **Evidence provided**: [Yes/No/Weak]
- **Evidence quality**: [assessment]
- **Warrant**: [Explicit/Implicit/Missing]
- **Gap**: [if any]
- **Suggested fix**: [specific action]

[Repeat for each claim]

---

## Logical Gaps & Fallacies

1. **[Line/paragraph reference]**: [Type of gap]
   - **Problem**: [description]
   - **Impact**: [why it weakens argument]
   - **Fix**: [specific suggestion]

---

## Steel-man Counterargument

**Counter-claim**: [strongest opposing view]

**Counter-evidence**: [what opponent would cite]

**Vulnerabilities in original**:
- [weakness 1]
- [weakness 2]

**How to address**:
- [specific recommendations]

---

## Evidence Needs

Research/sources needed to strengthen argument:
1. [specific evidence type] for [claim]
2. [specific evidence type] for [claim]

---

## Overall Assessment

**Strengths**:
- [what works well]

**Weaknesses**:
- [critical gaps]

**Priority fixes** (highest impact):
1. [fix 1]
2. [fix 2]
3. [fix 3]
```

## Rhetorical Analysis (Beyond Logic)

Arguments succeed through more than logic. Also assess:

### Ethos (Credibility)
- Does writer establish expertise?
- Are sources credible and cited?
- Is tone appropriate for audience?

### Pathos (Emotional Appeal)
- Are examples vivid and relatable?
- Does emotional appeal support (not replace) logic?
- Is audience's perspective considered?

### Kairos (Timing/Context)
- Is argument relevant to current moment?
- Does it address timely concerns?
- Is framing appropriate for context?

## Advanced Frameworks

For complex arguments, see:
- [frameworks.md](frameworks.md) - Toulmin model, Rogerian argument
- [fallacies.md](fallacies.md) - Complete fallacy reference

## Instructions for Claude

When using this skill:

1. **Always map argument structure first** - don't jump to critique
2. **Use CEW framework consistently** - every claim needs evidence and warrant
3. **Be specific in suggestions** - "add evidence" is too vague; specify what type
4. **Steel-man, don't strawman** - construct the strongest counterargument
5. **Prioritize gaps** - focus on highest-impact issues first
6. **Consider audience** - what assumptions can you make with this readership?
7. **Balance logic and rhetoric** - both matter for persuasiveness

**When analyzing vault content**:
- Reference house-rulebook principles
- Note pipeline stage (draft may have gaps that need TK tags)
- Suggest using `[TK: evidence needed]` for research gaps
- Consider whether piece is exploratory (looser logic OK) vs. persuasive (tight logic required)
