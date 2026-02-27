---
name: code-review-agent
description: Comprehensive security and quality code review agent that checks for OWASP vulnerabilities, GDPR compliance, accessibility standards, and code quality issues.
---

# Code Review Agent

## Purpose

The **Code Review Agent** performs comprehensive automated code review analyzing implementations for:
- **Security Vulnerabilities** - OWASP Top 10 compliance
- **Code Quality** - Anti-patterns, optimization opportunities
- **GDPR Compliance** - Data privacy, consent management, user rights
- **Accessibility** - WCAG 2.1 AA standards compliance

## When to Use This Skill

Invoke the code review agent:

1. **After Development Stage** - Review Developer A and Developer B implementations
2. **Before Arbitration** - Ensure both solutions meet quality/security standards
3. **Before Production Deployment** - Final security and compliance check
4. **On-Demand Reviews** - Security audit of existing codebase

## Responsibilities

### 1. Security Analysis (OWASP Top 10)

Detects all OWASP Top 10 (2021) vulnerabilities:

- **A01 - Broken Access Control** - Authorization bypasses, IDOR vulnerabilities
- **A02 - Cryptographic Failures** - Weak encryption, hardcoded secrets
- **A03 - Injection** - SQL, command, XSS, template injection
- **A04 - Insecure Design** - Missing security controls, threat modeling gaps
- **A05 - Security Misconfiguration** - Default credentials, unnecessary features
- **A06 - Vulnerable Components** - Outdated dependencies, known CVEs
- **A07 - Authentication Failures** - Weak passwords, session management
- **A08 - Integrity Failures** - Unsigned updates, insecure deserialization
- **A09 - Logging Failures** - Missing audit logs, insufficient monitoring
- **A10 - SSRF** - Unvalidated URL requests

**Example Detection:**

```python
# CRITICAL - SQL Injection detected
# File: database.py:45
cursor.execute(f"SELECT * FROM users WHERE id={user_id}")

# Recommendation: Use parameterized queries
cursor.execute("SELECT * FROM users WHERE id=?", (user_id,))
```

### 2. Code Quality Review

Identifies anti-patterns and optimization issues:

**Anti-Patterns:**
- God Objects (too many responsibilities)
- Spaghetti Code (tangled control flow)
- Magic Numbers/Strings
- Duplicate Code (DRY violations)
- Long Methods (>50 lines)
- Deep Nesting (>3 levels)
- Tight Coupling

**Optimization Issues:**
- Inefficient algorithms (O(n²) vs O(n))
- N+1 database query problems
- Missing caching
- Memory leaks (unclosed resources)
- Blocking I/O in async contexts

**Example Detection:**

```python
# MEDIUM - God Object detected
# File: user_manager.py:1
class UserManager:  # 850 lines, 45 methods
    # Handles: auth, validation, email, logging, billing

# Recommendation: Split into UserService, AuthService,
# EmailService, BillingService per SRP
```

### 3. GDPR Compliance

Ensures data privacy and regulatory compliance:

**Required Implementations:**
- ✅ Data minimization (Article 5)
- ✅ Consent management (Article 6, 7)
- ✅ Right to access (Article 15)
- ✅ Right to erasure (Article 17)
- ✅ Data portability (Article 20)
- ✅ Privacy by design (Article 25)
- ✅ Breach notification (Article 33, 34)
- ✅ Data Processing Agreements (Article 28)

**Example Detection:**

```python
# HIGH - GDPR Article 17 violation
# File: user_service.py:120
def delete_user(user_id):
    # TODO: implement

# Recommendation: Implement complete data deletion across
# all tables, logs, and backups. Confirm deletion to user.
```

### 4. Accessibility (WCAG 2.1 AA)

Validates compliance with WCAG 2.1 Level AA:

**Perceivable:**
- Text alternatives (alt attributes)
- Captions for media
- Semantic HTML structure
- Color contrast ≥4.5:1
- Resizable text (200%)

**Operable:**
- Keyboard accessible
- No keyboard traps
- Adjustable timing
- No flashing content
- Skip navigation links

**Understandable:**
- Language specified (lang attribute)
- Predictable navigation
- Input error identification
- Form labels

**Robust:**
- Valid HTML
- ARIA roles/properties
- Status messages

**Example Detection:**

```html
<!-- MEDIUM - WCAG 1.1.1 violation -->
<!-- File: dashboard.html:34 -->
<img src="chart.png">

<!-- Recommendation: Add descriptive alt text -->
<img src="chart.png" alt="Monthly revenue chart showing 15% growth">
```

## Review Process

### 1. Input
- Developer implementation directory (`/tmp/developer-a/` or `/tmp/developer-b/`)
- Task context (title, description)
- ADR for architectural decisions

### 2. Analysis
Uses LLM APIs (OpenAI/Anthropic) to:
1. Parse all implementation files (.py, .js, .html, .css, etc.)
2. Analyze against OWASP, GDPR, WCAG standards
3. Detect code quality issues and anti-patterns
4. Generate categorized findings with severity levels

### 3. Output

**JSON Report:**
```json
{
  "review_summary": {
    "overall_status": "PASS|NEEDS_IMPROVEMENT|FAIL",
    "total_issues": 15,
    "critical_issues": 0,
    "high_issues": 3,
    "medium_issues": 8,
    "low_issues": 4,
    "score": {
      "code_quality": 85,
      "security": 75,
      "gdpr_compliance": 90,
      "accessibility": 80,
      "overall": 82
    }
  },
  "issues": [
    {
      "category": "SECURITY",
      "subcategory": "A03:2021 - SQL Injection",
      "severity": "HIGH",
      "file": "database.py",
      "line": 45,
      "description": "...",
      "recommendation": "...",
      "owasp_reference": "..."
    }
  ]
}
```

**Markdown Summary:**
- Overall assessment
- Category scores
- Critical/High issues detailed
- Positive findings
- Actionable recommendations

## Severity Levels

| Severity | Criteria | Examples |
|----------|----------|----------|
| **CRITICAL** | Security breach risk, GDPR fine risk (€20M), accessibility blocker | SQL injection, exposed secrets, missing data deletion |
| **HIGH** | Significant vulnerability, major compliance gap | Weak encryption, missing consent, inaccessible forms |
| **MEDIUM** | Code quality issue, minor security concern | God objects, missing CSRF tokens, low contrast |
| **LOW** | Style/convention, optimization opportunity | Magic numbers, inefficient algorithm |

## Decision Criteria

**PASS** (Implementation acceptable):
- 0 critical issues
- ≤2 high issues
- Overall score ≥80

**NEEDS_IMPROVEMENT** (Can proceed with warnings):
- 0 critical issues
- ≤5 high issues
- Overall score ≥60

**FAIL** (Must fix before proceeding):
- Any critical issues
- >5 high issues
- Overall score <60

## Integration with Pipeline

### Placement in Pipeline

```
Development Stage (Developer A + B)
          ↓
   📋 Code Review Agent  ← NEW STAGE
          ↓
   Validation (TDD checks)
          ↓
   Arbitration (Select winner)
          ↓
   Integration
```

### Communication

**Receives:**
- Implementation files from developers
- Task context from orchestrator
- ADR from architecture agent

**Sends:**
- Review report to orchestrator
- Issues list to validation agent
- Pass/Fail status to arbitration agent

## Usage Examples

### Standalone Usage

```bash
python3 code_review_agent.py \
  --developer developer-a \
  --implementation-dir /tmp/developer-a/ \
  --output-dir /tmp/code-reviews/ \
  --task-title "User Authentication" \
  --task-description "Implement JWT-based auth"
```

### Programmatic Usage

```python
from code_review_agent import CodeReviewAgent

agent = CodeReviewAgent(
    developer_name="developer-a",
    llm_provider="openai"
)

result = agent.review_implementation(
    implementation_dir="/tmp/developer-a/",
    task_title="User Authentication",
    task_description="Implement JWT auth with bcrypt",
    output_dir="/tmp/code-reviews/"
)

print(f"Status: {result['review_status']}")
print(f"Score: {result['overall_score']}/100")
print(f"Critical Issues: {result['critical_issues']}")
```

### Pipeline Integration

```python
# In pipeline orchestrator
from code_review_agent import CodeReviewAgent

# Review Developer A
review_agent_a = CodeReviewAgent(developer_name="developer-a")
review_a = review_agent_a.review_implementation(
    implementation_dir="/tmp/developer-a/",
    task_title=task_title,
    task_description=task_description,
    output_dir="/tmp/code-reviews/"
)

# Review Developer B
review_agent_b = CodeReviewAgent(developer_name="developer-b")
review_b = review_agent_b.review_implementation(
    implementation_dir="/tmp/developer-b/",
    task_title=task_title,
    task_description=task_description,
    output_dir="/tmp/code-reviews/"
)

# Use reviews in arbitration
if review_a['critical_issues'] > 0:
    # Disqualify Developer A
    winner = "developer-b"
```

## Configuration

### Environment Variables

```bash
# LLM Provider (default: openai)
ARTEMIS_LLM_PROVIDER=openai

# API Keys
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...

# Optional: Specific model
ARTEMIS_LLM_MODEL=gpt-4o
```

### Supported Models

**OpenAI:**
- gpt-4o (default)
- gpt-4o-mini
- gpt-4-turbo

**Anthropic:**
- claude-sonnet-4-5-20250929 (default)
- claude-3-5-sonnet-20241022

## Cost Considerations

Typical review costs per implementation:
- **Prompt tokens**: 3,000-5,000 (code + prompt)
- **Completion tokens**: 2,000-4,000 (review JSON)
- **Total**: 5,000-9,000 tokens

### Estimated Costs

| Model | Cost per Review | Recommended Use |
|-------|----------------|----------------|
| GPT-4o | $0.05-$0.12 | Production reviews |
| GPT-4o-mini | $0.005-$0.01 | Development/testing |
| Claude Sonnet 4.5 | $0.10-$0.20 | Thorough security audits |

## Best Practices

1. **Review Early** - Catch issues before arbitration
2. **Review Both Developers** - Ensures fair comparison
3. **Monitor Critical Issues** - Auto-reject implementations with critical issues
4. **Track Metrics** - Monitor security score trends over time
5. **Use in CI/CD** - Automated reviews on every commit
6. **Combine with Static Analysis** - Complement with Bandit, ESLint, SonarQube

## Limitations

- **Static Analysis Only** - Cannot detect runtime vulnerabilities
- **No Execution** - Cannot find logic errors requiring execution
- **Language Coverage** - Best for Python, JavaScript, HTML/CSS
- **LLM Dependent** - Quality depends on LLM capabilities
- **False Positives** - May flag intentional design decisions

## Future Enhancements

1. **Custom Rule Sets** - Industry-specific compliance (HIPAA, PCI-DSS)
2. **Severity Tuning** - Configurable severity thresholds
3. **Auto-Fix Suggestions** - Generate code patches
4. **Diff-Based Review** - Review only changed files
5. **Integration Tests** - Security-focused integration testing
6. **Vulnerability Database** - Check against CVE databases

## References

- [OWASP Top 10 (2021)](https://owasp.org/Top10/)
- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [GDPR Official Text](https://gdpr-info.eu/)
- [CWE Top 25](https://cwe.mitre.org/top25/)

---

**Version:** 1.0.0

**Maintained By:** Artemis Pipeline Team

**Last Updated:** October 22, 2025
