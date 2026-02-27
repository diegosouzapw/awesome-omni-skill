---
name: safe-feature-addition
description: Elite guide for shipping features without breaking production. Universal patterns for JS, TS, Python, Go, Rust, and more. Implements "Additive over Destructive" development, Unified Safety Auditing, and Feature Flags.
version: 3.1.0
author: sufianmypa1203-oss
quality_score: 100
triggers:
  - "how to add feature safely"
  - "don't break production"
  - "feature flags"
  - "safe rollout"
  - "strangler fig"
  - "branch by abstraction"
---

# 🛡️ Safe Feature Addition: Ship Without Breaking

## What This Does

This skill provides a **Universal Safety Net** for adding new features to production systems across any framework or language. It implements an **"Additive Development"** philosophy where new code lives alongside old code, protected by feature flags and gradual rollouts.

> [!IMPORTANT]
> **The Golden Rule:** Always favor *additive* changes over *destructive* modifications. Existing code should remain untouched and functional.

---

## 🏗️ The Production Toolkit

This skill is powered by a unified safety engine compatible with **JS, TS, Python, Go, Rust, and Ruby**:

| Tool | Purpose |
|------|---------|
| `safe-feature verify` | Ensures all flags in code are defined in config. |
| `safe-feature audit` | Scans git diff for destructive changes (signature breaks). |
| `feature-flags.yml` | Central configuration for feature toggles. |

---

## 🚀 Step 1: Additive Development (Universal)

Instead of changing an existing function, extend it or create a new one.

### JavaScript / TypeScript
```javascript
// ✅ ELITE & ADDITIVE - Backward compatible
function processPayment(amount, method = 'credit-card') {
  if (method === 'crypto') return crypto(amount);
  return legacyPayment(amount);
}
```

### Python
```python
# ✅ ELITE & ADDITIVE - Backward compatible
def process_payment(amount, method='credit_card'):
    if method == 'crypto': return crypto(amount)
    return legacy_payment(amount)
```

---

## 🚩 Step 2: Feature Flags (The Kill Switch)

Wrap all new logic in feature flags. This allows you to deploy code to production while it is still "dormant".

### 1. Define the Flag (YAML/JSON)
```yaml
# feature-flags.yml
new_feature_v2:
  enabled: false
  rollout_percentage: 0  # Start at 0% traffic
```

### 2. Guard the Code
```typescript
// Works in React, Vue, Svelte, etc.
if (flags.isEnabled('new-feature-v2', user.id)) {
  return <NewComponent />;
}
return <LegacyComponent />;
```

### 3. Verify Deployment
Before you ship, run the safety check to ensure config matches code:
```bash
python scripts/safe-feature.py verify --path ./src --config ./feature-flags.yml
```

---

## 🔍 Step 3: Git Safety Audit

Before merging your PR, run the **Universal Safety Auditor** to detect destructive changes that might break production.

```bash
# Audits your current branch against master/main
python scripts/safe-feature.py audit --base main
```

**The Auditor detects:**
- Modified function signatures without default values.
- New code blocks without feature flag guards.
- Potential "Big Bang" changes that increase risk.

---

## 📈 Step 4: Gradual Rollout (Canary Schedule)

Never switch a feature to 100% on Day 1. Use an industry-standard gradual schedule:

| Phase | Traffic | Duration | Goal |
|-------|---------|----------|------|
| **Canary** | 5% | 1 hour | Check for smoke/crash loops |
| **Early Bird** | 25% | 1 day | Check for performance bugs |
| **Rollout** | 50% | 2 days | Monitor business metrics |
| **GA** | 100% | Permanent | Complete the rollout |

---

## 🛠️ Performance & Safety Checklist

- [ ] Is the change **additive**? (Yes/No)
- [ ] Is there a **Feature Flag**? (Yes/No)
- [ ] Did you run `safe-feature verify`? (Yes/No)
- [ ] Is the **Database Migration** backward-compatible? (Yes/No)
- [ ] Is there a **Kill Switch**? (Yes/No)

---

**Success = Zero Downtime + Instant Rollback.** 🚀
