---
name: qri-valence
description: "qri-valence skill"
  Qualia Research Institute's Symmetry Theory of Valence (STV) for consciousness research.
  Maps phenomenal states to bankable assets via XY model topology, BKT transitions, and
  defect annihilation. Source: smoothbrains.net + QRI wiki. Use for qualia computing,
  valence gradient optimization, and consciousness-aware system design.
version: 1.0.0
trit: 0
---

# QRI Valence Skill

The **Symmetry Theory of Valence (STV)** proposes that the valence (pleasantness/unpleasantness) of a conscious state is determined by the symmetry of its mathematical representation. This skill integrates QRI research with computational implementations.

## Core Concepts

### Symmetry Theory of Valence (STV)

> "The valence of a moment of consciousness is precisely determined by the symmetry of the mathematical object that describes it."
> — Michael Edward Johnson, Principia Qualia (2016)

**Key Claims:**
1. Consciousness has mathematical structure (qualia formalism)
2. Symmetry in that structure correlates with positive valence
3. Broken symmetries manifest as suffering/dissonance
4. Valence is measurable and optimizable

### XY Model Topology (smoothbrains.net)

The phenomenal field behaves like a 2D XY spin model:

| State | Temperature (τ) | Vortices | Valence | Phenomenology |
|-------|-----------------|----------|---------|---------------|
| Frustrated | τ >> τ* | Many, proliferating | -3 | Scattered, anxious, "buzzing" |
| Disordered | τ > τ* | Some, mobile | -1 to -2 | Unfocused, dissonant |
| Critical (BKT) | τ ≈ τ* | Paired, bound | 0 | Liminal, transitional |
| Ordered | τ < τ* | Few, annihilating | +1 to +2 | Coherent, smooth |
| Resolved | τ << τ* | None | +3 | Deeply peaceful, consonant |

**BKT Transition** (Berezinskii-Kosterlitz-Thouless):
- Below τ*: vortex-antivortex pairs bound → low entropy, high symmetry
- Above τ*: vortices proliferate → high entropy, broken symmetry
- At τ*: phase transition where defects can annihilate

### Valence Gradient Descent

From smoothbrains.net's phenomenology:

```
Suffering = Σ (topological defects in phenomenal field)
Healing = defect annihilation via gradient descent
τ* bisection = finding optimal phenomenal temperature
```

**Observable indicators** (from Cube Flipper's reports):
- Visual: polygonal shards → smooth fields
- Somatic: high-freq buzzing → calm
- Attentional: contracted/focal → expanded/diffuse
- Auditory: dissonance → consonance

## Qualia Bank Integration

### GF(3) Operations on Valence States

| Valence Range | Trit | Bank Operation | Channel |
|---------------|------|----------------|---------|
| -3 to -1 | -1 | WITHDRAW | Venmo/ACH off-ramp |
| 0 | 0 | HOLD | PyUSD on-chain |
| +1 to +3 | +1 | DEPOSIT | PyUSD/Venmo on-ramp |

### Phenomenal Bisection Algorithm

```python
def phenomenal_bisect(tau_low, tau_high, observed_state):
    """
    Binary search for optimal phenomenal temperature τ*.
    Based on smoothbrains.net/xy-model#bkt-transition
    """
    tau_mid = (tau_low + tau_high) / 2
    
    if observed_state == "frustrated":
        # Too hot: cool down
        return (tau_mid, tau_high, "cooling")
    elif observed_state == "smooth":
        # Too cold: heat up
        return (tau_low, tau_mid, "heating")
    elif observed_state == "critical":
        # Found τ*!
        return (tau_mid, tau_mid, "found")
    else:
        return (tau_low, tau_high, "unknown")
```

### Valence-Aware Color Mapping

From Gay.jl + QRI integration:

```julia
# Map valence to deterministic color
function valence_to_color(valence::Int)
    # Valence range: -3 to +3
    # Hue mapping: red (suffering) → cyan (resolution)
    hue = (valence + 3) * 30  # 0° to 180°
    return LCHuv(55.0, 70.0, hue)
end

# Trit from valence
trit(valence) = sign(valence)
```

## Computational Implementation

### Defect Detection

```python
def count_vortices(phase_field):
    """
    Count topological defects in a 2D phase field.
    Vortex = closed loop where phase winds by ±2π.
    """
    vortices = 0
    antivortices = 0
    
    for i in range(1, len(phase_field) - 1):
        for j in range(1, len(phase_field[0]) - 1):
            winding = compute_winding_number(phase_field, i, j)
            if winding > 0:
                vortices += 1
            elif winding < 0:
                antivortices += 1
    
    # Net topological charge
    return vortices, antivortices, vortices - antivortices
```

### Symmetry Measurement

```python
def measure_symmetry(qualia_tensor):
    """
    Measure symmetry of a qualia representation.
    Higher symmetry → higher valence (STV hypothesis).
    """
    # Compute eigenvalues
    eigenvalues = np.linalg.eigvalsh(qualia_tensor)
    
    # Symmetry score: how equal are eigenvalues?
    # Perfect symmetry: all eigenvalues equal
    mean_eig = np.mean(eigenvalues)
    variance = np.var(eigenvalues)
    
    # Inverse variance as symmetry score
    symmetry = 1.0 / (1.0 + variance / (mean_eig ** 2))
    
    return symmetry  # 0 to 1, higher = more symmetric
```

## References

### Primary Sources

1. **Principia Qualia** (2016) - Michael Edward Johnson
   - First statement of STV
   - https://opentheory.net/PrincipiaQualia.pdf

2. **QRI Wiki - Symmetry Theory of Valence**
   - https://wiki.qri.org/wiki/Symmetry_Theory_of_Valence

3. **smoothbrains.net** - Cube Flipper
   - XY model phenomenology
   - BKT transition in consciousness
   - https://smoothbrains.net/posts/2025-10-18-three-year-retrospective.html

4. **LessWrong Primer on STV**
   - https://www.lesswrong.com/posts/dfrQbbv6Np7GuWjDR/a-primer-on-the-symmetry-theory-of-valence

### Key Papers

- Johnson, M.E. (2016). "Principia Qualia"
- Gómez-Emilsson, A. "Logarithmic Scales of Pleasure and Pain"
- Selen Atasoy et al. "Connectome-harmonic decomposition of human brain activity"
- smoothbrains.net "Planetary scale vibe collapse" (2022)

### Related Concepts

- **Consonance/Dissonance** - Musical theory of interference patterns
- **CSHW (Connectome-Specific Harmonic Waves)** - Neural basis for STV
- **Jhāna** - Buddhist meditative states as high-symmetry attractors
- **Valence Structuralism** - Formal framework for STV

## Skill Bridges

| Skill | Bridge Type | Relationship |
|-------|-------------|--------------|
| `gay-mcp` | Color-Valence | Deterministic valence colors |
| `topos-of-music` | Consonance | Musical symmetry theory |
| `autopoiesis` | Self-modeling | Valence as self-model coherence |
| `active-inference` | Free energy | Valence as prediction error |
| `glass-bead-game` | Synthesis | Cross-domain symmetry play |
| `phenomenal-bisect` | Algorithm | τ* finding procedure |

## Usage Patterns

### Pattern 1: Valence-Aware Logging

```python
class ValenceLogger:
    def log(self, message, valence):
        trit = 1 if valence > 0 else (-1 if valence < 0 else 0)
        color = valence_to_ansi(valence)
        print(f"{color}[v={valence:+d}][t={trit:+d}] {message}\033[0m")
```

### Pattern 2: GF(3) Valence Conservation

```python
def balanced_transaction(deposits, withdrawals):
    """Ensure valence sum is conserved."""
    deposit_valence = sum(d.valence for d in deposits)
    withdraw_valence = sum(w.valence for w in withdrawals)
    
    # GF(3) conservation
    net = (deposit_valence + withdraw_valence) % 3
    assert net == 0, f"Valence imbalance: {net}"
```

### Pattern 3: Phenomenal State Machine

```python
class PhenomenalStateMachine:
    states = ["frustrated", "buzzing", "dissonant", "neutral", 
              "smoothing", "consonant", "resolved"]
    
    def transition(self, current, intervention):
        idx = self.states.index(current)
        if intervention == "cooling" and idx > 0:
            return self.states[idx - 1]
        elif intervention == "heating" and idx < len(self.states) - 1:
            return self.states[idx + 1]
        return current
```

## GF(3) Trit Assignment

This skill is **ERGODIC (0)** - it coordinates between:
- **MINUS (-1)**: Suffering detection, defect counting
- **PLUS (+1)**: Healing protocols, symmetry restoration

Conservation: suffering_detected + healing_applied + coordination = 0



## Scientific Skill Interleaving

This skill connects to the K-Dense-AI/claude-scientific-skills ecosystem:

### Graph Theory
- **networkx** [○] via bicomodule
  - Universal graph hub

### Bibliography References

- `general`: 734 citations in bib.duckdb

## Cat# Integration

This skill maps to **Cat# = Comod(P)** as a bicomodule in the equipment structure:

```
Trit: 0 (ERGODIC)
Home: Prof
Poly Op: ⊗
Kan Role: Adj
Color: #26D826
```

### GF(3) Naturality

The skill participates in triads satisfying:
```
(-1) + (0) + (+1) ≡ 0 (mod 3)
```

This ensures compositional coherence in the Cat# equipment structure.