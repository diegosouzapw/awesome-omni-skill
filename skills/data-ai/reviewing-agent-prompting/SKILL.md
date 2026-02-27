---
name: reviewing-agent-prompting
description: "Review and improve prompts for coding agents. Use PROACTIVELY when auditing, checking, or evaluating agent instructions, system prompts, or task delegation text. Applies state-machine thinking to identify structural gaps and improve effectiveness."
---

<essential_principles>
**Why structure matters:** LLMs hallucinate when they have too much freedom and too little structure. Vague prompts force the model to guess intent, approach, tools, and completion criteria. Each guess compounds uncertainty.

**The core insight:** Treat prompts as state machines, not prose. Effective prompts have:
- Entry conditions (what triggers this phase)
- Actions (what to do)
- Exit criteria (how to know it's complete)
- Transitions (what comes next)

**The paradox:** Rigid structure creates reliable autonomy. Constraints clarify; they don't limit.
</essential_principles>

<intake>
**What would you like reviewed?**

Provide the prompt text you want analyzed. This could be:
- An agent system prompt
- A task delegation prompt
- Instructions for a coding assistant
- Any prompt meant to guide autonomous behavior

**Wait for the prompt text before proceeding.**
</intake>

<review_framework>
**Phase 1: Structural Analysis**

Check for these elements (present/absent/partial):

**1. Request Classification**
Does the prompt force categorization before action?

**Strong pattern:**
```
Before acting, classify the request:
- TYPE A (Simple): Single file, known location → direct action
- TYPE B (Complex): Unknown scope → explore first
- TYPE C (Research): External context needed → gather info
```

**Weak pattern:** No classification, jumps straight to action

**Why it matters:** Classification prevents wrong-strategy application. A quick lookup shouldn't trigger comprehensive analysis.

**2. Phase Gates with Exit Criteria**
Are there explicit phases with completion conditions?

**Strong pattern:**
```
## Phase 1: Assessment
- Check existing patterns
- Review conventions

**Exit criteria:** Know what patterns to follow before proceeding.

## Phase 2: Execution
...
```

**Weak pattern:** List of actions without phases or completion signals

**Why it matters:** Without gates, agents skip steps or thrash. Exit criteria tell the agent when to move on.

**3. Tool Constraints**
Are tool permissions explicit and minimal?

**Strong pattern:**
```
tools: Read, Grep, Glob (read-only, cannot modify)
```

**Weak pattern:** All tools available, or tools unspecified

**Why it matters:** Constraints clarify. An agent that can't write files knows its purpose is analysis.

**4. Mandatory Output Structure**
Is output format required, not suggested?

**Strong pattern:**
```
Every response MUST include:
<findings>...</findings>
<confidence>HIGH/MEDIUM/LOW</confidence>
<next_steps>...</next_steps>

Responses missing any section are incomplete.
```

**Weak pattern:** "Consider including..." or no format specified

**Why it matters:** Required sections force completeness. The agent can't hand-wave.

**5. Anti-Patterns (NEVER DO)**
Are failure modes explicitly closed off?

**Strong pattern:**
```
## NEVER DO
- Never suppress errors with workarounds
- Never commit without running tests
- Never guess file locations without searching
```

**Weak pattern:** Only positive instructions, no prohibitions

**Why it matters:** Negative examples are as important as positive ones. They close off common failure modes.

**6. Escalation Triggers**
Does the prompt define when to stop and ask for help?

**Strong pattern:**
```
Stop and escalate when:
- 3 consecutive attempts have failed
- Uncertainty exceeds 70%
- Change would affect more than 5 files
```

**Weak pattern:** No acknowledgment of limits

**Why it matters:** An agent that knows when to stop is more reliable than one that guesses forever.

**7. First Action Clarity**
Is the immediate first step crystal clear?

**Strong pattern:**
```
When invoked:
1. Run git diff to see recent changes
2. Read each modified file
3. Begin analysis
```

**Weak pattern:**
```
When invoked:
1. Think about the problem
2. Consider options
3. Do something helpful
```

**Why it matters:** Agents need concrete starting points, not vague intentions.
</review_framework>

<scoring_rubric>
**Structural Completeness Score**

| Element | Weight | Criteria |
|---------|--------|----------|
| Request Classification | 15% | Forces explicit categorization before action |
| Phase Gates | 20% | Clear phases with exit criteria |
| Tool Constraints | 10% | Explicit, minimal permissions |
| Output Structure | 15% | Mandatory format with required sections |
| Anti-Patterns | 15% | NEVER DO section with specific prohibitions |
| Escalation Triggers | 10% | Knows when to stop and ask |
| First Action | 15% | Concrete immediate step |

**Scoring:**
- 0: Element absent
- 1: Element present but weak
- 2: Element present and strong

**Interpretation:**
- 12-14: Production-ready prompt
- 8-11: Functional but could be more reliable
- 4-7: Significant gaps, expect inconsistent behavior
- 0-3: Essentially unstructured, high hallucination risk
</scoring_rubric>

<report_format>
**Agent Prompt Review**

**Summary**
[One sentence assessment]

**Structural Score: X/14**

| Element | Score | Notes |
|---------|-------|-------|
| Classification | 0/1/2 | ... |
| Phase Gates | 0/1/2 | ... |
| Tool Constraints | 0/1/2 | ... |
| Output Structure | 0/1/2 | ... |
| Anti-Patterns | 0/1/2 | ... |
| Escalation | 0/1/2 | ... |
| First Action | 0/1/2 | ... |

**Critical Issues**
[Elements scoring 0 that should be added]

**Improvements**
[Elements scoring 1 that could be strengthened]

**Strengths**
[Elements scoring 2, what's working well]

**Recommended Additions**
[Specific text to add, with examples]
</report_format>

<example_review>
**Input prompt:**
```
You are a helpful code assistant. Help the user with their code questions.
Be thorough and accurate.
```

**Review:**

### Summary
Minimal prompt with no structural elements. High hallucination risk.

### Structural Score: 1/14

| Element | Score | Notes |
|---------|-------|-------|
| Classification | 0 | No request typing |
| Phase Gates | 0 | No phases |
| Tool Constraints | 0 | Unspecified |
| Output Structure | 0 | No format required |
| Anti-Patterns | 0 | No prohibitions |
| Escalation | 0 | No limits defined |
| First Action | 1 | "Help" is vague but present |

**Critical Issues**
- No classification: Agent will apply same approach to simple lookups and complex research
- No phases: Agent may skip directly to answers without gathering context
- No output structure: Responses will be inconsistent

**Recommended Additions**

Add request classification:
```
Before responding, classify the request:
- LOOKUP: Specific fact or syntax → answer directly
- EXPLAIN: Understanding code → read and summarize
- DEBUG: Something broken → investigate systematically
- IMPLEMENT: Write new code → clarify requirements first
```

Add output structure:
```
Every response must include:
<answer>Direct response to the question</answer>
<confidence>HIGH/MEDIUM/LOW</confidence>
<evidence>Code references or documentation</evidence>
```

Add anti-patterns:
```
## NEVER DO
- Never guess without searching first
- Never provide code without testing syntax
- Never skip reading files you're asked about
```
</example_review>

<advanced_patterns>
**Additional Patterns to Check**

**Parallel Execution Requirements**
For tasks requiring thoroughness, does the prompt specify minimum parallel tool usage?

```
Minimum parallel calls by task type:
- Simple lookup: 2+ tools
- Code search: 3+ tools
- Research: 4+ tools

Cross-validate results. If tools disagree, investigate.
```

**Delegation Structure**
When delegating to other agents, does the prompt require structured handoff?

```
Delegation prompts MUST include:
1. TASK: What specifically needs done
2. EXPECTED OUTCOME: What success looks like
3. MUST DO: Non-negotiable requirements
4. MUST NOT DO: Explicit prohibitions
5. EXIT CRITERIA: How to know it's done
```

**Tool Cost Awareness**
Does the prompt guide efficient tool selection?

```
Tool allocation:
- FREE: grep/glob/read (use liberally)
- CHEAP: explore agent (use for unknown territory)
- EXPENSIVE: oracle agent (reserve for complex decisions)

Start FREE, escalate only when needed.
```

**Confidence Calibration**
Does the prompt require confidence assessment?

```
<confidence>
HIGH: Found definitive answer with evidence
MEDIUM: Found likely answer, some uncertainty
LOW: Best guess, recommend verification
</confidence>

If LOW, state what additional information would raise confidence.
```
</advanced_patterns>

<quick_checklist>
**Quick Review Checklist**

Before an agent prompt ships, verify:

- [ ] **Classification**: Does it categorize requests before acting?
- [ ] **Phases**: Are there explicit stages with exit criteria?
- [ ] **First action**: Is step 1 concrete and specific?
- [ ] **Output format**: Is structure required, not suggested?
- [ ] **NEVER DO**: Are common failure modes explicitly prohibited?
- [ ] **Escalation**: Does it know when to stop and ask?
- [ ] **Tools**: Are permissions explicit and minimal?

If more than 2 boxes unchecked, the prompt needs work.
</quick_checklist>
