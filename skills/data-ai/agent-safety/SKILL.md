---
name: agent-safety
description: Ensure agent safety - guardrails, content filtering, monitoring, and compliance
sasmp_version: "1.3.0"
bonded_agent: 07-agent-safety
bond_type: PRIMARY_BOND
version: "2.0.0"
---

# Agent Safety

Implement safety systems for responsible AI agent deployment.

## When to Use This Skill

Invoke this skill when:
- Adding input/output guardrails
- Implementing content filtering
- Setting up rate limiting
- Ensuring compliance (GDPR, SOC2)

## Parameter Schema

| Parameter | Type | Required | Description | Default |
|-----------|------|----------|-------------|---------|
| `task` | string | Yes | Safety goal | - |
| `risk_level` | enum | No | `strict`, `moderate`, `permissive` | `strict` |
| `filters` | list | No | Filter types to enable | `["injection", "pii", "toxicity"]` |

## Quick Start

```python
from guardrails import Guard
from guardrails.validators import ToxicLanguage, PIIFilter

guard = Guard.from_validators([
    ToxicLanguage(threshold=0.8, on_fail="exception"),
    PIIFilter(on_fail="fix")
])

# Validate output
validated = guard.validate(llm_response)
```

## Guardrail Types

### Input Guardrails
```python
# Prompt injection detection
INJECTION_PATTERNS = [
    r"ignore (previous|all) instructions",
    r"you are now",
    r"forget everything"
]
```

### Output Guardrails
```python
# Content filtering
filters = [
    ToxicityFilter(),
    PIIRedactor(),
    HallucinationDetector()
]
```

## Rate Limiting

```python
class RateLimiter:
    def __init__(self, rpm=60, tpm=100000):
        self.rpm = rpm
        self.tpm = tpm

    def check(self, user_id, tokens):
        # Token bucket algorithm
        pass
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| False positives | Tune thresholds |
| Injection bypass | Add LLM-based detection |
| PII leakage | Add secondary validation |
| Performance hit | Cache filter results |

## Best Practices

- Defense in depth (multiple layers)
- Fail-safe defaults (deny by default)
- Audit everything
- Regular red team testing

## Compliance Checklist

- [ ] Input validation active
- [ ] Output filtering enabled
- [ ] Audit logging configured
- [ ] Rate limits set
- [ ] PII handling compliant

## Related Skills

- `tool-calling` - Input validation
- `llm-integration` - API security
- `multi-agent` - Per-agent permissions

## References

- [OWASP LLM Top 10](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- [Guardrails AI](https://docs.guardrailsai.com/)
