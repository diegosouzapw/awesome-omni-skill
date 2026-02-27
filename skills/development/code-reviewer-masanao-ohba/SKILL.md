---
name: code-reviewer
description: Code review methodology, severity classification, and quality assessment patterns
hooks:
  SessionStart:
    - type: command
      command: |
        if command -v yq &> /dev/null && [ -f ".claude/config.yaml" ]; then
          echo "=== Project Constraints ==="
          yq -o=json '.constraints' .claude/config.yaml 2>/dev/null || true
        fi
---

# Code Reviewer

A technology-agnostic skill for systematic code review and quality assessment.

## Configuration

Review constraints are loaded from `.claude/config.yaml` → `.constraints` section.
Project-specific rules should be defined as natural language constraints.

## Review Methodology

### Systematic Review Process

```yaml
process: structured_review
description: "Multi-pass review for comprehensive coverage"

passes:
  1_correctness:
    focus: "Does the code do what it's supposed to?"
    checks:
      - Logic errors
      - Edge cases
      - Error handling
      - Business rule compliance

  2_security:
    focus: "Are there security vulnerabilities?"
    checks:
      - Input validation
      - Authentication/authorization
      - Data exposure
      - Injection vulnerabilities

  3_maintainability:
    focus: "Is the code maintainable?"
    checks:
      - Naming clarity
      - Code organization
      - Documentation
      - Complexity (cyclomatic)

  4_performance:
    focus: "Are there performance concerns?"
    checks:
      - Algorithm efficiency
      - Resource usage
      - Query optimization
      - Caching opportunities
```

## Severity Classification

### Issue Severity Levels

```yaml
severity_levels:
  critical:
    description: "Must fix before merge"
    examples:
      - Security vulnerabilities
      - Data loss risk
      - Production-breaking bugs
    action: "BLOCK merge"

  major:
    description: "Should fix before merge"
    examples:
      - Logic errors
      - Missing error handling
      - Performance issues
    action: "Request changes"

  minor:
    description: "Should fix, but can merge"
    examples:
      - Code style violations
      - Minor inefficiencies
      - Documentation gaps
    action: "Suggest improvement"

  info:
    description: "Suggestions for improvement"
    examples:
      - Alternative approaches
      - Refactoring opportunities
      - Best practices
    action: "Comment only"
```

## Review Checklists

### Security Review Checklist

```yaml
security_checklist:
  input_validation:
    - "All user inputs validated?"
    - "SQL injection prevention?"
    - "XSS prevention?"
    - "Path traversal prevention?"

  authentication:
    - "Auth checks on all protected routes?"
    - "Session management secure?"
    - "Password handling correct?"

  data_protection:
    - "Sensitive data encrypted?"
    - "PII handling compliant?"
    - "Logs sanitized?"

  authorization:
    - "Role-based access enforced?"
    - "Resource ownership verified?"
    - "Privilege escalation prevented?"
```

### Code Quality Checklist

```yaml
quality_checklist:
  readability:
    - "Variable names descriptive?"
    - "Functions single-purpose?"
    - "Comments explain why, not what?"
    - "Magic numbers replaced with constants?"

  structure:
    - "DRY principle followed?"
    - "Appropriate abstraction level?"
    - "Dependencies minimized?"
    - "Cyclomatic complexity acceptable?"

  error_handling:
    - "All error cases handled?"
    - "Errors logged appropriately?"
    - "User-facing errors clear?"
    - "No silent failures?"

  testing:
    - "New code has tests?"
    - "Edge cases covered?"
    - "Tests are meaningful (not just coverage)?"
```

## Review Feedback Patterns

### Constructive Feedback Template

```yaml
feedback_structure:
  what: "Describe the issue clearly"
  why: "Explain why it's a problem"
  how: "Suggest a solution or alternative"
  severity: "Classify the severity"

example:
  what: "This function catches all exceptions silently"
  why: "Silent failures hide bugs and make debugging difficult"
  how: "Log the exception and re-throw or handle specifically"
  severity: "major"
```

### Feedback Tone Guidelines

```yaml
tone_guidelines:
  do:
    - Ask questions rather than make accusations
    - Focus on the code, not the person
    - Acknowledge good patterns when seen
    - Provide actionable suggestions

  avoid:
    - "Why would you do this?"
    - "This is wrong"
    - "Obviously you should..."
    - Sarcasm or condescension
```

## Review Metrics

### Quality Score Calculation

```yaml
quality_scoring:
  categories:
    correctness: 30
    security: 25
    maintainability: 25
    performance: 20

  calculation:
    base_score: 100
    deductions:
      critical_issue: -25
      major_issue: -10
      minor_issue: -3
      info_issue: 0

  thresholds:
    excellent: 90-100
    good: 75-89
    acceptable: 60-74
    needs_work: 40-59
    poor: 0-39
```

## Integration

### Used By Agents

```yaml
primary_users:
  - quality-reviewer: "Core review methodology"

secondary_users:
  - code-developer: "Self-review before submission"
  - deliverable-evaluator: "Quality assessment criteria"
```
