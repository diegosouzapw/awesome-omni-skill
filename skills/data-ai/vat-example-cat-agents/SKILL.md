---
name: vat-example-cat-agents
description: Comprehensive orchestration guide for Claude Code using the vat-example-cat-agents toolkit
---

# VAT Example Cat Agents

Comprehensive orchestration guide for Claude Code using the vat-example-cat-agents toolkit.

## Purpose: For Claude Code, Not the LLM

**Key distinction:**
- **This file** = Guidance for Claude Code (how to orchestrate agents)
- **Agent resources** = Content for the LLM (what to say/know)

This skill references agent resources via markdown links but doesn't duplicate LLM prompts.

## Agent Inventory

The vat-example-cat-agents package provides 8 agents across 4 archetypes:

### Pure Function Tools (2 agents)
- **haiku-validator** - Validates 5-7-5 syllable structure + kigo/kireji
- **name-validator** - Quirky characteristic-based validation

### One-Shot LLM Analyzers (4 agents)
- **photo-analyzer** - Vision LLM extracts characteristics from images
- **description-parser** - Text parsing extracts characteristics from descriptions
- **name-generator** - Creates characteristic-based cat names
- **haiku-generator** - Composes haikus about cats

### Conversational Assistant (1 agent)
- **breed-advisor** - Multi-turn breed selection through natural dialogue

### External Event Integrator (1 agent)
- **human-approval** - HITL approval gate (mockable)

## When to Use This Skill

Trigger this skill when:
- User wants help selecting a cat breed
- User has a cat photo and wants analysis
- User needs a cat name suggestion
- User wants a haiku about their cat
- User needs validation of generated content (names, haikus)
- User wants to combine multiple agents in a workflow

## High-Level Orchestration Patterns

### Pattern 1: Single Agent (Simple)

Use when user has a straightforward request that maps to one agent.

**Example workflows:**
- "What cat breed should I get?" → [breed-advisor](../agents/breed-advisor.md)
- "Analyze this cat photo" → [photo-analyzer](../agents/photo-analyzer.md)
- "Is this a valid haiku?" → haiku-validator (pure function)

### Pattern 2: Sequential Pipeline (Multi-Agent)

Use when output of one agent feeds into another.

**Example workflows:**
1. Photo → Characteristics → Name
   - [photo-analyzer](../agents/photo-analyzer.md) → [name-generator](../agents/name-generator.md)
2. Photo → Characteristics → Haiku
   - [photo-analyzer](../agents/photo-analyzer.md) → [haiku-generator](../agents/haiku-generator.md)
3. Description → Characteristics → Name → Validation
   - [description-parser](../agents/description-parser.md) → [name-generator](../agents/name-generator.md) → name-validator

### Pattern 3: Generate-Validate Loop (Iterative)

Use when generator produces content that needs validation with retry logic.

**Example workflows:**
1. Name generation with validation
   - Generate → Validate → If invalid, retry with feedback
   - Uses: [name-generator](../agents/name-generator.md) + name-validator
2. Haiku generation with validation
   - Generate → Validate → If invalid, retry
   - Uses: [haiku-generator](../agents/haiku-generator.md) + haiku-validator

**Orchestration tip:** Generator agents have NO knowledge of validation rules. This is intentional - forces iteration and tests feedback loops.

### Pattern 4: HITL Approval Gate (External Event)

Use when decision requires human judgment.

**Example workflows:**
1. Breed application approval
   - Gather info → Generate application → [human-approval](../agents/human-approval.md) → Process result
2. Name approval before finalization
   - Generate names → Present options → Human selects → Finalize

## Detailed Workflow Orchestration

### Workflow A: Breed Selection (Conversational)

**When:** User wants help finding the right cat breed.

**Agent:** [breed-advisor](../agents/breed-advisor.md)

**Orchestration strategy:**
1. **Phase 1 - Gathering** (see [Conversation Strategy](../agents/breed-advisor.md#conversation-strategy))
   - Collect ≥4 factors including music preference
   - ONE question at a time (don't bombard)
   - Extract factors after each turn
   - Monitor readiness: `factorsCollected >= 4 && musicPreference != null`

2. **Phase 2 - Recommendation**
   - Present 3-5 matched breeds
   - Allow exploration, questions, comparisons
   - Use conversational formatting (not data dump)

3. **Phase 3 - Selection**
   - Detect selection signals ("I'll take", "sounds good")
   - Conclude gracefully
   - Provide next steps or exit instruction

**Critical factor:** Music preference is the PRIMARY compatibility factor. Ask early, use as conversation anchor. See [Music Preference Insight](../agents/breed-advisor.md#music-preference-insight) for mappings.

**Reference implementation:** See [cat-breed-selection.md](./cat-breed-selection.md) for detailed breed selection orchestration.

### Workflow B: Photo Analysis Pipeline

**When:** User provides a cat photo and wants analysis, name, or haiku.

**Agents:** [photo-analyzer](../agents/photo-analyzer.md) → [name-generator](../agents/name-generator.md) OR [haiku-generator](../agents/haiku-generator.md)

**Orchestration strategy:**

```
Step 1: Analyze Photo
- Input: Image path/URL
- Agent: photo-analyzer
- Output: CatCharacteristics
- Note: Supports mock mode (reads EXIF) for testing

Step 2: Generate Content
- Input: CatCharacteristics from Step 1
- Agent: name-generator OR haiku-generator
- Output: NameSuggestion OR Haiku

Optional Step 3: Validate
- Input: Generated content + original characteristics
- Agent: name-validator OR haiku-validator (pure functions)
- Output: Validation result
- If invalid: Retry Step 2 with feedback
```

**Mockable behavior:** photo-analyzer reads EXIF metadata in mock mode. For production, set `mockable: false` to use real vision API.

### Workflow C: Text Description Pipeline

**When:** User describes a cat in text (no photo).

**Agents:** [description-parser](../agents/description-parser.md) → [name-generator](../agents/name-generator.md) OR [haiku-generator](../agents/haiku-generator.md)

**Orchestration strategy:**

```
Step 1: Parse Description
- Input: Text description
- Agent: description-parser
- Output: CatCharacteristics (same schema as photo-analyzer!)

Step 2-3: Same as Workflow B
- Multi-modal convergence: photo and text produce same schema
- Downstream agents (name-gen, haiku-gen) work with either
```

**Key insight:** Photo and text paths converge at `CatCharacteristics` schema. This enables multi-modal workflows without agent changes.

### Workflow D: Generate-Validate-Retry Loop

**When:** Generator produces content that must pass validation rules.

**Pattern:** Generator (LLM) + Validator (pure function) + Retry logic

**Implementation:**

```typescript
// Pseudo-code for orchestration
let attempts = 0;
const maxAttempts = 3;

while (attempts < maxAttempts) {
  // Generate content
  const suggestion = await generator(characteristics);

  // Validate (pure function, instant)
  const validation = validator(suggestion, characteristics);

  if (validation.status === 'valid') {
    return suggestion; // Success!
  }

  // Retry with feedback
  attempts++;
  // Optional: Adjust approach based on validation.reason
}

throw new Error('Could not generate valid content after 3 attempts');
```

**Why this pattern works:**
- Generator has NO knowledge of validation rules (isolated concerns)
- Validator is deterministic (pure function, fast)
- Feedback loop tests multi-turn orchestration
- ~60-70% initial rejection rate forces iteration

**Agents using this pattern:**
- [name-generator](../agents/name-generator.md) + name-validator
- [haiku-generator](../agents/haiku-generator.md) + haiku-validator

### Workflow E: HITL Approval Gate

**When:** Decision requires human judgment (compliance, taste, ethics).

**Agent:** [human-approval](../agents/human-approval.md)

**Orchestration strategy:**

```
Step 1: Prepare Request
- Gather all necessary context
- Format clearly for human reviewer
- Include relevant data (characteristics, generated content, reasoning)

Step 2: Emit Approval Request
- Agent: human-approval
- Input: Request payload
- Behavior: Blocks waiting for response
- Timeout: Configurable (default: 60s)

Step 3: Handle Response
- Approved: Continue workflow
- Rejected: Handle gracefully (retry, inform user, etc.)
- Timeout: Fall back to safe default or escalate
```

**Mockable behavior:** Set `mockable: true` to auto-approve without human (useful for testing).

**Real-world uses:**
- Breeding application approval
- Name selection from alternatives
- Content moderation decisions

## CLI Exposure for Pure Functions

Pure function tools (validators) can be exposed via CLI for direct invocation.

### Usage Pattern

```bash
# Pass JSON input, get JSON/YAML output
echo '{"name": "Mr. Whiskers", "characteristics": {...}}' | vat agents validate-name

# Output format (using 2>&1):
# [stdout - complete output]
# ---
# [stderr - error messages]
```

### Implementation Requirements

1. **Input:** JSON on stdin
2. **Output:** JSON or YAML on stdout (complete, flushed)
3. **Errors:** stderr (separated with `---` when using `2>&1`)
4. **Exit codes:** 0 = success, 1 = validation failure, 2 = error

### Sequence Example

```
[Start]
↓
Read stdin (JSON input)
↓
Parse and validate input schema
↓
Execute pure function
↓
Flush stdout with complete result
↓
Print "---" separator
↓
Flush stderr with any error messages
↓
Exit with appropriate code
```

### Benefits

- MCP can map to CLI calls (fast, stateless)
- No long-running processes for pure functions
- Clear separation of output vs errors
- Composable with other CLI tools

### Applicable Agents

- **name-validator** - `vat agents validate-name`
- **haiku-validator** - `vat agents validate-haiku`
- Future: Any pure function tool

## What Claude Code Does vs What Agents Do

### Claude Code's Role (This Skill):

**Orchestration:**
- Select which agent(s) to use
- Chain agents in workflows (pipelines)
- Manage state between agents (pass outputs as inputs)
- Handle retries and error recovery

**Monitoring:**
- Track conversation phase (gathering, recommendation, selection)
- Count validation attempts (retry limits)
- Detect completion signals (phase transitions)
- Monitor timeouts (HITL approvals)

**Decision Making:**
- When to transition between phases
- When to retry vs give up
- Which agent path to take (photo vs text)
- How to present results to user

### Agents' Role (Agent Resources):

**Pure Functions:**
- Execute deterministic logic (validation rules)
- Return results instantly
- No side effects, no state

**LLM Analyzers:**
- Extract structured data from unstructured input
- Apply domain knowledge to classification
- Generate creative content (names, haikus)

**Conversational Assistants:**
- Conduct natural dialogue
- Accumulate context over turns
- Make recommendations based on collected factors

**Event Integrators:**
- Emit events to external systems
- Block waiting for responses
- Handle timeouts and errors

**Key insight:** Agents are specialized, focused on their domain. Claude Code provides the glue, orchestration, and workflow logic.

## Content Separation Guidelines

### What Belongs in Agent Resources (LLM-Facing):

✅ System prompts and LLM instructions
✅ Domain knowledge (breed database, syllable counting rules, kigo lists)
✅ Extraction formats (JSON schemas, output templates)
✅ Examples for few-shot learning
✅ Natural language mappings
✅ Validation rules and constraints

**Location:** `resources/agents/*.md`

### What Belongs in Skills (Claude Code-Facing):

✅ When to trigger agents (user intent signals)
✅ How to chain agents (workflow patterns)
✅ What to monitor (readiness criteria, retry limits)
✅ Debugging guidance (common issues, pitfalls)
✅ Meta-strategy (why photo first, when to retry)
✅ CLI exposure patterns

**Location:** `resources/skills/*.md` (this file)

### Overlap (Reference, Don't Duplicate):

⚠️ Workflow structure (skill explains orchestration, agent implements)
⚠️ Phase transitions (skill monitors, agent executes)
⚠️ Validation criteria (skill manages retries, agent defines rules)

**Resolution:** Keep agent resources authoritative. Skills REFERENCE via links, don't duplicate.

## Common Pitfalls

### ❌ Don't: Call Agents Without Understanding Their Archetype

Each archetype has different behavior:
- Pure functions: Instant, deterministic
- LLM analyzers: Single call, non-deterministic
- Conversational: Multi-turn, stateful
- Event integrators: Blocking, timeout handling

### ✅ Do: Match Orchestration to Archetype

```
Pure function → Call directly, no retry needed
LLM analyzer → Call once, parse result
Conversational → Multi-turn loop with state
Event integrator → Emit, wait, handle timeout
```

### ❌ Don't: Skip Validation in Generate-Validate Loops

Validators exist for a reason. Don't bypass them or assume LLM output is always valid.

### ✅ Do: Embrace the Feedback Loop

```
Generate → Validate → If invalid, retry with feedback
```

This pattern tests real-world orchestration where first attempts often fail.

### ❌ Don't: Mix Mock and Production Modes Accidentally

Mock mode (EXIF metadata) is fast and free. Production mode (real APIs) is slow and expensive. Be explicit about which mode you're using.

### ✅ Do: Use Mock Mode for Development, Production for Deployment

```typescript
// Development/testing
const result = await analyzePhoto(path, { mockable: true });

// Production
const result = await analyzePhoto(path, { mockable: false });
```

### ❌ Don't: Bombard Users with Questions in Conversational Flows

One question at a time. Give users space to think and respond naturally.

### ✅ Do: Guide Conversation Gently

```
Bad: "Tell me your music, living space, activity level, grooming, family, and allergies"
Good: "What's your favorite type of music?" → [response] → "Great! Tell me about your living space..."
```

## Debugging: When Things Go Wrong

### Issue: Photo Analysis Fails

**Symptoms:** Vision API errors, unexpected characteristics

**Debug steps:**
1. Check if image path is valid
2. Verify image is actually a photo (not text, PDF, etc.)
3. Try mock mode first: `{ mockable: true }`
4. Check API key and rate limits for production mode
5. Verify EXIF metadata format if using mock mode

### Issue: Name Validation Always Fails

**Symptoms:** Name generator can't produce valid names after 3+ attempts

**Debug steps:**
1. Check validation rules: name-validator has quirky requirements
2. Review characteristics: Validation rules depend on cat traits
3. Verify characteristics schema: All required fields present?
4. Check feedback loop: Is validation.reason being used for retries?

**Common cause:** ~60-70% rejection rate is EXPECTED. This forces iteration and tests feedback loops.

### Issue: Haiku Validation Rejects Everything

**Symptoms:** Haiku generator struggles to meet 5-7-5 syllable structure

**Debug steps:**
1. Check syllable counting algorithm: May differ from LLM's internal counting
2. Verify kigo (seasonal word) presence: Required for valid haiku
3. Check kireji (cutting word) detection: Optional but improves score
4. Review haiku format: Must be exactly 3 lines

**Solution:** Allow multiple retry attempts (3-5). Haiku generation is hard.

### Issue: Conversation Stalls in Breed Selection

**Symptoms:** Agent doesn't transition to recommendations

**Debug steps:**
1. Check factor count: Need ≥4 factors collected
2. Verify music preference: REQUIRED for transition
3. Review conversation history: Are factors being extracted?
4. Check readiness criteria: Session state vs actual factors

**Solution:** Ensure extraction happens after each turn. Monitor `factorsCollected` metadata.

### Issue: HITL Approval Times Out

**Symptoms:** human-approval agent returns timeout status

**Debug steps:**
1. Check timeout value: Default 60s, may need adjustment
2. Verify event emission: Is request actually sent?
3. Check mock mode: Set `mockable: true` for testing
4. Review request format: Is it clear what needs approval?

**Solution:** For testing, use mock mode. For production, increase timeout or add retry logic.

## Agent Resource Links

### Pure Function Tools
- name-validator (no agent resource - pure TypeScript logic)
- haiku-validator (no agent resource - pure TypeScript logic)

### One-Shot LLM Analyzers
- [Photo Analyzer](../agents/photo-analyzer.md) - Vision analysis
- [Description Parser](../agents/description-parser.md) - Text parsing
- [Name Generator](../agents/name-generator.md) - Creative naming
- [Haiku Generator](../agents/haiku-generator.md) - Haiku composition

### Conversational Assistant
- [Breed Advisor](../agents/breed-advisor.md) - Multi-turn breed selection
  - [Welcome Message](../agents/breed-advisor.md#welcome-message)
  - [Music Preference Insight](../agents/breed-advisor.md#music-preference-insight)
  - [Factor Definitions](../agents/breed-advisor.md#factor-definitions)
  - [Conversation Strategy](../agents/breed-advisor.md#conversation-strategy)
  - [Factor Extraction Prompt](../agents/breed-advisor.md#factor-extraction-prompt)
  - [Transition Message](../agents/breed-advisor.md#transition-message)
  - [Recommendation Presentation](../agents/breed-advisor.md#recommendation-presentation-prompt)
  - [Selection Extraction](../agents/breed-advisor.md#selection-extraction-prompt)
  - [Conclusion Prompt](../agents/breed-advisor.md#conclusion-prompt)

### External Event Integrator
- [Human Approval](../agents/human-approval.md) - HITL approval gate

### Related Documentation
- [Cat Breed Selection Skill](./cat-breed-selection.md) - Detailed breed advisor orchestration
- [Package README](../../README.md) - Human-facing documentation
- [Package Structure](../../docs/structure.md) - Technical organization

## Success Criteria

Your orchestration is successful when:

**For Single Agents:**
- Agent called with valid input schema
- Output parsed and validated correctly
- Errors handled gracefully
- User receives clear, actionable results

**For Pipelines:**
- Data flows correctly between agents
- Schema compatibility maintained (CatCharacteristics convergence)
- Intermediate results stored appropriately
- Final output meets user's original intent

**For Loops:**
- Retry logic prevents infinite loops (max attempts)
- Feedback improves subsequent generations
- User informed of progress ("Generating... Attempt 2 of 3")
- Success achieved within reasonable attempts

**For Conversational Flows:**
- User feels heard, not interrogated
- Natural dialogue rhythm maintained
- Factors collected efficiently (4-6 turns typical)
- Recommendations feel personalized
- Selection confirmed clearly

**For HITL Workflows:**
- Request clearly formatted for human reviewer
- Timeout handling prevents indefinite blocking
- Approval/rejection handled appropriately
- User understands what happened

## Advanced Orchestration Patterns

### Pattern: Parallel Execution

When agents don't depend on each other, run them in parallel.

**Example:** Generate both name and haiku from same characteristics

```typescript
const [name, haiku] = await Promise.all([
  generateCatName(characteristics),
  generateCatHaiku(characteristics),
]);
```

**Benefits:** Faster execution, better user experience

### Pattern: Fallback Chain

When one agent fails, try alternatives.

**Example:** Photo analysis with text description fallback

```typescript
let characteristics;
try {
  characteristics = await analyzePhoto(imagePath);
} catch {
  // Fallback to text description
  const description = await getUserDescription();
  characteristics = await parseDescription(description);
}
```

**Benefits:** Resilience, better error handling

### Pattern: Conditional Routing

Choose agent path based on user input type.

**Example:** Multi-modal input handling

```typescript
if (input.type === 'image') {
  characteristics = await analyzePhoto(input.imagePath);
} else if (input.type === 'text') {
  characteristics = await parseDescription(input.description);
} else {
  // Conversational gathering
  characteristics = await breedAdvisor.gather();
}
```

**Benefits:** Flexible input handling, better UX

### Pattern: Staged Approval

Break complex workflows into approval stages.

**Example:** Multi-stage breeding application

```typescript
// Stage 1: Basic info approval
const basicApproval = await humanApproval({ stage: 'basic', data });
if (!basicApproval.approved) return;

// Stage 2: Detailed questionnaire
const detailApproval = await humanApproval({ stage: 'detail', data });
if (!detailApproval.approved) return;

// Stage 3: Final review
const finalApproval = await humanApproval({ stage: 'final', data });
```

**Benefits:** Checkpoints prevent wasted work, clearer decision points

## CLI Integration Examples

### Example 1: Validate Name via CLI

```bash
# Input
echo '{
  "name": "Mr. Whiskers",
  "characteristics": {
    "physical": {
      "furColor": "Orange",
      "furPattern": "Tabby",
      "size": "medium"
    },
    "behavioral": {
      "personality": ["Playful", "Curious"]
    }
  }
}' | vat agents validate-name

# Output (stdout)
{
  "status": "valid",
  "reason": "Name meets all quirky validation rules",
  "confidence": 0.95
}
---
# (stderr - empty if no errors)
```

### Example 2: Validate Haiku via CLI

```bash
# Input
echo '{
  "lines": [
    "Orange sunset fur",
    "Paws dance across the tatami",
    "Zen master purrs"
  ]
}' | vat agents validate-haiku

# Output (stdout)
{
  "status": "valid",
  "syllableCounts": [5, 7, 5],
  "hasKigo": true,
  "kigo": "sunset",
  "hasKireji": false,
  "errors": []
}
---
# (stderr - empty if no errors)
```

### Example 3: Error Handling

```bash
# Invalid input
echo '{"invalid": "schema"}' | vat agents validate-name

# Output
# (stdout - empty)
---
Error: Invalid input schema. Expected 'name' and 'characteristics' fields.
Schema validation failed:
  - Missing required field: name
  - Missing required field: characteristics
# (exit code: 2)
```

## Next Steps

Once you've successfully orchestrated agents:

1. **Explore Combinations** - Try chaining agents in new ways
2. **Add Custom Validation** - Create your own quirky validation rules
3. **Build New Workflows** - Combine agents for complex use cases
4. **Contribute Patterns** - Share successful orchestration strategies
5. **Package Skills** - Create distributable skill packages with pure functions

## Questions?

- **For orchestration patterns:** This file (SKILL.md)
- **For agent-specific details:** See individual agent resources
- **For implementation examples:** See examples/ directory
- **For archetype theory:** See package README.md
