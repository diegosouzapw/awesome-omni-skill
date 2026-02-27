---
name: code-analysis
description: Analyze code quality, detect code smells, identify bugs, and provide improvement recommendations. Use when reviewing code, checking quality, analyzing complexity, or when user mentions code review, refactoring suggestions, or quality assessment.
---

# Code Analysis

A comprehensive code analysis skill that helps evaluate code quality, identify potential issues, and provide actionable recommendations for improvement.

## Quick Start

Basic code analysis workflow:

```python
# Read the code file
with open("target_file.py", "r") as f:
    code = f.read()

# Run an LLM code review following code-review-instructions/copilot-instructions.md
# Summarize findings by severity with clear, actionable recommendations
```

## LLM Code Review Mode (Preferred)

Use the LLM (no scripts) to perform reviews guided by [`code-review-instructions/copilot-instructions.md`](code-review-instructions/copilot-instructions.md):
- Keep the instructions open; follow the severity model (Blocking, Important, Suggestions) and priority order (Security → Reliability → Performance/Cost → Testing → Observability → Readability).
- Make the review scope explicit (files/diff) and skip excluded assets like generated/vendor/lock files unless instructed otherwise.
- Pull the relevant stack rules and checklists referenced in the instructions; cite them when flagging issues.
- Report findings first with file:line, impact, and the specific fix or test to add before closing with positives.

## Review Checklists and Stack Rules

- **Start with general review lists**: `docs/review/code-conventions.md`, `docs/review/readability-checklist.md`, `docs/review/reliability-checklist.md`, `docs/review/security-checklist.md`, `docs/review/performance-checklist.md`, `docs/review/testing-checklist.md`.
- **Apply stack-specific rules**: `docs/stack-rules/react-typescript-rules.md`, `docs/stack-rules/angular-rules.md`, `docs/stack-rules/python-rules.md`, `docs/stack-rules/java-rules.md`, `docs/stack-rules/java-kotlin-rules.md`, `docs/stack-rules/scala-rules.md`, `docs/stack-rules/go-rules.md`.
- **Report with references**: Link findings to the checklist/rule anchor that was violated for fast remediation.

## Core Capabilities

### 1. Code Quality Assessment

Evaluate overall code quality across multiple dimensions:

- **Readability**: Variable naming, function clarity, code organization
- **Maintainability**: Code complexity, coupling, cohesion
- **Performance**: Potential bottlenecks, inefficient patterns
- **Security**: Common vulnerabilities, input validation
- **Best Practices**: Language-specific conventions and idioms

### 2. Code Smell Detection

Identify common code smells:

- **Long Method**: Functions exceeding reasonable length
- **Large Class**: Classes with too many responsibilities
- **Duplicate Code**: Similar code patterns across files
- **Dead Code**: Unused variables, functions, imports
- **Magic Numbers**: Hardcoded values without explanation
- **Deep Nesting**: Excessive indentation levels
- **God Object**: Classes doing too much

### 3. Complexity Metrics

Calculate and interpret:

- **Cyclomatic Complexity**: Number of independent paths
- **Cognitive Complexity**: How difficult code is to understand
- **Lines of Code**: Total, comment, and blank lines
- **Function/Method Count**: Per class or module
- **Dependency Analysis**: Import structure and coupling

### 4. Language-Specific Analysis

#### React + TypeScript
- Reference: `docs/stack-rules/react-typescript-rules.md`
- Component patterns (functional vs class)
- Hooks usage and custom hooks
- Props validation and typing
- State management patterns
- Performance optimization (memo, useMemo, useCallback)
- Key prop usage in lists
- Event handling best practices
- Accessibility (a11y) compliance

#### Angular + TypeScript
- Reference: `docs/stack-rules/angular-rules.md`
- Component lifecycle hooks
- RxJS observable patterns
- Dependency injection
- Change detection strategies
- Template syntax and bindings
- Service architecture
- Module organization
- TypeScript strict mode compliance

#### Python
- Reference: `docs/stack-rules/python-rules.md`
- PEP 8 compliance
- Type hints usage (Python 3.8+)
- Async/await patterns
- Exception handling patterns
- List/dict comprehensions
- Generator vs list usage
- FastAPI/Django best practices
- Pydantic model validation

#### Java + Spring Boot
- Reference: `docs/stack-rules/java-rules.md`
- SOLID principles adherence
- Design pattern opportunities
- Spring annotations usage
- JPA/Hibernate patterns
- Exception handling (@ControllerAdvice)
- Bean validation
- Lombok usage
- Transaction management
- REST API design

#### Kotlin
- Reference: `docs/stack-rules/java-kotlin-rules.md`
- Idiomatic Kotlin patterns
- Coroutines and Flow
- Null safety
- Data classes and sealed classes
- Extension functions
- Scope functions (let, run, with, apply, also)
- Companion objects
- Destructuring declarations

#### Scala
- Reference: `docs/stack-rules/scala-rules.md`
- Option/Either/Try usage instead of null
- Immutability and collection best practices
- Pattern matching exhaustiveness and for-comprehensions
- Concurrency with Future/IO and resource safety
- Controlled use of implicits/givens and type classes

#### Go
- Reference: `docs/stack-rules/go-rules.md`
- Explicit error handling and wrapping with context
- Context propagation and cancellation
- Concurrency correctness with goroutines/channels
- Interface-oriented design and zero-value-safe structs
- Database/HTTP patterns with proper resource management

## Workflows

### Workflow 0: LLM Code Review (Copilot Instructions)

1. **Open the instructions**: Keep [`code-review-instructions/copilot-instructions.md`](code-review-instructions/copilot-instructions.md) in view.
2. **Identify scope**: List files/diffs included; skip excluded files noted in the instructions.
3. **Assess critical paths first**: Security → reliability → performance/cost → tests → observability → readability/UX.
4. **Apply stack rules**: Pull the relevant concise or full stack rule docs referenced by the instructions.
5. **Report findings**: Start with blocking/important issues, cite file:line, describe impact, and propose the fix.
6. **Close with next steps**: Tests to add/run and any follow-ups required to de-risk.

### Workflow 1: Comprehensive File Analysis

1. **Read the target file**
2. **Parse code structure**: Functions, classes, imports
3. **Calculate metrics**: Complexity, LOC, depth
4. **Detect issues**: Code smells, anti-patterns
5. **Generate report**: Structured findings with priorities
6. **Provide recommendations**: Specific, actionable improvements

### Workflow 2: Multi-File Project Analysis

1. **Scan project structure**
2. **Identify dependencies**
3. **Analyze each file**: Apply single-file workflow
4. **Cross-reference issues**: Duplicate code, circular dependencies
5. **Aggregate findings**: Project-wide statistics
6. **Prioritize improvements**: Impact vs effort matrix

### Workflow 3: Focused Issue Investigation

1. **Receive specific concern**: Performance, security, readability
2. **Deep dive analysis**: Targeted inspection
3. **Identify root causes**
4. **Suggest solutions**: Multiple options with trade-offs
5. **Provide examples**: Before/after code snippets

## Analysis Output Format

Structure your analysis as follows:

```markdown
## Code Analysis Report

### Summary
- File: [filename]
- Language: [language]
- Lines of Code: [total]
- Overall Quality: [High/Medium/Low]

### Metrics
- Cyclomatic Complexity: [average] (functions: [list high-complexity ones])
- Maintainability Index: [score]
- Code Smells Detected: [count]

### Critical Issues (Priority: High)
1. [Issue description]
   - Location: Line [X]
   - Impact: [explanation]
   - Recommendation: [specific fix]

### Moderate Issues (Priority: Medium)
[Similar structure]

### Minor Improvements (Priority: Low)
[Similar structure]

### Positive Aspects
- [What's done well]
- [Good patterns observed]

### Actionable Recommendations
1. [Priority 1 action]
2. [Priority 2 action]
3. [Priority 3 action]
```

## Best Practices for Analysis

1. **Be Specific**: Point to exact line numbers and code sections
2. **Provide Context**: Explain why something is an issue
3. **Offer Solutions**: Don't just identify problems, suggest fixes
4. **Show Examples**: Include code snippets when helpful
5. **Consider Trade-offs**: Acknowledge when recommendations have costs
6. **Prioritize**: Not all issues are equal importance
7. **Be Constructive**: Focus on improvement, not criticism

## Common Patterns to Check

### Anti-Patterns

```python
# Anti-pattern: God Object
class UserManager:
    def authenticate(self): pass
    def send_email(self): pass
    def generate_report(self): pass
    def process_payment(self): pass
    # Too many responsibilities!

# Better: Single Responsibility
class Authenticator:
    def authenticate(self): pass

class EmailService:
    def send_email(self): pass
```

### Performance Issues

```python
# Inefficient: Multiple iterations
data = [process(x) for x in items]
filtered = [x for x in data if x > 0]
result = [transform(x) for x in filtered]

# Efficient: Single pass
result = [transform(process(x)) for x in items if process(x) > 0]
```

### Security Concerns

```python
# Vulnerable: SQL Injection
query = f"SELECT * FROM users WHERE id = {user_id}"

# Safe: Parameterized query
query = "SELECT * FROM users WHERE id = ?"
cursor.execute(query, (user_id,))
```

## Integration with Other Tools

This skill can reference:

- **Linting tools**: pylint, eslint, golangci-lint
- **Testing**: pytest, jest, JUnit results
- **Coverage**: Code coverage reports
- **Profiling**: Performance profiler output

For detailed language-specific guidelines, see [LANGUAGE_GUIDES.md](LANGUAGE_GUIDES.md).

- Prefer LLM-driven review; do not call scripts like `scripts/analyze.py` unless explicitly requested.

## Limitations

- Cannot execute code for dynamic analysis
- Limited to static code analysis
- May miss runtime-only issues
- Context-dependent recommendations require human judgment

## When to Use This Skill

Use this skill when:
- Reviewing pull requests
- Auditing legacy code
- Planning refactoring efforts
- Conducting code quality assessments
- Training team members on best practices
- Investigating specific code issues
- Preparing for production deployment

## Examples

See [EXAMPLES.md](EXAMPLES.md) for detailed analysis examples across different languages and scenarios.

See [EXAMPLES_STACK.md](EXAMPLES_STACK.md) for comprehensive examples with React, TypeScript, Angular, Python, Java, and Kotlin.
