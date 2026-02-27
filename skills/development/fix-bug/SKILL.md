---
name: "fix-bug"
description: "Test-first bug fixing: Write failing test that reproduces bug → Fix code → Test passes. Systematic, verifiable approach prevents regression and wastes no time on trial-and-error deployments."
version: "1.0.0"
type: "problem-solving"
category: "debugging"
---

# Fix Bug: Test-First Bug Fixing

Resolve software defects using a disciplined test-first approach. Write a failing test that reproduces the bug BEFORE modifying any production code.

## Instructions

**Phase 1: Write a Failing Test (DO THIS FIRST)**

1. **Choose test level:**
   - Unit test (isolated component/function)
   - Integration test (component interaction)
   - Acceptance test (end-to-end behavior)

2. **Write test that reproduces the bug:**

   ```python
   # Python example
   def test_bug_description():
       """Bug: [Describe what's wrong]"""
       # Arrange: Set up scenario that triggers bug
       input_data = create_bug_triggering_input()

       # Act: Execute the failing code
       result = system_under_test.process(input_data)

       # Assert: What SHOULD happen (test will FAIL)
       assert result == expected_correct_behavior
   ```

   ```java
   // Java example
   @Test
   @DisplayName("Bug: [Describe what's wrong]")
   void reproduceBug() {
       // Arrange: Set up scenario that triggers bug
       var input = createBugTriggeringInput();

       // Act: Execute the failing code
       var result = systemUnderTest.process(input);

       // Assert: What SHOULD happen (test will FAIL)
       assertThat(result).isEqualTo(expectedCorrectBehavior);
   }
   ```

3. **Run the test - MUST FAIL:**
   - If test passes, it doesn't reproduce the bug
   - Adjust test until it fails

4. **Verify failure reason:**
   - Change assertion to expect WRONG behavior
   - Test should now PASS
   - Revert assertion back to correct behavior
   - This proves test reproduces the actual bug

5. **Commit the failing test:**
   ```bash
   git add tests/...
   git commit -m "test: reproduce bug - [description]

   Failing test demonstrates [bug behavior].
   Expected: [correct behavior]
   Actual: [buggy behavior]

   Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
   ```

**Phase 2: Fix the Code**

1. **Minimal change:**
   - Use test as guide for debugging
   - Make smallest change to pass test
   - Avoid "while I'm here" refactoring

2. **Run the test - MUST PASS:**
   - Test transitions from RED → GREEN
   - This proves bug is fixed

3. **Run ALL tests:**
   - Ensure no regressions
   - All existing tests must still pass

4. **Refactor if needed:**
   - Improve code quality
   - Keep tests passing

**Phase 3: Commit the Fix**

```bash
git add src/...
git commit -m "fix: [description of bug fix]

Resolves issue where [bug behavior] occurred.
Root cause: [explanation]
Fix: [what was changed]

Test: tests/.../test_bug_reproduction.py

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

## Golden Rules

❌ **DO NOT:**
- Make changes to production code before writing test
- Run full application to debug
- Create side-scripts to investigate
- Deploy and test manually
- Guess and check
- Skip writing test because "I know what's wrong"

✅ **DO:**
- Write failing test FIRST (no exceptions)
- Verify test fails for the RIGHT reason
- Make minimal change to pass test
- Run all tests before committing
- Commit test and fix together
- Document root cause in commit message

## Why This Works

**Traditional Approach (Anti-Pattern):**
```
1. Guess where problem is
2. Change production code
3. Deploy and test manually
4. Still broken? Go to step 1
5. Eventually works (you think)
6. Hope you didn't break anything
```

**Problems:**
- ❌ No proof bug is fixed
- ❌ No proof nothing else broke
- ❌ Wastes time on deployments
- ❌ Bug can return unnoticed

**Test-First Approach:**
```
1. Write failing test (reproduces bug)
2. Fix production code
3. Test passes
4. Run all tests
5. Done - proven fixed
```

**Benefits:**
- ✅ Proof bug existed
- ✅ Proof bug is fixed
- ✅ Proof nothing else broke
- ✅ Fast feedback (no deployment)
- ✅ Regression prevention
- ✅ Documents the issue

## Example Workflow

```
User: Bug report - API returns 500 when input contains special characters

Agent (Backend-Dev):
  Phase 1: Reproduce bug with test

  Created: tests/test_api_special_chars.py

  def test_api_handles_special_characters():
      \"\"\"Bug: API returns 500 for input with special chars\"\"\"
      client = TestClient(app)
      response = client.post("/api/process", json={
          "text": "Hello <world> & friends!"
      })

      # Should return 200, not 500
      assert response.status_code == 200
      assert response.json()["status"] == "success"

  Running test... ❌ FAILED
  AssertionError: assert 500 == 200

  Good! Test reproduces the bug.

  Committed: git commit -m "test: reproduce API 500 with special chars"

  Phase 2: Fix the code

  Root cause: Input sanitization missing for HTML entities

  Fixed: src/api/processor.py
  - Added html.escape() before processing input

  Running test... ✅ PASSED
  Running all tests... ✅ 127/127 PASSED

  Phase 3: Commit fix

  git commit -m "fix: sanitize HTML entities in API input

  Resolves 500 error when input contains <, >, &, etc.
  Root cause: Missing input sanitization
  Fix: Added html.escape() in processor

  Test: tests/test_api_special_chars.py

  Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"

  ✅ Bug fixed with proof (test passes)
  ✅ Regression prevented (test stays in suite)
```

## Test Level Selection Guide

**Unit Test** - Choose when:
- Bug in single function/method
- Isolated component behavior wrong
- Logic error in calculation
- Example: "Function returns wrong value for edge case"

**Integration Test** - Choose when:
- Bug in component interaction
- Data flow between modules broken
- API contract violated
- Example: "Service A sends wrong format to Service B"

**Acceptance Test** - Choose when:
- End-to-end user flow broken
- Full feature behaves incorrectly
- System-level behavior wrong
- Example: "User cannot complete checkout process"

**Start with smallest test that reproduces bug** (usually unit test).

## Common Pitfalls

**Pitfall 1: "I'll write test after fixing"**
- ❌ Never works - you forget or run out of time
- ✅ Test FIRST ensures it gets written

**Pitfall 2: "Test passes but bug still exists"**
- ❌ Test doesn't reproduce actual bug
- ✅ Verify test fails for RIGHT reason (change assertion to wrong value, should pass)

**Pitfall 3: "Too hard to write test"**
- ❌ Usually means code is hard to test (design smell)
- ✅ Use this as opportunity to improve testability

**Pitfall 4: "I fixed it manually, works now"**
- ❌ No proof, bug can return
- ✅ Capture fix in test for regression prevention

## Related Skills

- `/iterate` - Use after fixing bug to continue with next batch
- `/review` - Request architect review for complex bug fixes

## References

- **Directive 028:** `.github/agents/directives/028_bugfixing_techniques.md`
- **Approach:** `.github/agents/approaches/test-first-bug-fixing.md`
- **Checklist:** `.github/agents/approaches/bug-fixing-checklist.md`
- **Directive 017:** `.github/agents/directives/017_test_driven_development.md` (TDD)
