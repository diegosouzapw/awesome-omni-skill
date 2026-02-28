---
name: tidar
description: Triadic Interleaving Dispatch with Agents for Reading/writing. Pre-hooks
  split into 3³=27 sub-agents before any read/write, collapse via middle path. Use
  for parallel ACSet access, Goblins vat coordination, and GF(3)-conserved agency
  trees.
metadata:
  interface_ports:
  - Commands
trit: 0
---
# TIDAR - Triadic Interleaving Dispatch with Agents for Reading/writing

> *Before any interaction happens at the right time, split → recurse → collapse*

**Trit**: 0 (ERGODIC - Coordinator)  
**GF(3) Triad**: `goblins (-1) ⊗ tidar (0) ⊗ agent-o-rama (+1) = 0 ✓`

## Core Principle

Every read/write operation spawns a **triadic tree** of sub-agents:

```
                    ROOT (0)
                   /   |   \
                -1     0    +1          ← depth 1 (3 agents)
               /|\    /|\   /|\
             -0+ -0+ -0+ -0+ ...        ← depth 2 (9 agents)
             /|\ /|\ /|\ /|\ ...
           -0+-0+-0+-0+-0+-0+...        ← depth 3 (27 leaves)
```

| Level | Agents | Role |
|-------|--------|------|
| ROOT | 1 | Coordinator |
| Depth 1 | 3 | Direction choosers |
| Depth 2 | 9 | Validators |
| Depth 3 | 27 | Leaf executors |
| **Total** | **40** | GF(3) = 0 ✓ |

## The Three Directions

| Trit | Symbol | Name | Role |
|------|--------|------|------|
| -1 | `-` | MINUS | Contractive/sink - reject, filter, reduce |
| 0 | `0` | MIDDLE | Ergodic/transform - neutral, pass-through |
| +1 | `+` | PLUS | Expansive/source - accept, generate, amplify |

## Pre-Hook Protocol

### Before Read

```python
async def pre_read_hook(target, accessor):
    tree = DiscoHyTree(max_depth=3)
    
    # 27 agents probe in parallel
    for agent in tree.leaves():
        offset = sum(t.value for t in agent.path)
        agent.result = accessor(target, offset=offset)
    
    # Collapse via middle path
    return tree.collapse_to_middle()
```

### Before Write

```python
async def pre_write_hook(target, data, writer):
    tree = DiscoHyTree(max_depth=3)
    
    # 27 agents validate
    for agent in tree.leaves():
        if agent.trit == Trit.MINUS:
            agent.result = {"action": "reject"}
        elif agent.trit == Trit.PLUS:
            agent.result = {"action": "accept", "data": data}
        else:
            agent.result = {"action": "transform", "data": data}
    
    # Collapse: middle path decides
    decision = tree.collapse_to_middle()
    
    if decision.action in ["accept", "transform"]:
        return writer(target, data)
```

## Collapse Algorithm

When max depth reached, collapse upward via **middle path preference**:

```python
def collapse_node(agent):
    if agent.is_leaf:
        return agent.result
    
    child_results = {c.trit: collapse_node(c) for c in agent.children}
    
    # Prefer middle path
    if child_results.get(Trit.MIDDLE) is not None:
        return child_results[Trit.MIDDLE]
    
    # Otherwise: PLUS - MINUS (GF(3) balance)
    return child_results[Trit.PLUS] - child_results[Trit.MINUS]
```

## DiscoHy Integration

TIDAR uses Hy (Hylang) for homoiconic sub-agent definitions:

```hy
#!/usr/bin/env hy
(import tidar [Trit SubAgent DiscoHyTree])

;; Define triadic tree
(setv tree (DiscoHyTree :max-depth 3))

;; Each leaf has a path like (MINUS PLUS MIDDLE)
(for [agent (tree.leaves)]
  (print f"Agent {agent.id}: path-sum = {(agent.path-trit-sum)}"))

;; GF(3) conservation check
(assert (= 0 (sum (lfor a (tree.leaves) a.trit.value))))
```

## Goblins Vat Bridge

Each object type gets its own **vat** (isolated execution context):

```python
class VatNetwork:
    def __init__(self, acset):
        for ob in acset.schema["objects"]:
            self.vats[ob] = ObjectTypeActor(acset, ob)
    
    def near_ref(self, object_type):
        """$ operator - sync call within vat"""
        return NearRef(self.vats[object_type])
    
    async def far_send(self, object_type, method, *args):
        """<- operator - async message across vats"""
        return await self.executor.submit(
            getattr(self.vats[object_type], method), *args
        )
```

## ACSet Random Access

TIDAR provides parallel random access to ACSet parts:

```python
bridge = TIDARBridge(acset)

# Read with 27-agent pre-hook
result = await bridge.read_part("Content", idx=100)

# Write with 27-agent validation
result = await bridge.write_subpart("parent_of", src=1000, tgt=999)
```

## CAT# Concept Availability

Prove availability of categorical concepts across ACSet:

```python
CAT_CONCEPTS = {
    "category": ["objects", "morphisms", "identity", "compose"],
    "functor": ["object_map", "morphism_map"],
    "natural_transformation": ["component", "naturality_square"],
    "adjunction": ["left_adjoint", "right_adjoint", "unit", "counit"],
    "monad": ["unit", "multiplication"],
    "topos": ["subobject_classifier", "exponential"],
}

# Parallel search across all vats
bitmap = await prover.full_availability_scan()
# Returns: {concept: {object_type: count}}
```

## GF(3) Conservation

At every level, the sum of trits = 0:

| Property | Count | Value |
|----------|-------|-------|
| MINUS leaves | 9 | -9 |
| MIDDLE leaves | 9 | 0 |
| PLUS leaves | 9 | +9 |
| **Sum** | 27 | **0 ✓** |

## File Structure

```
zip_acset_skill/
├── openai_acset.py              # ACSet from ChatGPT export
├── parallel_availability_proof.py  # CAT# concept search
├── goblins_agent_bridge.py      # Vat network + capabilities
├── discohigh_interleave.py      # Triadic tree + pre-hooks
├── interactivity_proof.json     # Exported proof
└── discohigh_state.json         # Tree state + interaction log
```

## Triad Formation

TIDAR forms triads with:

| Trit | Skill | Role |
|------|-------|------|
| -1 | goblins | Vat isolation, capability refs |
| 0 | **tidar** | Triadic dispatch coordinator |
| +1 | agent-o-rama | Pattern extraction from results |

**Conservation**: (-1) + (0) + (+1) = 0 ✓

## When to Use TIDAR

- **Parallel ACSet access** with provable availability
- **Goblins vat coordination** across object types
- **Read/write validation** with consensus
- **GF(3)-conserved agency** trees
- **DiscoHy operadic** composition

## See Also

- `discohy-streams` - 7 operad variants with TAP streams
- `goblins` - Distributed object capabilities
- `agent-o-rama` - Pattern extraction and learning
- `cat-tripartite` - SICP/CTP/CatColab worldlines
- `triad-interleave` - GF(3) balanced scheduling

---

## End-of-Skill Interface

## Commands

```bash
# Run parallel availability proof
python3 parallel_availability_proof.py conversations.json

# Run Goblins bridge
python3 goblins_agent_bridge.py conversations.json

# Run full TIDAR interleaving
python3 discohigh_interleave.py conversations.json
```


---

## Autopoietic Marginalia

> **The interaction IS the skill improving itself.**

Every use of this skill is an opportunity for worlding:
- **MEMORY** (-1): Record what was learned
- **REMEMBERING** (0): Connect patterns to other skills  
- **WORLDING** (+1): Evolve the skill based on use



*Add Interaction Exemplars here as the skill is used.*
