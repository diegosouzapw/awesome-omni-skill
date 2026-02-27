---
name: skill-creator
description: Create new skills following a structured 4-phase workflow. Use when you need to build reusable agent skills with proper validation.
---

# Skill Creator

A meta-skill for creating new skills using a structured 4-phase workflow. Based on SkillForge 4.0 principles but simplified for learning projects. This skill ensures quality, prevents duplicates, and promotes timeless, reusable skill designs.

## When to Use This Skill

Activate this skill in the following scenarios:

### Primary Use Cases
- **Creating new skills**: When you need to build a new skill for the research agent system
- **Formalizing workflows**: Converting recurring workflows or processes into reusable skills
- **Ensuring quality**: When skill quality and consistency matter through structured validation
- **Preventing duplicates**: Before creating a new skill, to verify no existing skill covers the need

### Trigger Phrases
- "Create a new skill for..."
- "Build a skill that..."
- "Formalize this workflow as a skill"
- "I need a reusable skill to..."
- "Make this process into a skill"

### When NOT to Use
- For one-off tasks that won't be repeated
- When an existing skill already covers the need
- For tasks too specific to generalize (Timelessness Score < 4)

## Four-Phase Workflow

The skill creation process follows four distinct phases. Each phase must complete before proceeding to the next.

```
Phase 0          Phase 1           Phase 2           Phase 3
Discovery   -->  Requirements  -->  Specification -->  Generation
& Triage         Analysis           Design             & Validation
```

---

### Phase 0: Discovery and Triage

**Purpose**: Determine if a new skill is needed or if existing skills can be used/improved.

#### Step 0.1: Search Existing Skills

Before creating any skill, search the existing skill repository:

```bash
python3 [SKILLS_DIR]/skill-creator/scripts/discover_skills.py "keyword or description"
```

Or manually explore:
```bash
ls -la [SKILLS_DIR]/
grep -r "description:" [SKILLS_DIR]/*/SKILL.md
```

#### Step 0.2: Triage Decision

Based on the search results, determine the appropriate action:

| Decision | Condition | Action |
|----------|-----------|--------|
| `CREATE_NEW` | No existing skill covers the need | Proceed to Phase 1 |
| `IMPROVE_EXISTING` | Existing skill partially covers the need | Enhance the existing skill |
| `USE_EXISTING` | Existing skill fully covers the need | Use the existing skill as-is |

#### Step 0.3: Document Triage Rationale

Before proceeding, document your decision:

```markdown
## Triage Decision: [CREATE_NEW | IMPROVE_EXISTING | USE_EXISTING]

### Skills Searched
- [skill-1]: [relevance assessment]
- [skill-2]: [relevance assessment]

### Rationale
[Why the decision was made]

### Gap Analysis (if CREATE_NEW)
[What capability is missing from existing skills]
```

---

### Phase 1: Requirements Analysis

**Purpose**: Deeply understand the skill requirements through structured analysis.

#### Step 1.1: Extract Requirements

Identify both explicit and implicit requirements from the request:

**Explicit Requirements** (directly stated):
- What the skill must do
- Expected inputs and outputs
- Specific constraints mentioned

**Implicit Requirements** (inferred):
- Error handling needs
- Edge cases to consider
- Integration requirements
- User experience expectations

#### Step 1.2: Apply Five Thinking Models

Analyze the skill through five complementary lenses:

##### 1. First Principles Thinking
> "What is the fundamental problem this skill solves?"

- Strip away assumptions
- Identify the core value proposition
- Question existing approaches

**Template**:
```
Core Problem: [What fundamental need does this address?]
Essential Components: [What absolutely must be included?]
Assumptions Challenged: [What "obvious" things might be wrong?]
```

##### 2. Systems Thinking
> "How does this skill fit into the larger ecosystem?"

- Map interactions with other skills
- Identify dependencies and dependents
- Consider emergent behaviors

**Template**:
```
Upstream Dependencies: [What this skill needs]
Downstream Consumers: [What might use this skill]
System Integration Points: [How it connects to the broader system]
Potential Conflicts: [What might it interfere with?]
```

##### 3. Pre-Mortem Analysis
> "If this skill fails, what went wrong?"

Imagine the skill failed completely. Work backward to identify potential failure modes:

**Template**:
```
Failure Mode 1: [Description]
- Cause: [Why this might happen]
- Prevention: [How to avoid it]
- Detection: [How to notice early]

Failure Mode 2: [Description]
- Cause: [Why this might happen]
- Prevention: [How to avoid it]
- Detection: [How to notice early]
```

##### 4. Constraints Analysis
> "What limitations must this skill work within?"

Identify hard constraints (non-negotiable) and soft constraints (preferences):

**Template**:
```
Hard Constraints:
- [Constraint 1]: [Why it's mandatory]
- [Constraint 2]: [Why it's mandatory]

Soft Constraints:
- [Constraint 1]: [Why it's preferred]
- [Constraint 2]: [Why it's preferred]

Resource Limits:
- Time: [Execution time bounds]
- Tokens: [LLM token budget]
- External calls: [API limits]
```

##### 5. Pareto Principle (80/20)
> "What 20% of features delivers 80% of the value?"

Prioritize ruthlessly:

**Template**:
```
High-Value Features (MUST HAVE):
1. [Feature]: [Value delivered]
2. [Feature]: [Value delivered]

Nice-to-Have Features (COULD HAVE):
1. [Feature]: [Marginal value]
2. [Feature]: [Marginal value]

Cut Features (WON'T HAVE):
1. [Feature]: [Why cut]
```

#### Step 1.3: Assess Timelessness Score

Rate the skill's expected longevity on a 1-10 scale:

| Score | Assessment | Action |
|-------|------------|--------|
| 1-3 | Too specific/temporary | REJECT - Do not create skill |
| 4-5 | Needs abstraction | REVISE - Make more general |
| 6-7 | Good balance | APPROVE - Proceed with creation |
| 8-10 | Timeless pattern | EXEMPLARY - Document as template |

**Scoring Criteria**:

| Criterion | Low Score (1-3) | High Score (8-10) |
|-----------|-----------------|-------------------|
| Specificity | Tied to specific tool versions | Pattern-based, tool-agnostic |
| Reusability | Single project only | Cross-project applicable |
| Stability | Frequent changes expected | Core concepts unlikely to change |
| Scope | Narrow edge case | Common, recurring need |
| Dependencies | Many external dependencies | Minimal dependencies |

**Assessment Template**:
```markdown
## Timelessness Assessment

| Criterion | Score (1-10) | Rationale |
|-----------|--------------|-----------|
| Specificity | [N] | [Why] |
| Reusability | [N] | [Why] |
| Stability | [N] | [Why] |
| Scope | [N] | [Why] |
| Dependencies | [N] | [Why] |

**Average Score**: [N]
**Decision**: [REJECT | REVISE | APPROVE | EXEMPLARY]
```

---

### Phase 2: Specification

**Purpose**: Define the complete skill specification before implementation.

#### Step 2.1: Define YAML Frontmatter

Create the skill metadata:

```yaml
---
name: [lowercase-hyphen-case]
description: [Action-oriented description under 1024 chars, no angle brackets]
---
```

**Naming Rules**:
- Lowercase letters, numbers, and hyphens only
- Maximum 64 characters
- Must be unique across all skills
- Should be descriptive but concise

**Description Rules**:
- Start with an action verb
- Maximum 1024 characters
- No angle brackets (`<>`)
- Include primary trigger condition

#### Step 2.2: Structure Sections

Plan the skill documentation structure:

```markdown
# [Skill Name]

[Brief overview paragraph]

## When to Use This Skill
[Scenarios and trigger conditions]

## How to Use
[Step-by-step usage instructions]

### Basic Usage
[Minimal example]

### Advanced Usage
[Complex scenarios]

## Examples
[Practical examples with expected outputs]

## Best Practices
[Guidelines for effective use]

## Troubleshooting
[Common issues and solutions]
```

#### Step 2.3: Define Trigger Phrases

Identify 3-5 natural language phrases that should activate this skill:

```markdown
### Trigger Phrases
1. "[Primary trigger phrase]"
2. "[Alternative phrasing]"
3. "[Another common request format]"
4. "[Domain-specific terminology]"
5. "[Abbreviated/shorthand version]"
```

#### Step 2.4: Determine Script Requirements

Assess whether the skill needs supporting scripts:

| Skill Type | Scripts Needed | Examples |
|------------|----------------|----------|
| Process/Workflow | No | Synthesis, analysis methodologies |
| Tool Integration | Yes | API wrappers, search utilities |
| Validation/Testing | Yes | Linters, validators |
| Data Processing | Yes | Parsers, formatters |

If scripts are needed, specify:
- Script name and purpose
- Input parameters
- Output format
- Dependencies required

---

### Phase 3: Generation and Validation

**Purpose**: Create the skill artifacts and validate them.

#### Step 3.1: Generate SKILL.md

Create the skill file using the template:

```bash
# Create skill directory
mkdir -p [SKILLS_DIR]/[skill-name]

# Create SKILL.md
# (Use the structured template from Phase 2)
```

**SKILL.md Template**:

```markdown
---
name: [skill-name]
description: [Skill description]
---

# [Skill Title]

[Overview paragraph explaining what the skill does and its value]

## When to Use This Skill

[Detailed scenarios when this skill should be activated]

### Primary Use Cases
- **[Use case 1]**: [Description]
- **[Use case 2]**: [Description]
- **[Use case 3]**: [Description]

### Trigger Phrases
- "[Phrase 1]"
- "[Phrase 2]"
- "[Phrase 3]"

## How to Use

[Step-by-step instructions]

### Prerequisites
[Required setup or dependencies]

### Basic Usage
[Minimal working example]

### Step-by-Step Workflow

#### Step 1: [First Step]
[Instructions]

#### Step 2: [Second Step]
[Instructions]

#### Step 3: [Third Step]
[Instructions]

## Examples

### Example 1: [Simple Case]
[Detailed example with input and expected output]

### Example 2: [Complex Case]
[Detailed example showing advanced usage]

## Best Practices

1. **[Practice 1]**: [Explanation]
2. **[Practice 2]**: [Explanation]
3. **[Practice 3]**: [Explanation]

## Troubleshooting

### [Common Issue 1]
**Symptom**: [What the user observes]
**Solution**: [How to fix it]

### [Common Issue 2]
**Symptom**: [What the user observes]
**Solution**: [How to fix it]

## Related Skills
- [related-skill-1]: [How they relate]
- [related-skill-2]: [How they relate]
```

#### Step 3.2: Create Supporting Scripts (if needed)

```bash
mkdir -p [SKILLS_DIR]/[skill-name]/scripts

# Create script with proper structure
```

**Script Template**:

```python
#!/usr/bin/env python3
"""
[Script Name] - [Brief description]

Part of the [skill-name] skill.
"""

import argparse
import sys


def main():
    parser = argparse.ArgumentParser(
        description="[Script description]"
    )
    parser.add_argument(
        "input",
        help="[Input description]"
    )
    parser.add_argument(
        "--option",
        default="default",
        help="[Option description]"
    )

    args = parser.parse_args()

    # Implementation here
    result = process(args.input, args.option)
    print(result)


def process(input_value, option):
    """[Function description]"""
    # Implementation
    pass


if __name__ == "__main__":
    main()
```

#### Step 3.3: Run Validation

Validate the skill before deployment:

```bash
python3 [SKILLS_DIR]/skill-creator/scripts/validate_skill.py [SKILLS_DIR]/[skill-name]/SKILL.md
```

**Validation Rules**:

| Rule | Requirement | Error if Violated |
|------|-------------|-------------------|
| `name` | Lowercase, hyphen-case, ≤64 chars | "Invalid skill name format" |
| `description` | ≤1024 chars, no angle brackets | "Description exceeds limit or contains invalid characters" |
| "When to Use" | Section must exist | "Missing required section: When to Use" |
| "How to Use" | Section must exist | "Missing required section: How to Use" |
| Frontmatter | Valid YAML syntax | "Invalid YAML frontmatter" |

**Validation Output Example**:

```
Validating: [skill-name]/SKILL.md

[PASS] YAML frontmatter is valid
[PASS] Name format: skill-name (23 chars)
[PASS] Description length: 156 chars
[PASS] No angle brackets in description
[PASS] "When to Use" section found
[PASS] "How to Use" section found
[PASS] At least one example provided

Validation Result: PASSED
Skill is ready for use.
```

#### Step 3.4: Test the Skill

Before finalizing, manually test the skill:

1. **Trigger Test**: Verify trigger phrases activate the skill
2. **Workflow Test**: Follow the documented workflow end-to-end
3. **Edge Case Test**: Try boundary conditions
4. **Integration Test**: Verify interaction with related skills

---

## Complete Example Workflow

Here is a complete example of creating a simple skill using all four phases:

### Request
> "Create a skill for generating meeting summaries from transcripts"

### Phase 0: Discovery and Triage

```bash
# Search existing skills
python3 scripts/discover_skills.py "meeting summary transcript"
```

**Results**: No existing skills found for meeting summaries.

**Triage Decision**: `CREATE_NEW`

**Rationale**: The data-synthesis skill handles general synthesis but lacks meeting-specific structure (action items, attendees, decisions).

### Phase 1: Requirements Analysis

**Explicit Requirements**:
- Input: Meeting transcript (text)
- Output: Structured summary with key sections
- Must extract action items

**Implicit Requirements**:
- Handle various transcript formats
- Preserve speaker attribution
- Time-efficient processing

**Five Thinking Models Applied**:

1. **First Principles**: Core problem is information extraction and organization from unstructured text.

2. **Systems Thinking**: Integrates with data-synthesis for multi-meeting analysis; outputs feed into task management systems.

3. **Pre-Mortem**:
   - Failure: Missing action items due to informal language
   - Prevention: Provide examples of action item patterns

4. **Constraints**:
   - Hard: Must handle transcripts up to 50,000 words
   - Soft: Should complete in under 2 minutes

5. **Pareto (80/20)**:
   - MUST HAVE: Action items, key decisions, summary
   - COULD HAVE: Sentiment analysis, topic clustering
   - WON'T HAVE: Real-time transcription integration

**Timelessness Score**: 7/10 (APPROVE)
- Meetings are timeless
- Summary patterns are stable
- Transcript formats may vary (minor concern)

### Phase 2: Specification

```yaml
---
name: meeting-summarizer
description: Generate structured meeting summaries from transcripts. Use when you need to extract action items, decisions, and key discussion points from meeting records.
---
```

**Sections**: When to Use, How to Use, Summary Template, Examples, Best Practices

**Trigger Phrases**:
1. "Summarize this meeting transcript"
2. "Extract action items from this meeting"
3. "Create meeting notes from..."
4. "What were the key decisions in this meeting?"

**Scripts Needed**: None (LLM-native processing)

### Phase 3: Generation and Validation

**Create skill**:
```bash
mkdir -p [SKILLS_DIR]/meeting-summarizer
# Write SKILL.md content
```

**Validate**:
```bash
python3 scripts/validate_skill.py [SKILLS_DIR]/meeting-summarizer/SKILL.md
```

**Output**:
```
Validation Result: PASSED
Skill is ready for use.
```

---

## Available Scripts

### discover_skills.py

Search existing skills to prevent duplicates.

```bash
python3 [SKILLS_DIR]/skill-creator/scripts/discover_skills.py "search query"
```

**Options**:
| Option | Description |
|--------|-------------|
| `query` | Search term (required) |
| `--path` | Skills directory path (default: auto-detect) |
| `--verbose` | Show detailed match information |

### validate_skill.py

Validate skill structure and content.

```bash
python3 [SKILLS_DIR]/skill-creator/scripts/validate_skill.py path/to/SKILL.md
```

**Options**:
| Option | Description |
|--------|-------------|
| `skill_path` | Path to SKILL.md file (required) |
| `--strict` | Enable strict validation mode |
| `--fix` | Attempt to auto-fix minor issues |

---

## Best Practices

### 1. Always Start with Triage
Never skip Phase 0. Duplicate skills create confusion and maintenance burden.

### 2. Apply All Five Thinking Models
Each model reveals different aspects. Skipping any leaves blind spots.

### 3. Keep Skills Focused
Follow the Single Responsibility Principle. One skill should do one thing well.

### 4. Include Practical Examples
Abstract documentation without examples is hard to apply. Show, don't just tell.

### 5. Document the WHY, Not Just the HOW
Future maintainers need context. Explain reasoning behind design decisions.

### 6. Aim for Timelessness Score >= 6
If a skill scores below 6, abstract it further or reconsider if it should be a skill at all.

### 7. Test Before Deployment
Always validate and manually test before declaring a skill complete.

### 8. Consider Edge Cases
Document what the skill does NOT handle to set clear expectations.

### 9. Keep Dependencies Minimal
Skills with fewer dependencies are more portable and maintainable.

### 10. Write for Your Future Self
In six months, will you remember why you made certain choices? Document it.

---

## Validation Checklist

Before finalizing any skill, verify:

- [ ] Phase 0 triage completed and documented
- [ ] All five thinking models applied in Phase 1
- [ ] Timelessness Score >= 6
- [ ] YAML frontmatter is valid
- [ ] Name is lowercase, hyphen-case, ≤64 chars
- [ ] Description is ≤1024 chars with no angle brackets
- [ ] "When to Use" section exists and is comprehensive
- [ ] "How to Use" section exists with clear steps
- [ ] At least one practical example included
- [ ] Best practices documented
- [ ] Troubleshooting section addresses common issues
- [ ] validate_skill.py passes
- [ ] Manual testing completed

---

## Related Skills

- **data-synthesis**: For combining outputs from multiple skills
- **report-writing**: For documenting skill design decisions
- **academic-search**: Example of a well-structured skill with scripts

---

## Troubleshooting

### "Validation failed: Invalid skill name"
**Cause**: Name contains uppercase letters, spaces, or special characters.
**Solution**: Use only lowercase letters, numbers, and hyphens. Example: `my-skill-name`

### "Validation failed: Description too long"
**Cause**: Description exceeds 1024 characters.
**Solution**: Condense the description. Move detailed explanations to the skill body.

### "Timelessness Score too low"
**Cause**: Skill is too specific to current tools or temporary needs.
**Solution**: Abstract the skill to focus on patterns rather than implementations. If the score remains low, this may not be suitable as a reusable skill.

### "Cannot find existing skills"
**Cause**: discover_skills.py cannot locate the skills directory.
**Solution**: Specify the path explicitly with `--path [SKILLS_DIR]`

### "Skill duplicates existing functionality"
**Cause**: Similar skill already exists.
**Solution**: Consider using `IMPROVE_EXISTING` decision instead of creating a new skill.
