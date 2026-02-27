---
name: compliance-checker
description: Policy-based compliance assessment for OpenClaw skills. Define security policies, assess skills against them, track violations, and generate compliance reports. Maps findings to frameworks like CIS Controls and OWASP. Integrates with arc-skill-scanner and arc-trust-verifier.
user-invocable: true
metadata: {"openclaw": {"emoji": "🛡️", "os": ["darwin", "linux"], "requires": {"bins": ["python3"]}}}
---

# Compliance Checker

Assess OpenClaw skills against defined security policies. Track compliance posture across your skill inventory with framework-mapped findings and remediation tracking.

## Why This Exists

Security scanners find vulnerabilities. Trust verifiers check provenance. But neither answers: "Does this skill meet our security policy?" Compliance Checker bridges the gap — define what "compliant" means for your environment, then assess every skill against those rules.

## Quick Start

### Define a policy
```bash
python3 {baseDir}/scripts/checker.py policy create --name "production" --description "Production deployment requirements"
```

### Add rules to the policy
```bash
python3 {baseDir}/scripts/checker.py policy add-rule --policy "production" \
  --rule "no-critical-findings" \
  --description "No CRITICAL findings from skill scanner" \
  --severity critical

python3 {baseDir}/scripts/checker.py policy add-rule --policy "production" \
  --rule "trust-verified" \
  --description "Must have VERIFIED or TRUSTED trust level" \
  --severity high

python3 {baseDir}/scripts/checker.py policy add-rule --policy "production" \
  --rule "no-network-calls" \
  --description "No unauthorized network calls in scripts" \
  --severity high

python3 {baseDir}/scripts/checker.py policy add-rule --policy "production" \
  --rule "no-shell-exec" \
  --description "No shell=True or subprocess calls" \
  --severity medium

python3 {baseDir}/scripts/checker.py policy add-rule --policy "production" \
  --rule "has-checksum" \
  --description "Must have SHA-256 checksums for all scripts" \
  --severity medium
```

### Assess a skill against a policy
```bash
python3 {baseDir}/scripts/checker.py assess --skill "arc-budget-tracker" --policy "production"
```

### Assess all installed skills
```bash
python3 {baseDir}/scripts/checker.py assess-all --policy "production"
```

### View compliance status
```bash
python3 {baseDir}/scripts/checker.py status --policy "production"
```

### Generate compliance report
```bash
python3 {baseDir}/scripts/checker.py report --policy "production" --format json
python3 {baseDir}/scripts/checker.py report --policy "production" --format text
```

## Built-in Rules

The following rules are available out of the box:

| Rule | What it checks | Framework mapping |
|------|---------------|-------------------|
| `no-critical-findings` | No CRITICAL findings from scanner | CIS Control 16, OWASP A06 |
| `no-high-findings` | No HIGH findings from scanner | CIS Control 16, OWASP A06 |
| `trust-verified` | Trust level is VERIFIED or TRUSTED | CIS Control 2 |
| `no-network-calls` | No unauthorized network requests | CIS Control 9, OWASP A10 |
| `no-shell-exec` | No shell execution patterns | CIS Control 2, OWASP A03 |
| `no-eval-exec` | No eval/exec patterns | OWASP A03 |
| `has-checksum` | SHA-256 checksums for all files | CIS Control 2 |
| `no-env-access` | No environment variable access | CIS Control 3 |
| `no-data-exfil` | No data exfiltration patterns | CIS Control 3, CIS Control 13 |
| `version-pinned` | All dependencies version-pinned | CIS Control 2 |

## Compliance Status

Each skill-policy assessment produces one of:

- **COMPLIANT** — Passes all rules in the policy
- **NON-COMPLIANT** — Fails one or more rules
- **EXEMPTED** — Has approved exemptions for all failures
- **UNKNOWN** — Not yet assessed

## Exemptions

Sometimes a skill legitimately needs to violate a rule (e.g., a network monitoring skill needs network access). Record exemptions with justification:

```bash
python3 {baseDir}/scripts/checker.py exempt --skill "arc-skill-scanner" \
  --rule "no-network-calls" \
  --reason "Scanner needs network access to check URLs against blocklists" \
  --approved-by "arc"
```

## Remediation Tracking

When a skill fails compliance, track the fix:

```bash
python3 {baseDir}/scripts/checker.py remediate --skill "some-skill" \
  --rule "no-shell-exec" \
  --action "Replaced subprocess.call with safer alternative" \
  --status fixed
```

## Storage

Compliance data is stored in `~/.openclaw/compliance/`:
- `policies/` — Policy definitions (JSON)
- `assessments/` — Assessment results per skill (JSON)
- `exemptions/` — Approved exemptions (JSON)
- `remediations/` — Remediation tracking (JSON)

## Integration

Compliance Checker reads output from:
- **arc-skill-scanner** — vulnerability findings
- **arc-trust-verifier** — trust levels and attestations

Run a full pipeline:
```bash
# Scan → verify trust → assess compliance
python3 {baseDir}/scripts/checker.py pipeline --skill "some-skill" --policy "production"
```
