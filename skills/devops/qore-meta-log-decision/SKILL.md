---
name: qore-meta-log-decision
description: Record major engineering decisions into the QoreLogic meta-ledger with rationale, risk grading, and auditable evidence.
creator: MythologIQ Labs, LLC
license: Proprietary (FailSafe Project)
---
# Log Decision Skill
## Implement Meta-Ledger for Auditable Decision Tracking

**Skill Name:** qore-meta-log-decision
**Version:** 1.0
**Purpose:** Implement QoreLogic SOA Ledger for development decisions

---

## Usage

```
/qore-meta-log-decision <decision_type> <decision> <rationale> [risk_grade]
```

Or invoke in conversation:
> "Let's log this architecture decision to the meta-ledger..."

---

## What This Skill Does

Implements QoreLogic's **SOA Ledger** principle for meta-governance: maintaining a Merkle-chained,
cryptographically signed audit trail of all major development decisions. Enables traceability,
accountability, and rollback capability.

---

## Skill Instructions

When this skill is invoked:

### 1. Gather Decision Details

Collect comprehensive information:

**Required Fields:**
- **decision_type:** Category from taxonomy below
- **decision:** What was decided (concise)
- **rationale:** Why this decision was made
- **evidence:** Supporting data/citations
- **approver:** Who approved (Lead Reviewer, User, etc.)
- **risk_grade:** L1/L2/L3 classification

**Optional Fields:**
- **alternatives_considered:** What else was evaluated
- **trade_offs:** What we're sacrificing
- **reversibility:** Can this be undone? How easily?
- **dependencies:** What depends on this decision
- **timeline_impact:** How does this affect schedule

### 2. Classify Decision Type

Use standardized taxonomy:

| Decision Type | Description | Risk Grade | Examples |
|---------------|-------------|------------|----------|
| **ARCHITECTURE** | System design, component structure | L2/L3 | "Use SQLite not PostgreSQL" |
| **SECURITY** | Cryptography, auth, key management | L3 | "Migrate to platform keystore" |
| **SCOPE_CHANGE** | Add/remove features | L2/L3 | "Defer PyVeritas to Phase 3" |
| **TECH_STACK** | Languages, frameworks, dependencies | L2 | "Add pytest-cov for coverage" |
| **TIMELINE** | Schedule adjustments | L1/L2 | "Extend Week 2 by 3 days" |
| **QUALITY_GATE** | Standards, thresholds, criteria | L2 | "Require 80% test coverage" |
| **PROCESS** | Development workflow changes | L1/L2 | "Weekly drift audits every Friday" |
| **VENDOR** | Third-party service selection | L2/L3 | "Use GitHub Actions for CI/CD" |
| **DEPLOYMENT** | Infrastructure, hosting decisions | L3 | "Deploy to AWS Lambda" |
| **BUDGET** | Resource allocation, costs | L2 | "Allocate 2 engineers full-time" |

### 3. Validate Risk Grade

Ensure correct L1/L2/L3 classification:

**L3 CRITICAL (Requires User Approval):**
- Security/cryptographic changes
- Data schema modifications
- Authentication/authorization
- Production deployment architecture
- Budget >$10K
- Timeline slips >2 weeks

**L2 FUNCTIONAL (Requires Lead Reviewer Approval):**
- Architecture changes
- Tech stack additions
- Scope changes
- Quality gate definitions
- Timeline slips 1-2 weeks
- Budget $1K-$10K

**L1 ROUTINE (Lead Reviewer Decision):**
- Process improvements
- Documentation updates
- Minor timeline adjustments (<1 week)
- Budget <$1K

### 4. Load Current Ledger State

Read `docs/META_LEDGER.md` to get:
- Last entry ID
- Previous entry hash
- Current chain status

### 5. Create New Entry

Generate ledger entry:

```json
{
  "entry_id": 5,
  "timestamp": "2025-12-24T16:00:00Z",
  "decision_type": "ARCHITECTURE",
  "decision": "Use SQLite for Phases 1-2, defer PostgreSQL until 100+ agent scale",
  "rationale": "KISS principle - no proven bottleneck yet. Database audit shows <100 TPS sufficient for pilot. SQLite simpler, zero ops overhead, suitable for single-node deployment.",
  "evidence": [
    "Database audit: current load <10 TPS",
    "SQLite supports <100 TPS per documentation",
    "PostgreSQL adds deployment complexity without current benefit"
  ],
  "alternatives_considered": [
    "PostgreSQL: Rejected - no proven scalability need",
    "MongoDB: Rejected - relational data better fit",
    "MySQL: Rejected - no advantages over SQLite for this scale"
  ],
  "trade_offs": {
    "accepting": "Single-writer limit (~100 TPS), migration cost if scale exceeds",
    "gaining": "Simplicity, zero-ops deployment, file-based backups"
  },
  "approver": "Lead Reviewer",
  "risk_grade": "L2",
  "reversibility": "Medium - migration to PostgreSQL possible but requires schema export/import",
  "dependencies": [
    "Database hardening plan (Week 2)",
    "Backup strategy (requires file-based approach)"
  ],
  "timeline_impact": "Saves 1 week setup time vs PostgreSQL deployment",
  "related_decisions": [3, 4],
  "previous_hash": "b4a1f2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1",
  "entry_hash": "c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6"
}
```

### 6. Calculate Hash

Generate Merkle chain link:

```python
import hashlib
import json

def calculate_entry_hash(entry, previous_hash):
    # Create canonical representation
    hash_payload = {
        "entry_id": entry["entry_id"],
        "timestamp": entry["timestamp"],
        "decision_type": entry["decision_type"],
        "decision": entry["decision"],
        "rationale": entry["rationale"],
        "approver": entry["approver"],
        "risk_grade": entry["risk_grade"],
        "previous_hash": previous_hash
    }

    # Sort keys for deterministic hashing
    canonical = json.dumps(hash_payload, sort_keys=True)

    # SHA-256 hash
    return hashlib.sha256(canonical.encode()).hexdigest()
```

### 7. Append to Ledger

Update `docs/META_LEDGER.md`:

```markdown
## Entry #5: SQLite vs PostgreSQL Decision

**Type:** ARCHITECTURE (L2)
**Date:** 2025-12-24 16:00:00 UTC
**Approver:** Lead Reviewer

### Decision
Use SQLite for Phases 1-2, defer PostgreSQL migration until 100+ agent scale proven necessary.

### Rationale
KISS principle - no proven bottleneck. Database audit shows current load <10 TPS, SQLite supports <100 TPS. PostgreSQL adds deployment complexity without current benefit.

### Evidence
- Database audit: current load <10 TPS
- SQLite documentation: supports <100 TPS single-writer
- PostgreSQL deployment: requires containerization, ops overhead

### Alternatives Considered
1. **PostgreSQL** - Rejected: No proven scalability need, adds complexity
2. **MongoDB** - Rejected: Relational data better fit for ledger structure
3. **MySQL** - Rejected: No advantages over SQLite at this scale

### Trade-offs
**Accepting:**
- Single-writer limit (~100 TPS)
- Migration cost if scale exceeds assumptions

**Gaining:**
- Simplicity (zero-ops, file-based)
- Faster setup (saves 1 week)
- File-based backups (easier disaster recovery)

### Reversibility
**Medium** - Migration to PostgreSQL possible via schema export/import. Estimated 1 week effort if needed.

### Dependencies
- Database hardening plan (Week 2) - must use file-based backup strategy
- Performance testing (Week 7) - validates TPS assumptions

### Timeline Impact
Saves 1 week setup time vs PostgreSQL deployment in Week 1.

### Related Decisions
- #3: Database backup strategy
- #4: Multi-tenant isolation approach

### Hash Chain
```
Previous: b4a1f2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1
Current:  c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6
```

---
```

### 8. Validate Chain Integrity

After appending, verify:

```python
def validate_ledger_integrity(ledger_file):
    """Verify entire Merkle chain integrity"""
    entries = load_ledger_entries(ledger_file)

    previous_hash = "0" * 64  # Genesis hash

    for entry in entries:
        # Recalculate hash
        expected_hash = calculate_entry_hash(entry, previous_hash)

        if entry["entry_hash"] != expected_hash:
            return False, f"Chain broken at entry {entry['entry_id']}"

        previous_hash = entry["entry_hash"]

    return True, "Chain integrity verified"
```

### 9. Check L3 Approval Requirements

If risk_grade is L3:

```markdown
## ⚠️ L3 APPROVAL REQUIRED

This decision requires explicit user approval before implementation.

**Decision:** {decision}
**Risk Grade:** L3 CRITICAL
**Impact:** {impact description}

**Approval Status:** PENDING
**Approval Deadline:** {24 hours from now}

**To approve, user must:**
1. Review decision, rationale, and evidence
2. Confirm understanding of trade-offs
3. Provide written approval: "APPROVED: Entry #{entry_id}"
4. Sign approval (optional but recommended)

**Until approved:**
- Implementation blocked
- Related work can proceed if not dependent
- Decision remains in CONDITIONAL status
```

---

## Examples

### Example 1: Architecture Decision (L2)

**Command:**
```
/qore-meta-log-decision ARCHITECTURE "Use SQLite for Phases 1-2" "KISS - no proven bottleneck" L2
```

**Output:**
```markdown
✅ Meta-Ledger Updated: Entry #5

**Decision Type:** ARCHITECTURE (L2)
**Decision:** Use SQLite for Phases 1-2
**Approver:** Lead Reviewer
**Status:** APPROVED (L2 - Lead Reviewer authority)

**Hash Chain:** ✅ Integrity verified
**File Updated:** docs/META_LEDGER.md

**Next Steps:**
- Implementation can proceed
- No user approval required (L2)
```

### Example 2: Security Decision (L3)

**Command:**
```
/qore-meta-log-decision SECURITY "Migrate passphrase to platform keystore" "Fix CRITICAL vulnerability" L3
```

**Output:**
```markdown
⚠️ Meta-Ledger Updated: Entry #6 (PENDING APPROVAL)

**Decision Type:** SECURITY (L3 CRITICAL)
**Decision:** Migrate passphrase storage to platform keystore
**Impact:** Fixes CRITICAL security vulnerability (plaintext master passphrase)

**Status:** CONDITIONAL - Awaiting user approval
**Approval Required By:** 2025-12-25 16:00:00 UTC (24 hours)

**Rationale:**
Current plaintext storage at ~/.qorelogic/security/identity.secret creates single point of compromise. Platform keystore (Windows Credential Manager, macOS Keychain, Linux Secret Service) provides OS-level encryption and access control.

**Trade-offs:**
- **Accepting:** Platform dependency, different implementation per OS
- **Gaining:** Hardware-backed encryption, audit trail, no plaintext secrets

**Reversibility:** Low - migration one-way, manual keystore deletion needed to rollback

**User Action Required:**
Please review and respond with:
- "APPROVED: Entry #6" to proceed with implementation
- "CONDITIONAL: Entry #6 - {concerns}" to request changes
- "REJECTED: Entry #6 - {rationale}" to block implementation

**File Updated:** docs/META_LEDGER.md (entry marked CONDITIONAL)
```

### Example 3: Scope Change (L2)

**Command:**
```
/qore-meta-log-decision SCOPE_CHANGE "Defer PyVeritas to Phase 3" "Focus on validated features first" L2
```

**Output:**
```markdown
✅ Meta-Ledger Updated: Entry #7

**Decision Type:** SCOPE_CHANGE (L2)
**Decision:** Defer PyVeritas implementation to Phase 3 (Week 9+)
**Approver:** Lead Reviewer
**Status:** APPROVED

**Rationale:**
PyVeritas is aspirational (80% accuracy per research, not validated). Focus Phases 1-2 on working features (CBMC wrapper, Z3 integration) with proven ROI. Re-evaluate PyVeritas after baseline comparison in Week 5.

**Impact:**
- Phase 1-2: Tier 3 verification uses CBMC + Z3 (proven tools)
- Phase 3: Evaluate PyVeritas based on validation data
- Documentation: Update to reflect current vs. planned status

**Timeline Impact:** None - removes unproven work from critical path

**Hash Chain:** ✅ Integrity verified
**File Updated:** docs/META_LEDGER.md
```

---

## Meta-Ledger File Structure

Located at: `docs/META_LEDGER.md`

```markdown
# Q-DNA Development Meta-Ledger
## Auditable Decision Trail

**Version:** 1.0
**Created:** 2025-12-24
**Purpose:** Merkle-chained audit trail of development decisions
**Chain Status:** ✅ VALID (7 entries)

---

## Ledger Statistics

- **Total Entries:** 7
- **L3 Decisions:** 2 (1 approved, 1 pending)
- **L2 Decisions:** 4
- **L1 Decisions:** 1
- **Chain Breaks:** 0
- **Last Validation:** 2025-12-24 16:30:00 UTC

---

## Entry #0: Genesis (Ledger Initialization)

**Type:** GENESIS
**Date:** 2025-12-24 12:00:00 UTC
**Approver:** System

### Decision
Initialize Meta-Ledger for Q-DNA development governance.

### Hash Chain
```
Previous: 0000000000000000000000000000000000000000000000000000000000000000
Current:  a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2
```

---

## Entry #1: ...

[Continue with entries as shown in examples]

---

## Pending Approvals

### Entry #6: Keystore Migration (L3)
**Status:** CONDITIONAL - Awaiting user approval
**Submitted:** 2025-12-24 15:00:00 UTC
**Deadline:** 2025-12-25 15:00:00 UTC (23 hours remaining)
**Action Required:** User review and approval

---

## Chain Validation Log

| Date | Validator | Status | Notes |
|------|-----------|--------|-------|
| 2025-12-24 16:30 | System | ✅ VALID | 7 entries verified |
| 2025-12-24 15:00 | System | ✅ VALID | 6 entries verified |
```

---

## Success Criteria

This skill succeeds when:

1. ✅ **Complete Audit Trail:** All major decisions logged, zero gaps
2. ✅ **Chain Integrity:** 100% hash chain validation passes
3. ✅ **L3 Compliance:** All L3 decisions have user approval before implementation
4. ✅ **Traceability:** Can trace any implementation back to decision rationale
5. ✅ **Accountability:** Clear approver for every decision
6. ✅ **Reversibility Analysis:** Know cost/risk of undoing any decision

---

## Integration with QoreLogic

This skill implements:

- **SOA Ledger:** Merkle-chained audit trail
- **Risk Grading:** L1/L2/L3 classification with approval workflows
- **Progressive Formalization:** Decision → Evidence → Hash → Signature
- **Divergence Doctrine:** Transparent decision-making, no hidden choices

---

## When to Use

Invoke this skill for:

- ✅ Architecture decisions (component structure, database choice, etc.)
- ✅ Security changes (cryptography, auth, key management)
- ✅ Scope changes (add/remove features, timeline adjustments)
- ✅ Tech stack changes (new dependencies, frameworks, languages)
- ✅ Quality gates (coverage thresholds, performance SLAs)
- ✅ Deployment architecture
- ✅ Budget allocations
- ✅ Any decision that affects multiple developers or phases

**Don't use for:**
- ❌ Trivial choices (variable names, code formatting)
- ❌ Implementation details (which loop type to use)
- ❌ Personal preferences without technical impact

---

## Output

This skill will:

1. Create/update `docs/META_LEDGER.md`
2. Add new entry with hash chain link
3. Validate entire chain integrity
4. Flag L3 items for user approval
5. Generate approval request if needed
6. Update ledger statistics

**Validation Output:**

```markdown
## Chain Integrity Validation

**Timestamp:** 2025-12-24 16:30:00 UTC
**Entries Validated:** 7
**Result:** ✅ PASS

**Hash Verification:**
- Entry #0 (Genesis): ✅ Valid
- Entry #1: ✅ Valid (links to #0)
- Entry #2: ✅ Valid (links to #1)
- Entry #3: ✅ Valid (links to #2)
- Entry #4: ✅ Valid (links to #3)
- Entry #5: ✅ Valid (links to #4)
- Entry #6: ✅ Valid (links to #5)

**Chain Head:** c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6
```

---

## Notes

- Meta-Ledger is **append-only** - never delete or edit entries
- L3 decisions **block implementation** until approved
- Hash chain enables **tamper detection** - any modification breaks chain
- Decisions are **reversible** but reversals must also be logged
- **Future decisions** can reference past entries by ID
- **Quarterly audits** should validate chain integrity

---

**Remember:** The meta-ledger isn't bureaucracy - it's institutional memory. When someone asks "why did we choose X?", the answer is in the ledger with full context and rationale.

