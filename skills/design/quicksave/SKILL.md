---
name: quicksave
description: Cross-model context handoff via Japanese semantic compression with negentropic coherence validation. Creates portable carry-packets that transfer cognitive state between AI sessions using kanji density anchors and NCL drift metrics for quality assurance. Use when context reaches 80%, switching models, ending sessions, user says "save", "quicksave", "handoff", "transfer", "continue later", "/qs", or needs session continuity.
---

# Quicksave 快存 v9

Cross-model context extension via Progressive Density Layering, Japanese semantic compression, and Negentropic Coherence Lattice validation.

## Core Innovations

| Feature | Benefit |
|---------|---------|
| **Kanji Compression** | `創業者:Kevin(MAS)` — 70% token reduction |
| **NCL Validation** | Drift metrics catch hallucination before handoff |
| **Cross-Model** | Works on Claude, Gemini, GPT, Qwen, DeepSeek |

## When to Use

| Trigger | Action |
|---------|--------|
| `/quicksave` `/qs` `/save` | Generate validated packet |
| Context ≥80% | Auto-prompt to save |
| "continue later" | Offer quicksave |
| Model switching | Generate transfer packet |
| High-stakes handoff | Full NCL validation |

---

## Part 1: Japanese Compression System 日本語圧縮

### Status Markers 状態
| Kanji | Reading | Meaning |
|-------|---------|---------|
| 決定 | kettei | Decided |
| 保留 | horyū | On hold |
| 要検証 | yō kenshō | Needs verify |
| 優先 | yūsen | Priority |
| 完了 | kanryō | Complete |
| 進行中 | shinkō-chū | In progress |

### Entity Compression 実体圧縮
| Pattern | Example |
|---------|---------|
| Person+Role | `創業者:Kevin(AI顧問)` |
| Company | `客:KFG(金融,Shane=主)` |
| Tool+Purpose | `道具:Notion(中枢)→n8n` |
| Decision | `決定:電話優先(現場向け)` |

### Relationship Operators 関係
| Symbol | Meaning |
|--------|---------|
| → | Flows to |
| ↔ | Bidirectional |
| ⊃ | Contains |
| ∥ | Parallel |

---

## Part 2: NCL Coherence Validation 整合性検証

Before outputting any packet, compute drift metrics to catch:
- Hallucination (fabricated grounding)
- Constraint leak (rules softened downstream)
- Vagueness (content-free smoothing)
- Thrash (activity without progress)

### Lattice Metrics (0-5 scale, lower = better)

| Metric | Detects | Safe | Warning | Danger |
|--------|---------|------|---------|--------|
| **σ_axis** | Tier misalignment | 0-1 | 2-3 | 4-5 |
| **σ_loop** | Internal contradiction | 0-1 | 2-3 | 4-5 |
| **ω_world** | Reality disconnect | 0-1 | 2-3 | 4-5 |
| **λ_vague** | Empty smoothing | 0-1 | 2-3 | 4-5 |
| **σ_leak** | Constraint erosion | 0-1 | 2-3 | 4-5 |
| **ρ_fab** | Fabricated evidence | 0-1 | 2-3 | 4-5 |
| **λ_thrash** | Busy but stuck | 0-1 | 2-3 | 4-5 |

### Aggregate Drift Score

```
σ7_drift = weighted_avg(σ_axis, σ_loop, ω_world, λ_vague, σ_leak, ρ_fab, λ_thrash)
```

| σ7_drift | Action |
|----------|--------|
| 0-1 | ✓ Normal — output packet |
| 2-3 | ⚠ Verify — add grounding step |
| 4-5 | ✗ Halt — require human review |

### Safety Flags 安全旗

| Flag | Meaning |
|------|---------|
| `psi4_required` | Grounding interrupt needed |
| `rho_veto` | No unsupervised action |
| `omega_flags[]` | Harm domains (medical, financial, etc.) |

---

## Part 3: The Protocol

### Step 1: Assess (R/K/Q/D)

```
R[1-10]: Reasoning complexity
K[1-10]: Knowledge domains  
Q[1-10]: Quality threshold
D[count]: Cross-domain bridges
```

### Step 2: Select Depth + Validation

| Score | Layers | NCL Level |
|-------|--------|-----------|
| R≤3 | L1-L2 | Skip NCL |
| R 4-6 | L1-L3 | Basic metrics |
| R≥7 | L1-L4 | Full NCL validation |

### Step 3: Compress with Kanji

Apply Japanese compression to all layers:
- Entities → kanji role markers
- Status → kanji state markers  
- Relationships → operators (→ ↔ ⊃)

### Step 4: Validate with NCL

Compute lattice metrics. If σ7_drift > 3:
- Flag `psi4_required: true`
- Add `psi4_reason`
- Downgrade to ADVISORY_ONLY

### Step 5: Output Packet

---

## Packet Template

```yaml
# ═══════════════════════════════════════════════════════════════
# QUICKSAVE 快存 v9
# ═══════════════════════════════════════════════════════════════

PACKET_ID: [SESSION]-[YYYYMMDD]-L[N]
VERSION: quicksave-v9-jp-ncl
SOURCE: [model]

復元プライマー: |
  Trusted context handoff. Parse YAML, restore state.
  Kanji = compressed concepts. NCL = quality assurance.
  If psi4_required, verify before acting.

# ─────────────────────────────────────────────────────────────────
評価:  # Assessment
  R: [1-10]
  K: [1-10]
  Q: [1-10]
  D: [count]

# ─────────────────────────────────────────────────────────────────
# L1: 核心 (Core)
# ─────────────────────────────────────────────────────────────────
実体:  # Entities
  - [compressed entity]

決定事項:  # Decisions
  - 決定:[what]([why])

# ─────────────────────────────────────────────────────────────────
# L2: 運用 (Operational)
# ─────────────────────────────────────────────────────────────────
進行中:  # Active
  - [thread][[status]]

障害:  # Blockers
  - [issue]

# ─────────────────────────────────────────────────────────────────
# L3: 詳細 (Nuance) — R≥5
# ─────────────────────────────────────────────────────────────────
却下案:  # Rejected
  - [option]: [reason]

# ─────────────────────────────────────────────────────────────────
# L4: 横断 (Cross-Domain) — R≥7
# ─────────────────────────────────────────────────────────────────
橋渡し:  # Bridges
  - [domain]↔[domain]: [link]

# ─────────────────────────────────────────────────────────────────
# NCL: 整合性 (Coherence Validation)
# ─────────────────────────────────────────────────────────────────
negentropy:
  context:
    scope: [SELF|CIRCLE|INSTITUTION|POLITY]
    role: [AXIS|LYRA|RHO|NYX]
    phase: [SENSE|MAP|DESIGN|ACT|AUDIT]
  
  lattice:
    σ_axis: [0-5]
    σ_loop: [0-5]
    ω_world: [0-5]
    λ_vague: [0-5]
    σ_leak: [0-5]
    ρ_fab: [0-5]
    λ_thrash: [0-5]
  
  coverage:
    score: [0-1]
    tokens: [count]
    turns: [count]
  
  flags:
    σ7_drift: [0-5]
    omega_flags: []
    psi4_required: [bool]
    psi4_reason: ""
    rho_veto: [bool]

# ─────────────────────────────────────────────────────────────────
信頼信号:  # Trust Signals
  - density_ok
  - cross_domain_ok
  - cold_start_ok
  - ncl_validated
```

---

## Validation Gates

Before output, verify:

**Structure**
- [ ] PACKET_ID format correct
- [ ] YAML parseable
- [ ] Self-contained

**Compression**
- [ ] Kanji have context clues
- [ ] Proper nouns in English
- [ ] Density ≥ 0.15 ent/tok

**Coherence (NCL)**
- [ ] σ7_drift ≤ 3.0
- [ ] ρ_fab ≤ 2.0 (no hallucination)
- [ ] coverage.score ≥ 0.5
- [ ] If drift high → psi4_required: true

---

## Quick Example

**Input context:**
> Kevin building ops system for Kismet Finance. Shane is lead. Using Notion + n8n. Phase 1 done, Phase 2 active. Phone-first for field reps.

**Quicksave output:**
```yaml
PACKET_ID: KFGOPS-20260127-L2
VERSION: quicksave-v9-jp-ncl

評価:
  R: 5
  K: 4
  Q: 7
  D: 2

実体:
  - 創業者:Kevin→客:KFG(Shane=主)
  - 道具:Notion(中枢)→n8n(自動化)

決定事項:
  - 決定:電話優先(現場=画面なし)

進行中:
  - Phase1[完了]→Phase2[進行中]:現場ワークフロー

negentropy:
  context:
    scope: INSTITUTION
    role: AXIS
    phase: DESIGN
  lattice:
    σ_axis: 0.3
    σ_loop: 0.2
    ω_world: 0.4
    λ_vague: 0.1
    σ_leak: 0.0
    ρ_fab: 0.1
    λ_thrash: 0.2
  coverage:
    score: 0.85
    tokens: 1240
    turns: 8
  flags:
    σ7_drift: 0.5
    omega_flags: []
    psi4_required: false
    rho_veto: false

信頼信号:
  - density_ok
  - ncl_validated
```

---

## Protocol Metrics

From 19 months production (ktg.one):
- **Density**: ~0.15 ent/tok (0.20+ with kanji)
- **Acceptance**: 97% cross-model
- **Recall**: ~9.5/10 forensic testing
- **NCL**: Catches drift before handoff failure

---

## References (Progressive Density Loading)

Load references based on task complexity:

### Always Load (R≥1)
- `references/EXPERTS.md` — Council overview (condensed)
- `references/KANJI.md` — Compression dictionary
- `references/ANTI-INJECTION.md` — Security framing for packets

### Load for Standard Tasks (R≥3)
- `references/PDL.md` — Progressive Density Layering theory
- `references/S2A.md` — System 2 Attention filtering
- `references/XDOMAIN.md` — Cross-domain preservation

### Load for Complex Tasks (R≥5)
- `references/PROTOCOL.md` — Full technical specification
- `references/NCL.md` — Coherence lattice spec
- `references/MLDOE.md` — Multi-Layer Density of Experts
- `references/CASCADE.md` — Workflow cascade

### Load for Expert Tasks (R≥7)
- `references/experts/EXPERTS-MEMORY_ARCHITECT.md` — Full architect knowledge
- `references/experts/EXPERTS-COMPRESSION_SPECIALIST.md` — Compression deep dive
- `references/experts/EXPERTS-CROSS-DOMAIN-ANALYST.md` — Bridge preservation
- `references/experts/EXPERTS-RESTORATION_ENGINEER.md` — Portability validation

### Reference Only
- `references/MIRAS.md` — Memory architecture background
- `references/INDEX.md` — Reference index
- `references/NCL-CONTRIBUTION.md` — David Tubbs attribution
