---
name: error-root-analyzer
description: Comprehensive error analysis and root cause resolution. Use when programs fail, crash, or produce errors during execution. This skill performs deep debugging by identifying root causes (not just surface-level symptoms), conducting thorough module reviews to uncover related bugs and exceptions, and implementing holistic fixes that address all discovered issues.
---

# Error Root Cause Analyzer

This skill provides a systematic approach to identify and resolve programming errors by analyzing root causes and conducting comprehensive module reviews.

## Core Principles

1. **Root Cause Analysis**: Never stop at surface-level error messages. Trace back to the fundamental cause.
2. **Holistic Review**: When fixing a module, review its entire scope for related issues.
3. **Comprehensive Fixes**: Address all discovered problems, not just the immediate error.
4. **Prevention**: Identify and fix potential errors before they manifest.

## Systematic Error Resolution Workflow

### Phase 1: Error Capture and Initial Analysis

1. **Capture Complete Error Information**
   - Full stack trace with line numbers
   - Error type and message
   - Input/state that triggered the error
   - Environment context (Python version, OS, dependencies)

2. **Identify Error Surface**
   - Immediate error location (file, function, line)
   - Error category (syntax, runtime, logic, dependency, environment)
   - Affected modules and components

### Phase 2: Root Cause Investigation

**Critical**: Do not proceed to fixes until root cause is identified.

1. **Trace Error Backwards**
   - Start from error line
   - Trace execution flow backwards
   - Identify the point where incorrect state/data originated
   - Check all assumptions and preconditions

2. **Analyze Error Patterns** (See `references/error-patterns.md`)
   - Import errors → dependency/path issues
   - AttributeError → object lifecycle/initialization issues
   - TypeError → data contract violations
   - IndexError/KeyError → boundary/validation issues
   - Connection errors → environment/configuration issues

3. **Check Common Root Causes**
   - Missing or incorrect initialization
   - Invalid assumptions about data/state
   - Race conditions or timing issues
   - Configuration mismatches
   - Dependency version conflicts
   - Resource availability (memory, disk, network)
   - Permission issues

### Phase 3: Comprehensive Module Review

When an error is found in a module, conduct a thorough review:

1. **Static Code Analysis**
   - Review all functions/classes in the affected module
   - Check for similar patterns that might fail
   - Verify error handling completeness
   - Validate input sanitization and boundary checks
   - Check resource cleanup (file handles, connections, locks)

2. **Dependency Analysis**
   - Review all imports and their usage
   - Check for version compatibility
   - Verify API contract assumptions
   - Identify fragile dependencies

3. **Logic Flow Analysis**
   - Trace critical paths through the module
   - Identify edge cases not handled
   - Check state management
   - Verify transaction boundaries
   - Review concurrency/threading issues

4. **Exception Landscape**
   - Map all possible exceptions
   - Verify exception handling coverage
   - Check exception propagation correctness
   - Ensure graceful degradation paths exist

5. **Use Debugging Checklist** (See `references/debugging-checklist.md`)
   - Language-specific checks
   - Framework-specific validations
   - Common anti-patterns

### Phase 4: Fix Design and Implementation

1. **Design Comprehensive Fix**
   - Address root cause directly
   - Fix all related issues found in module review
   - Add missing error handling
   - Implement defensive programming where needed
   - Add validation/sanitization as appropriate

2. **Prioritize Fixes**
   - **Critical**: Root cause and blocking issues
   - **High**: Related bugs that will fail soon
   - **Medium**: Missing error handling, edge cases
   - **Low**: Code quality improvements

3. **Implement Fixes Systematically**
   - Fix root cause first
   - Apply related fixes in logical order
   - Add tests for each fix if possible
   - Document complex changes

4. **Add Prevention Mechanisms**
   - Input validation
   - Precondition checks
   - Better error messages
   - Logging for debugging
   - Assertions for invariants

### Phase 5: Verification and Testing

1. **Verify Root Cause Fixed**
   - Test with original failure case
   - Verify error no longer occurs
   - Check fix doesn't introduce new issues

2. **Test Related Fixes**
   - Test each identified issue
   - Verify edge cases handled
   - Check error messages are helpful

3. **Regression Testing**
   - Test existing functionality still works
   - Verify no new errors introduced
   - Test with various inputs/scenarios

4. **Integration Testing**
   - Test module in context of larger system
   - Verify interactions with other modules
   - Check end-to-end workflows

## Error Analysis Strategies

### For Import/Dependency Errors

1. Check module installation and versions
2. Verify import paths and PYTHONPATH
3. Check for circular dependencies
4. Verify virtual environment activation
5. Review `requirements.txt` or `pyproject.toml`
6. Check for naming conflicts

### For Runtime Errors

1. Trace data flow to error point
2. Check variable initialization
3. Verify object lifecycle
4. Review state mutations
5. Check method call order
6. Validate data types and contracts

### For Logic Errors

1. Review algorithm correctness
2. Check boundary conditions
3. Verify loop invariants
4. Validate state machines
5. Review calculation logic
6. Check for off-by-one errors

### For Performance/Timeout Errors

1. Profile code execution
2. Identify bottlenecks
3. Check for infinite loops
4. Review algorithm complexity
5. Check database query efficiency
6. Review memory usage patterns

## Documentation and Communication

When reporting findings and fixes:

1. **Error Summary**
   - What failed and how it manifested
   - Root cause identified
   - Why it occurred

2. **Comprehensive Fix List**
   - Primary fix for root cause
   - All related issues addressed
   - Prevention mechanisms added

3. **Risk Assessment**
   - Potential impact of changes
   - Areas requiring extra testing
   - Known limitations

4. **Next Steps**
   - Recommended testing approach
   - Monitoring suggestions
   - Future improvements

## Integration with Development Workflow

- Use this skill when any error occurs during development or testing
- Apply before deploying fixes to production
- Use for post-mortem analysis of production issues
- Apply during code review when bugs are found

## Reference Materials

- **Error Patterns**: See `references/error-patterns.md` for common error patterns and their root causes
- **Debugging Checklist**: See `references/debugging-checklist.md` for language and framework-specific checks
