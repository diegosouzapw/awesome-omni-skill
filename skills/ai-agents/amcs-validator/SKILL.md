---
name: amcs-validator
description: Score composed artifacts against blueprint rubric. Evaluates hook density, singability, rhyme tightness, section completeness, and profanity compliance. Returns scores and issues list to determine if fix loop is needed. Use after COMPOSE to validate artifacts before rendering.
---

# AMCS Validator

Evaluates composed artifacts (lyrics, style, producer notes) against the blueprint's scoring rubric to determine if the composition meets quality thresholds. If scores fall below the threshold (min_total < 0.85), the workflow transitions to the FIX loop.

## When to Use

Invoke this skill after COMPOSE completes. This is the quality gate before RENDER.

## Input Contract

```yaml
inputs:
  - name: lyrics
    type: string
    required: true
    description: Complete lyrics with section markers
  - name: style
    type: amcs://schemas/style-1.0.json
    required: true
    description: Validated style specification
  - name: producer_notes
    type: amcs://schemas/producer-notes-1.0.json
    required: true
    description: Production arrangement and mix guidance
  - name: blueprint
    type: amcs://schemas/blueprint-1.0.json
    required: true
    description: Genre-specific rules and scoring rubric
  - name: composed_prompt
    type: amcs://schemas/composed-prompt-0.2.json
    required: false
    description: Optional composed prompt for additional validation
  - name: seed
    type: integer
    required: true
    description: Determinism seed (use seed+5 for this node)
```

## Output Contract

```yaml
outputs:
  - name: scores
    type: object
    description: |
      Scoring breakdown:
      - total: Weighted average score (0-1)
      - hook_density: Hook repetition score (0-1, target ≥ 0.7)
      - singability: Syllable/meter consistency (0-1, target ≥ 0.8)
      - rhyme_tightness: Rhyme scheme adherence (0-1, target ≥ 0.75)
      - section_completeness: Required sections present (0-1, target 1.0)
      - profanity_score: Policy compliance (0-1, target 1.0 for clean)
  - name: issues
    type: array[string]
    description: List of specific failures (e.g., "Low hook density: 0.5 (target 0.7)")
  - name: pass
    type: boolean
    description: True if scores meet rubric thresholds (total ≥ min_total)
```

## Determinism Requirements

- **Seed**: `run_seed + 5` (for any probabilistic scoring, if needed)
- **Temperature**: N/A (rule-based scoring, no LLM generation)
- **Top-p**: N/A
- **Retrieval**: None
- **Hashing**: Hash scores object for provenance

## Constraints & Policies

- **min_total threshold**: Default 0.85 (from blueprint.eval_rubric.thresholds.min_total)
- **Hook density target**: ≥ 0.7 (chorus hooks repeated, memorable)
- **Singability target**: ≥ 0.8 (consistent syllable counts, natural phrasing)
- **Rhyme tightness target**: ≥ 0.75 (rhyme scheme followed)
- **Section completeness target**: 1.0 (all required sections present)
- **Profanity target**: 1.0 for clean songs (0.9 if explicit allowed)
- **Rubric weights**: Must sum to 1.0, used for weighted total

## Implementation Guidance

### Step 1: Load Rubric from Blueprint

Extract rubric configuration:

```python
rubric = blueprint["eval_rubric"]
weights = rubric["weights"]
thresholds = rubric["thresholds"]
min_total = thresholds["min_total"]  # Default: 0.85
```

### Step 2: Evaluate Hook Density

**Definition**: Percentage of lines that contain hook phrases from the chorus.

**Algorithm**:
1. Extract chorus section from lyrics
2. Identify hook phrases (repeated phrases ≥ 3 words)
3. Count total lines across all sections
4. Count lines containing hook phrases
5. Score = (hook_lines / total_lines)

**Target**: ≥ 0.7

**Example**:
```
Chorus:
Family time is what we need  <-- hook
Love and joy in every deed   <-- hook

Verse:
Family time is what we need  <-- hook repeated
...

Score: 3 hook lines / 16 total lines = 0.1875 → LOW
```

**Issues**:
- If score < 0.7: `"Low hook density: {score:.2f} (target 0.7)"`

### Step 3: Evaluate Singability

**Definition**: Consistency of syllable counts and natural phrasing.

**Algorithm**:
1. For each section, extract lines
2. Count syllables per line (use pyphen or simple vowel counting)
3. Compute standard deviation of syllable counts within section
4. Score = 1.0 - (stddev / mean_syllables)
5. Average across all sections

**Target**: ≥ 0.8

**Example**:
```
Verse:
Gathering 'round on Christmas Eve (9 syllables)
The kids decorate, we all believe (9 syllables)
Family time is what we need (8 syllables)

Stddev = 0.47, Mean = 8.67
Score = 1.0 - (0.47 / 8.67) = 0.95 → PASS
```

**Issues**:
- If score < 0.8: `"Weak singability: {score:.2f} (target 0.8) - inconsistent syllable counts"`

### Step 4: Evaluate Rhyme Tightness

**Definition**: Adherence to the intended rhyme scheme (ABAB, AABB, etc.)

**Algorithm**:
1. Parse lyrics.constraints.rhyme_scheme from SDS (e.g., "ABAB")
2. Extract end words from each line in verse/chorus
3. Check phonetic similarity (use pronouncing library or simple suffix matching)
4. Score = (matching_rhymes / expected_rhymes)

**Target**: ≥ 0.75

**Example**:
```
Rhyme scheme: ABAB
Verse:
Gathering 'round on Christmas Eve (A)
The kids decorate, we all believe (B) ✓ rhymes with A
Family time is what we need (A) ✓ rhymes with A
Love and joy in every deed (B) ✓ rhymes with B

Score = 4/4 = 1.0 → PASS
```

**Issues**:
- If score < 0.75: `"Weak rhyme tightness: {score:.2f} (target 0.75) - rhyme scheme not followed"`

### Step 5: Evaluate Section Completeness

**Definition**: All required sections from blueprint are present in lyrics.

**Algorithm**:
1. Get required_sections from blueprint (e.g., ["Verse", "Chorus", "Bridge"])
2. Extract section markers from lyrics (e.g., `[Verse]`, `[Chorus]`)
3. Check if all required sections present
4. Score = (present_sections / required_sections)

**Target**: 1.0

**Example**:
```
Required: ["Verse", "Chorus", "Bridge"]
Present: ["Intro", "Verse", "Chorus", "Verse", "Chorus"]
Missing: ["Bridge"]

Score = 2/3 = 0.67 → FAIL
```

**Issues**:
- If score < 1.0: `"Missing required sections: {missing_sections}"`

### Step 6: Evaluate Profanity Score

**Definition**: Compliance with profanity policy based on explicit flag.

**Algorithm**:
1. Get banned_terms from blueprint.rules
2. Check constraints.explicit from SDS
3. Scan lyrics for banned terms (case-insensitive)
4. If explicit=false and banned terms found: score = 0.0
5. If explicit=true: score = 0.9 (allowed but noted)
6. If clean: score = 1.0

**Target**: 1.0 for clean, 0.9 for explicit allowed

**Example**:
```
Explicit: false
Banned terms: ["damn", "hell"]
Lyrics: "What the hell is going on?"

Score = 0.0 → FAIL
Issue: "Profanity detected (explicit=false): hell"
```

**Issues**:
- If explicit=false and banned terms found: `"Profanity detected (explicit=false): {terms}"`

### Step 7: Compute Weighted Total Score

**Algorithm**:
1. Multiply each score by its weight from rubric
2. Sum weighted scores
3. Total = (hook_density * w1) + (singability * w2) + (rhyme * w3) + (section * w4) + (profanity * w5)

**Example**:
```python
weights = {
  "hook_density": 0.25,
  "singability": 0.25,
  "rhyme_tightness": 0.20,
  "section_completeness": 0.20,
  "profanity_score": 0.10
}

scores = {
  "hook_density": 0.65,
  "singability": 0.90,
  "rhyme_tightness": 0.80,
  "section_completeness": 0.67,
  "profanity_score": 1.0
}

total = (0.65 * 0.25) + (0.90 * 0.25) + (0.80 * 0.20) + (0.67 * 0.20) + (1.0 * 0.10)
      = 0.1625 + 0.225 + 0.16 + 0.134 + 0.10
      = 0.7815 → FAIL (< 0.85)
```

### Step 8: Determine Pass/Fail

```python
if total_score >= min_total:
    pass_validation = True
    issues = []  # No critical issues
else:
    pass_validation = False
    # Collect all failing criteria
```

### Step 9: Build Issues List

Collect all failing criteria with specific thresholds:

```python
issues = []
if hook_density < 0.7:
    issues.append(f"Low hook density: {hook_density:.2f} (target 0.7)")
if singability < 0.8:
    issues.append(f"Weak singability: {singability:.2f} (target 0.8)")
if rhyme_tightness < 0.75:
    issues.append(f"Weak rhyme tightness: {rhyme_tightness:.2f} (target 0.75)")
if section_completeness < 1.0:
    issues.append(f"Missing required sections: {missing_sections}")
if profanity_score < 1.0:
    issues.append(f"Profanity detected: {banned_terms_found}")
```

### Step 10: Return Validation Results

```json
{
  "scores": {
    "total": 0.7815,
    "hook_density": 0.65,
    "singability": 0.90,
    "rhyme_tightness": 0.80,
    "section_completeness": 0.67,
    "profanity_score": 1.0
  },
  "issues": [
    "Low hook density: 0.65 (target 0.7)",
    "Missing required sections: ['Bridge']"
  ],
  "pass": false,
  "_hash": "abc123..."
}
```

## Examples

### Example 1: Passing Validation

**Input**:
```json
{
  "lyrics": "[Verse]\nFamily time is what we need\n...\n[Chorus]\nFamily time is what we need\nLove and joy in every deed\n...\n[Bridge]\nTogether we can share the light\n...",
  "style": {...},
  "producer_notes": {...},
  "blueprint": {
    "rules": {"required_sections": ["Verse", "Chorus", "Bridge"]},
    "eval_rubric": {
      "weights": {"hook_density": 0.25, "singability": 0.25, "rhyme_tightness": 0.20, "section_completeness": 0.20, "profanity_score": 0.10},
      "thresholds": {"min_total": 0.85}
    }
  }
}
```

**Output**:
```json
{
  "scores": {
    "total": 0.92,
    "hook_density": 0.85,
    "singability": 0.95,
    "rhyme_tightness": 0.90,
    "section_completeness": 1.0,
    "profanity_score": 1.0
  },
  "issues": [],
  "pass": true
}
```

### Example 2: Failing Validation (Low Hook Density)

**Input**:
```json
{
  "lyrics": "[Verse]\nWalking through the snowy night\n...\n[Chorus]\nChristmas time is here again\n...",
  "blueprint": {...}
}
```

**Output**:
```json
{
  "scores": {
    "total": 0.78,
    "hook_density": 0.45,
    "singability": 0.88,
    "rhyme_tightness": 0.82,
    "section_completeness": 1.0,
    "profanity_score": 1.0
  },
  "issues": [
    "Low hook density: 0.45 (target 0.7)"
  ],
  "pass": false
}
```

## Common Pitfalls

1. **Weights Not Summing to 1.0**: Validate weights before computing total score
2. **Missing Sections Not Detected**: Ensure section parsing handles variations (`[Chorus]`, `[CHORUS]`, `[Chorus 1]`)
3. **False Positives on Profanity**: Use case-insensitive matching and word boundaries
4. **Syllable Counting Errors**: Use robust library (pyphen) instead of naive vowel counting
5. **Hook Identification**: Don't just count word repetition; identify memorable phrases (≥ 3 words)
6. **Rhyme Detection**: Use phonetic similarity, not just suffix matching
7. **Determinism**: Ensure scoring algorithm is deterministic (no random sampling)
8. **Empty Sections**: Handle cases where section has no lyrics

## Related Skills

- **COMPOSE**: Produces artifacts validated by this skill
- **FIX**: Consumes issues list to apply targeted improvements
- **Blueprint Loading**: Requires blueprint with eval_rubric configuration

## References

- PRD: `docs/project_plans/PRDs/blueprint.prd.md` (rubric specification)
- PRD: `docs/project_plans/PRDs/claude_code_orchestration.prd.md` (section 3.6)
- Blueprint Examples: `docs/hit_song_blueprint/AI/*.md`
