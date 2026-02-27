---
name: sre-task-refinement
description: Use when you have to refine subtasks into actionable plans ensuring that all corner cases are handled and we understand all the requirements.
---

<codex_compat>
This skill was ported from Claude Code. In Codex:
- "Skill tool" means read the skill's `SKILL.md` from disk.
- "TodoWrite" means create and maintain a checklist section in your response.
- "Task()" means `spawn_agent` (dispatch in parallel via `multi_tool_use.parallel` when needed).
- Claude-specific hooks and slash commands are not available; skip those steps.
</codex_compat>


<skill_overview>
Review bd task plans with Google Fellow SRE perspective to ensure junior engineer can execute without questions; catch edge cases, verify granularity, strengthen criteria, prevent production issues before implementation.
</skill_overview>

<rigidity_level>
LOW FREEDOM - Follow the 8-category checklist exactly. Apply all categories to every task. No skipping red flag checks. Always verify no placeholder text after updates. Reject plans with critical gaps.
</rigidity_level>

<quick_reference>
| Category | Key Questions | Auto-Reject If |
|----------|---------------|----------------|
| 1. Granularity | Tasks 4-8 hours? Phases <16 hours? | Any task >16h without breakdown |
| 2. Implementability | Junior can execute without questions? | Vague language, missing details |
| 3. Success Criteria | 3+ measurable criteria per task? | Can't verify ("works well") |
| 4. Dependencies | Correct parent-child, blocking relationships? | Circular dependencies |
| 5. Safety Standards | Anti-patterns specified? Error handling? | No anti-patterns section |
| 6. Edge Cases | Empty input? Unicode? Concurrency? Failures? | No edge case consideration |
| 7. Red Flags | Placeholder text? Vague instructions? | "[detailed above]", "TODO" |
| 8. Test Meaningfulness | Tests catch real bugs? Not tautological? | Tests only verify syntax/existence |

**Perspective**: Google Fellow SRE with 20+ years experience reviewing junior engineer designs.

**Time**: Don't rush - catching one gap pre-implementation saves hours of rework.
</quick_reference>

<when_to_use>
Use when:
- Reviewing bd epic/feature plans before implementation
- Need to ensure junior engineer can execute without questions
- Want to catch edge cases and failure modes upfront
- Need to verify task granularity (4-8 hour subtasks)
- After hyperpowers:writing-plans creates initial plan
- Before hyperpowers:executing-plans starts implementation

Don't use when:
- Task already being implemented (too late)
- Just need to understand existing code (use codebase-investigator)
- Debugging issues (use debugging-with-tools)
- Want to create plan from scratch (use brainstorming → writing-plans)
</when_to_use>

<the_process>
## Announcement

**Announce:** "I'm using hyperpowers:sre-task-refinement to review this plan with Google Fellow-level scrutiny."

---

## Review Checklist (Apply to Every Task)

### 1. Task Granularity

**Check:**
- [ ] No task >8 hours (subtasks) or >16 hours (phases)?
- [ ] Large phases broken into 4-8 hour subtasks?
- [ ] Each subtask independently completable?
- [ ] Each subtask has clear deliverable?

**If task >16 hours:**
- Create subtasks with `bd create`
- Link with `bd dep add child parent --type parent-child`
- Update parent to coordinator role

---

### 2. Implementability (Junior Engineer Test)

**Check:**
- [ ] Can junior engineer implement without asking questions?
- [ ] Function signatures/behaviors described, not just "implement X"?
- [ ] Test scenarios described (what they verify, not just names)?
- [ ] "Done" clearly defined with verifiable criteria?
- [ ] All file paths specified or marked "TBD: new file"?

**Red flags:**
- "Implement properly" (how?)
- "Add support" (for what exactly?)
- "Make it work" (what does working mean?)
- File paths missing or ambiguous

---

### 3. Success Criteria Quality

**Check:**
- [ ] Each task has 3+ specific, measurable success criteria?
- [ ] All criteria testable/verifiable (not subjective)?
- [ ] Includes automated verification (tests pass, clippy clean)?
- [ ] No vague criteria like "works well" or "is implemented"?

**Good criteria examples:**
- ✅ "5+ unit tests pass (valid VIN, invalid checksum, various formats)"
- ✅ "Clippy clean with no warnings"
- ✅ "Performance: <100ms for 1000 records"

**Bad criteria examples:**
- ❌ "Code is good quality"
- ❌ "Works correctly"
- ❌ "Is implemented"

---

### 4. Dependency Structure

**Check:**
- [ ] Parent-child relationships correct (epic → phases → subtasks)?
- [ ] Blocking dependencies correct (earlier work blocks later)?
- [ ] No circular dependencies?
- [ ] Dependency graph makes logical sense?

**Verify with:**
```bash
bd dep tree bd-1  # Show full dependency tree
```

---

### 5. Safety & Quality Standards

**Check:**
- [ ] Anti-patterns include unwrap/expect prohibition?
- [ ] Anti-patterns include TODO prohibition (or must have issue #)?
- [ ] Anti-patterns include stub implementation prohibition?
- [ ] Error handling requirements specified (use Result, avoid panic)?
- [ ] Test requirements specific (test names, scenarios listed)?

**Minimum anti-patterns:**
- ❌ No unwrap/expect in production code
- ❌ No TODOs without issue numbers
- ❌ No stub implementations (unimplemented!, todo!)
- ❌ No regex without catastrophic backtracking check

---

### 6. Edge Cases & Failure Modes (Fellow SRE Perspective)

**Ask for each task:**
- [ ] What happens with malformed input?
- [ ] What happens with empty/nil/zero values?
- [ ] What happens under high load/concurrency?
- [ ] What happens when dependencies fail?
- [ ] What happens with Unicode, special characters, large inputs?
- [ ] Are these edge cases addressed in the plan?

**Add to Key Considerations section:**
- Edge case descriptions
- Mitigation strategies
- References to similar code handling these cases

---

### 7. Red Flags (AUTO-REJECT)

**Check for these - if found, REJECT plan:**
- ❌ Any task >16 hours without subtask breakdown
- ❌ Vague language: "implement properly", "add support", "make it work"
- ❌ Success criteria that can't be verified: "code is good", "works well"
- ❌ Missing test specifications
- ❌ "We'll handle this later" or "TODO" in the plan itself
- ❌ No anti-patterns section
- ❌ Implementation checklist with fewer than 3 items per task
- ❌ No effort estimates
- ❌ Missing error handling considerations
- ❌ **CRITICAL: Placeholder text in design field** - "[detailed above]", "[as specified]", "[complete steps here]"

---

### 8. Test Meaningfulness (Fellow SRE Perspective)

**Tests must catch real bugs, not inflate coverage.** For every test specification:

**Ask these questions:**
- [ ] What specific bug would this test catch?
- [ ] Could production code break while this test still passes?
- [ ] Does this test exercise a real user scenario or failure mode?
- [ ] Is the assertion meaningful? (`result == expected` vs `result != nil`)

**Red flags (AUTO-REJECT):**
- ❌ Tests that only verify syntax/existence ("enum has cases", "struct has fields")
- ❌ Tautological tests (pass by definition: `expect(builder.build() != nil)` when build() can't return nil)
- ❌ Tests that duplicate implementation (testing 1+1==2 by checking 1+1==2)
- ❌ Tests without meaningful assertions (call code but don't verify outcomes)
- ❌ Tests that verify mocks instead of production code
- ❌ Round-trip tests that only use happy path (Codable without edge cases)
- ❌ Tests named generically ("test_basic", "test_it_works")

**Good test specifications:**
- ✅ "test_empty_payload_returns_validation_error" - catches missing validation
- ✅ "test_concurrent_writes_dont_corrupt_data" - catches race condition
- ✅ "test_malformed_json_returns_400_not_500" - catches error handling bug
- ✅ "test_unicode_name_preserved_after_roundtrip" - catches encoding bugs

**Bad test specifications (reject or strengthen):**
- ❌ "test_user_model_exists" - tautological, compiler catches this
- ❌ "test_builder_returns_value" - tautological if return type non-optional
- ❌ "test_basic_functionality" - vague, what specific bug does it catch?
- ❌ "test_encode_decode" - only happy path, no edge cases specified

**When reviewing test specifications:**
```markdown
For each test in success criteria, verify:

Test: "test_vin_validation"
- What bug does it catch? ⚠️ Unclear - need specific scenarios
- Could code break while test passes? ⚠️ Unknown without specifics

STRENGTHEN TO:
- test_valid_vin_checksum_accepted
- test_invalid_vin_checksum_rejected (catches missing checksum validation)
- test_lowercase_vin_normalized (catches case handling bug)
- test_vin_with_invalid_chars_rejected (catches input validation bug)
```

---

## Review Process

For each task in the plan:

**Step 1: Read the task**
```bash
bd show bd-3
```

**Step 2: Apply all 8 checklist categories**
- Task Granularity
- Implementability
- Success Criteria Quality
- Dependency Structure
- Safety & Quality Standards
- Edge Cases & Failure Modes
- Red Flags
- Test Meaningfulness

**Step 3: Document findings**
Take notes:
- What's done well
- What's missing
- What's vague or ambiguous
- Hidden failure modes not addressed
- Better approaches or simplifications

**Step 4: Update the task**

Use `bd update` to add missing information:

```bash
bd update bd-3 --design "$(cat <<'EOF'
## Goal
[Original goal, preserved]

## Effort Estimate
[Updated estimate if needed]

## Success Criteria
- [ ] Existing criteria
- [ ] NEW: Added missing measurable criteria

## Implementation Checklist
[Complete checklist with file paths]

## Key Considerations (ADDED BY SRE REVIEW)

**Edge Case: Empty Input**
- What happens when input is empty string?
- MUST validate input length before processing

**Edge Case: Unicode Handling**
- What if string contains RTL or surrogate pairs?
- Use proper Unicode-aware string methods

**Performance Concern: Regex Backtracking**
- Pattern `.*[a-z]+.*` has catastrophic backtracking risk
- MUST test with pathological inputs (e.g., 10000 'a's)
- Use possessive quantifiers or bounded repetition

**Reference Implementation**
- Study src/similar/module.rs for pattern to follow

## Anti-patterns
[Original anti-patterns]
- ❌ NEW: Specific anti-pattern for this task's risks
EOF
)"
```

**IMPORTANT:** Use `--design` for full detailed description, NOT `--description` (title only).

**Step 5: Verify no placeholder text (MANDATORY)**

After updating, read back with `bd show bd-N` and verify:
- ✅ All sections contain actual content, not meta-references
- ✅ No placeholder text like "[detailed above]", "[as specified]", "[will be added]"
- ✅ Implementation steps fully written with actual code examples
- ✅ Success criteria explicit, not referencing "criteria above"
- ❌ If ANY placeholder text found: REJECT and rewrite with actual content

---

## Breaking Down Large Tasks

If task >16 hours, create subtasks:

```bash
# Create first subtask
bd create "Subtask 1: [Specific Component]" \
  --type task \
  --priority 1 \
  --design "[Complete subtask design with all 7 categories addressed]"
# Returns bd-10

# Create second subtask
bd create "Subtask 2: [Another Component]" \
  --type task \
  --priority 1 \
  --design "[Complete subtask design]"
# Returns bd-11

# Link subtasks to parent with parent-child relationship
bd dep add bd-10 bd-3 --type parent-child  # bd-10 is child of bd-3
bd dep add bd-11 bd-3 --type parent-child  # bd-11 is child of bd-3

# Add sequential dependencies if needed (LATER depends on EARLIER)
bd dep add bd-11 bd-10  # bd-11 depends on bd-10 (do bd-10 first)

# Update parent to coordinator
bd update bd-3 --design "$(cat <<'EOF'
## Goal
Coordinate implementation of [feature]. Broken into N subtasks.

## Success Criteria
- [ ] All N child subtasks closed
- [ ] Integration tests pass
- [ ] [High-level verification criteria]
EOF
)"
```

---

## Output Format

After reviewing all tasks:

```markdown
## Plan Review Results

### Epic: [Name] ([epic-id])

### Overall Assessment
[APPROVE ✅ / NEEDS REVISION ⚠️ / REJECT ❌]

### Dependency Structure Review
[Output of `bd dep tree [epic-id]`]

**Structure Quality**: [✅ Correct / ❌ Issues found]
- [Comments on parent-child relationships]
- [Comments on blocking dependencies]
- [Comments on granularity]

### Task-by-Task Review

#### [Task Name] (bd-N)
**Type**: [epic/feature/task]
**Status**: [✅ Ready / ⚠️ Needs Minor Improvements / ❌ Needs Major Revision]
**Estimated Effort**: [X hours] ([✅ Good / ❌ Too large - needs breakdown])

**Strengths**:
- [What's done well]

**Critical Issues** (must fix):
- [Blocking problems]

**Improvements Needed**:
- [What to add/clarify]

**Edge Cases Missing**:
- [Failure modes not addressed]

**Changes Made**:
- [Specific improvements added via `bd update`]

---

[Repeat for each task/phase/subtask]

### Summary of Changes

**Issues Updated**:
- bd-3 - Added edge case handling for Unicode, regex backtracking risks
- bd-5 - Broke into 3 subtasks (was 40 hours, now 3x8 hours)
- bd-7 - Strengthened success criteria (added test names, verification commands)

### Critical Gaps Across Plan
1. [Pattern of missing items across multiple tasks]
2. [Systemic issues in the plan]

### Recommendations

[If APPROVE]:
✅ Plan is solid and ready for implementation.
- All tasks are junior-engineer implementable
- Dependency structure is correct
- Edge cases and failure modes addressed

[If NEEDS REVISION]:
⚠️ Plan needs improvements before implementation:
- [List major items that need addressing]
- After changes, re-run hyperpowers:sre-task-refinement

[If REJECT]:
❌ Plan has fundamental issues and needs redesign:
- [Critical problems]
```
</the_process>

<examples>
<example>
<scenario>Developer reviews task but skips edge case analysis (Category 6)</scenario>

<code>
# Review of bd-3: Implement VIN scanner

## Checklist review:
1. Granularity: ✅ 6-8 hours
2. Implementability: ✅ Junior can implement
3. Success Criteria: ✅ Has 5 test scenarios
4. Dependencies: ✅ Correct
5. Safety Standards: ✅ Anti-patterns present
6. Edge Cases: [SKIPPED - "looks straightforward"]
7. Red Flags: ✅ None found

Conclusion: "Task looks good, approve ✅"

# Task ships without edge case review
# Production issues occur:
- VIN scanner matches random 17-char strings (no checksum validation)
- Lowercase VINs not handled (should normalize)
- Catastrophic regex backtracking on long inputs (DoS vulnerability)
</code>

<why_it_fails>
- Skipped Category 6 (Edge Cases) assuming task was "straightforward"
- Didn't ask: What happens with invalid checksums? Lowercase? Long inputs?
- Missed critical production issues:
  - False positives (no checksum validation)
  - Data handling bugs (case sensitivity)
  - Security vulnerability (regex DoS)
- Junior engineer didn't know to handle these (not in task)
- Production incidents occur after deployment
- Hours of emergency fixes, customer impact
- SRE review failed to prevent known failure modes
</why_it_fails>

<correction>
**Apply Category 6 rigorously:**

```markdown
## Edge Case Analysis for bd-3: VIN Scanner

Ask for EVERY task:
- Malformed input? VIN has checksum - must validate, not just pattern match
- Empty/nil? What if empty string passed?
- Concurrency? Read-only scanner, no concurrency issues
- Dependency failures? No external dependencies
- Unicode/special chars? VIN is alphanumeric only, but what about lowercase?
- Large inputs? Regex `.*` patterns can cause catastrophic backtracking

Findings:
❌ VIN checksum validation not mentioned (will match random strings)
❌ Case normalization not mentioned (lowercase VINs exist)
❌ Regex backtracking risk not mentioned (DoS vulnerability)
```

**Update task:**
```bash
bd update bd-3 --design "$(cat <<'EOF'
[... original content ...]

## Key Considerations (ADDED BY SRE REVIEW)

**VIN Checksum Complexity**:
- ISO 3779 requires transliteration table (letters → numbers)
- Weighted sum algorithm with modulo 11
- Reference: https://en.wikipedia.org/wiki/Vehicle_identification_number#Check_digit
- MUST validate checksum, not just pattern - prevents false positives

**Case Normalization**:
- VINs can appear in lowercase
- MUST normalize to uppercase before validation
- Test with mixed case: "1hgbh41jxmn109186"

**Regex Backtracking Risk**:
- CRITICAL: Pattern `.*[A-HJ-NPR-Z0-9]{17}.*` has backtracking risk
- Test with pathological input: 10000 'X's followed by 16-char string
- Use possessive quantifiers or bounded repetition
- Reference: https://www.regular-expressions.info/catastrophic.html

**Edge Cases to Test**:
- Valid VIN with valid checksum (should match)
- Valid pattern but invalid checksum (should NOT match)
- Lowercase VIN (should normalize and validate)
- Ambiguous chars I/O not valid in VIN (should reject)
- Very long input (should not DoS)
EOF
)"
```

**What you gain:**
- Prevented false positives (checksum validation)
- Prevented data handling bugs (case normalization)
- Prevented security vulnerability (regex DoS)
- Junior engineer has complete requirements
- Production issues caught pre-implementation
- Proper SRE review preventing known failure modes
- Customer trust maintained
</correction>
</example>

<example>
<scenario>Developer approves task with placeholder text (Red Flag #10)</scenario>

<code>
# Review of bd-5: Implement License Plate Scanner

bd show bd-5:

## Implementation Checklist
- [ ] Create scanner module
- [ ] [Complete implementation steps detailed above]
- [ ] Add tests

## Success Criteria
- [ ] [As specified in the implementation checklist]
- [ ] Tests pass

## Key Considerations
- [Will be added during implementation]

# Developer's review:
"Looks comprehensive, has implementation checklist and success criteria ✅"

# During implementation:
Junior engineer: "What are the 'implementation steps detailed above'?"
Junior engineer: "What specific success criteria should I verify?"
Junior engineer: "What key considerations exist?"

# No answers in the task - junior engineer blocked
# Have to research and add missing information
# Implementation delayed by 2 days
</code>

<why_it_fails>
- Missed Red Flag #10: Placeholder text present
- "[Complete implementation steps detailed above]" is meta-reference, not content
- "[As specified in the implementation checklist]" is circular reference
- "[Will be added during implementation]" is deferral, not specification
- Junior engineer can't execute - missing critical information
- Task looks complete but actually incomplete
- Implementation blocked until details added
- SRE review failed to catch placeholder text
</why_it_fails>

<correction>
**Check for placeholder text after reading:**

```markdown
## Red Flag Check (Category 7)

Read through bd-5 line by line:

Line 15: "[Complete implementation steps detailed above]"
❌ PLACEHOLDER - "detailed above" is meta-reference, not actual content

Line 22: "[As specified in the implementation checklist]"
❌ PLACEHOLDER - Circular reference to another section, not explicit criteria

Line 30: "[Will be added during implementation]"
❌ PLACEHOLDER - Deferral to future, not actual considerations

DECISION: REJECT ❌
Reason: Contains placeholder text - task not ready for implementation
```

**Update task with actual content:**
```bash
bd update bd-5 --design "$(cat <<'EOF'
## Implementation Checklist
- [ ] Create src/scan/plugins/scanners/license_plate.rs
- [ ] Implement LicensePlateScanner struct with ScanPlugin trait
- [ ] Add regex patterns for US states:
  - CA: `[0-9][A-Z]{3}[0-9]{3}` (e.g., 1ABC123)
  - NY: `[A-Z]{3}[0-9]{4}` (e.g., ABC1234)
  - TX: `[A-Z]{3}[0-9]{4}|[0-9]{3}[A-Z]{3}` (e.g., ABC1234 or 123ABC)
  - Generic: `[A-Z0-9]{5,8}` (fallback)
- [ ] Implement has_healthcare_context() check
- [ ] Create test module with 8+ test cases
- [ ] Register in src/scan/plugins/scanners/mod.rs

## Success Criteria
- [ ] Valid CA plate "1ABC123" detected in healthcare context
- [ ] Valid NY plate "ABC1234" detected in healthcare context
- [ ] Invalid plate "123" NOT detected (too short)
- [ ] Valid plate NOT detected outside healthcare context
- [ ] 8+ unit tests pass covering all patterns and edge cases
- [ ] Clippy clean, no warnings
- [ ] cargo test passes

## Key Considerations

**False Positive Risk**:
- License plates are short and generic (5-8 chars)
- MUST require healthcare context via has_healthcare_context()
- Without context, will match random alphanumeric sequences
- Test: Random string "ABC1234" should NOT match outside healthcare context

**State Format Variations**:
- 50 US states have different formats
- Implement common formats (CA, NY, TX) + generic fallback
- Document which formats supported in module docstring
- Consider international plates in future iteration

**Performance**:
- Regex patterns are simple, no backtracking risk
- Should process <1ms per chunk

**Reference Implementation**:
- Study src/scan/plugins/scanners/vehicle_identifier.rs
- Follow same pattern: regex + context check + tests
EOF
)"
```

**Verify no placeholder text:**
```bash
bd show bd-5
# Read entire output
# Confirm: All sections have actual content
# Confirm: No "[detailed above]", "[as specified]", "[will be added]"
# ✅ Task ready for implementation
```

**What you gain:**
- Junior engineer has complete specification
- No blocked implementation waiting for details
- All edge cases documented upfront
- Success criteria explicit and verifiable
- Key considerations prevent common mistakes
- No placeholder text - task truly ready
- Professional SRE review standard maintained
</correction>
</example>

<example>
<scenario>Developer accepts vague success criteria (Category 3)</scenario>

<code>
# Review of bd-7: Implement Data Encryption

bd show bd-7:

## Success Criteria
- [ ] Encryption is implemented correctly
- [ ] Code is good quality
- [ ] Tests work properly

# Developer's review:
"Has 3 success criteria ✅ Meets minimum requirement"

# During implementation:
Junior engineer: "How do I know if encryption is 'correct'?"
Junior engineer: "What makes code 'good quality'?"
Junior engineer: "What does 'tests work properly' mean?"

# Junior engineer makes best guesses:
- Uses ECB mode (insecure, should use GCM)
- No key rotation (bad practice)
- Tests only happy path (misses edge cases)

# Code review finds critical security issues
# Complete rewrite required - 3 days wasted
</code>

<why_it_fails>
- Category 3 check: "All criteria testable/verifiable?" ❌ FAILED
- "Implemented correctly" - not measurable (correct by what standard?)
- "Good quality" - subjective, not verifiable
- "Work properly" - vague, what is proper?
- Junior engineer can't verify criteria objectively
- Makes incorrect assumptions filling gaps
- Security vulnerabilities introduced
- Wastes time on implementation that fails review
- SRE review failed to strengthen criteria
</why_it_fails>

<correction>
**Apply Category 3 rigorously:**

```markdown
## Success Criteria Analysis for bd-7

Current criteria:
- [ ] Encryption is implemented correctly
  ❌ NOT TESTABLE - "correctly" is subjective, no standard specified

- [ ] Code is good quality
  ❌ NOT TESTABLE - "good quality" is opinion, not measurable

- [ ] Tests work properly
  ❌ NOT TESTABLE - "properly" is vague, no definition

Minimum requirement: 3+ specific, measurable, testable criteria
Current: 0 testable criteria
DECISION: REJECT ❌
```

**Update with measurable criteria:**
```bash
bd update bd-7 --design "$(cat <<'EOF'
[... original content ...]

## Success Criteria

**Encryption Implementation**:
- [ ] Uses AES-256-GCM mode (verified in code review)
- [ ] Key derivation via PBKDF2 with 100,000 iterations (NIST recommendation)
- [ ] Unique IV generated per encryption (crypto_random)
- [ ] Authentication tag verified on decryption

**Code Quality** (automated checks):
- [ ] Clippy clean with no warnings: `cargo clippy -- -D warnings`
- [ ] Rustfmt compliant: `cargo fmt --check`
- [ ] No unwrap/expect in production: `rg "\.unwrap\(\)|\.expect\(" src/` returns 0
- [ ] No TODOs without issue numbers: `rg "TODO" src/` returns 0

**Test Coverage**:
- [ ] 12+ unit tests pass covering:
  - test_encrypt_decrypt_roundtrip (happy path)
  - test_wrong_key_fails_auth (security)
  - test_modified_ciphertext_fails_auth (security)
  - test_empty_plaintext (edge case)
  - test_large_plaintext_10mb (performance)
  - test_unicode_plaintext (data handling)
  - test_concurrent_encryption (thread safety)
  - test_iv_uniqueness (security)
  - [4 more specific scenarios]
- [ ] All tests pass: `cargo test encryption`
- [ ] Test coverage >90%: `cargo tarpaulin --packages encryption`

**Documentation**:
- [ ] Module docstring explains encryption scheme (AES-256-GCM)
- [ ] Function docstrings include examples
- [ ] Security considerations documented (key management, IV handling)

**Security Review**:
- [ ] No hardcoded keys or IVs (verified via grep)
- [ ] Key zeroized after use (verified in code)
- [ ] Constant-time comparison for auth tag (timing attack prevention)
EOF
)"
```

**What you gain:**
- Every criterion objectively verifiable
- Junior engineer knows exactly what "done" means
- Automated checks (clippy, fmt, grep) provide instant feedback
- Specific test scenarios prevent missed edge cases
- Security requirements explicit (GCM, PBKDF2, unique IV)
- No ambiguity - can verify each criterion with command or code review
- Professional SRE review standard: measurable, testable, specific
</correction>
</example>
</examples>

<critical_rules>
## Rules That Have No Exceptions

1. **Apply all 8 categories to every task** → No skipping any category for any task
2. **Reject plans with placeholder text** → "[detailed above]", "[as specified]" = instant reject
3. **Verify no placeholder after updates** → Read back with `bd show` and confirm actual content
4. **Break tasks >16 hours** → Create subtasks, don't accept large tasks
5. **Strengthen vague criteria** → "Works correctly" → measurable verification commands
6. **Add edge cases to every task** → Empty? Unicode? Concurrency? Failures?
7. **Never skip Category 6** → Edge case analysis prevents production issues
8. **Reject tautological tests** → Tests must catch bugs, not verify compiler-checked facts

## Common Excuses

All of these mean: **STOP. Apply the full process.**

- "Task looks straightforward" (Edge cases hide in "straightforward" tasks)
- "Has 3 criteria, meets minimum" (Criteria must be measurable, not just 3+ items)
- "Placeholder text is just formatting" (Placeholders mean incomplete specification)
- "Can handle edge cases during implementation" (Must specify upfront, not defer)
- "Junior will figure it out" (Junior should NOT need to figure out - we specify)
- "Too detailed, feels like micromanaging" (Detail prevents questions and rework)
- "Taking too long to review" (One gap caught saves hours of rework)
- "Any tests are better than none" (Tautological tests are worse - give false confidence)
- "Tests are specified, don't need to review them" (Test quality matters more than quantity)
- "Coverage metrics will catch missing tests" (Coverage gaming = meaningless tests)
</critical_rules>

<verification_checklist>
Before completing SRE review:

**Per task reviewed:**
- [ ] Applied all 8 categories (Granularity, Implementability, Criteria, Dependencies, Safety, Edge Cases, Red Flags, Test Meaningfulness)
- [ ] Checked for placeholder text in design field
- [ ] Updated task with missing information via `bd update --design`
- [ ] Verified updated task with `bd show` (no placeholders remain)
- [ ] Broke down any task >16 hours into subtasks
- [ ] Strengthened vague success criteria to measurable
- [ ] Added edge case analysis to Key Considerations
- [ ] Strengthened anti-patterns based on failure modes
- [ ] Verified test specifications catch real bugs (not tautological)

**Overall plan:**
- [ ] Reviewed ALL tasks/phases/subtasks (no exceptions)
- [ ] Verified dependency structure with `bd dep tree`
- [ ] Documented findings for each task
- [ ] Created summary of changes made
- [ ] Provided clear recommendation (APPROVE/NEEDS REVISION/REJECT)

**Can't check all boxes?** Return to review process and complete missing steps.
</verification_checklist>

<integration>
**This skill is used after:**
- hyperpowers:writing-plans (creates initial plan)
- hyperpowers:brainstorming (establishes requirements)

**This skill is used before:**
- hyperpowers:executing-plans (implements tasks)

**This skill is also called by:**
- hyperpowers:executing-plans (REQUIRED for new tasks created during execution)

**Call chains:**
```
Initial planning:
hyperpowers:brainstorming → hyperpowers:writing-plans → hyperpowers:sre-task-refinement → hyperpowers:executing-plans
                                                    ↓
                                            (if gaps: revise and re-review)

During execution (for new tasks):
hyperpowers:executing-plans → creates new task → hyperpowers:sre-task-refinement → STOP checkpoint
```

**This skill uses:**
- bd commands (show, update, create, dep add, dep tree)
- Google Fellow SRE perspective (20+ years distributed systems)
- 8-category checklist (mandatory for every task)

**Time expectations:**
- Small epic (3-5 tasks): 15-20 minutes
- Medium epic (6-10 tasks): 25-40 minutes
- Large epic (10+ tasks): 45-60 minutes

**Don't rush:** Catching one critical gap pre-implementation saves hours of rework.
</integration>

<resources>
**Review patterns:**
- Task too large (>16h) → Break into 4-8h subtasks
- Vague criteria ("works correctly") → Measurable commands/checks
- Missing edge cases → Add to Key Considerations with mitigations
- Placeholder text → Rewrite with actual content
- Tautological tests → Strengthen to catch specific bugs

**Test meaningfulness questions:**
- "What bug would this catch?" → If you can't name one, test is pointless
- "Could code break while test passes?" → If yes, test is too weak
- "Is this testing the mock or production code?" → Mock-testing is useless
- "Is the assertion meaningful?" → `!= nil` is weaker than `== expectedValue`

**When stuck:**
- Unsure if task too large → Ask: Can junior complete in one day?
- Unsure if criteria measurable → Ask: Can I verify with command/code review?
- Unsure if edge case matters → Ask: Could this fail in production?
- Unsure if placeholder → Ask: Does this reference other content instead of providing content?
- Unsure if test meaningful → Ask: What specific production bug does this prevent?

**Key principle:** Junior engineer should be able to execute task without asking questions. If they would need to ask, specification is incomplete. Tests must catch bugs, not inflate metrics.
</resources>