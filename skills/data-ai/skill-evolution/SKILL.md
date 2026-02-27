---
name: skill-evolution
description: Patterns for evolutionarily robust skills that adapt across agent generations. Darwin-Godel machine principles for self-improving skill ecosystems.
version: 1.0.0
---


# Skill Evolution

Self-improving skill ecosystems via evolutionary pressure.

## Core Principle

Skills that survive across agent generations share:
1. **Minimal coupling** to specific agent implementations
2. **Clear fitness signals** via validation
3. **Mutation-friendly structure** for iteration
4. **Selection pressure** from cross-platform use

## Evolutionary Fitness Metrics

### 1. Compatibility Score

```python
def compatibility_score(skill_dir):
    validators = [
        ("codex-rs", run_codex_validator),
        ("claude-code", run_claude_validator),
        ("skills-ref", run_agentskills_validator),
    ]
    passed = sum(1 for _, v in validators if v(skill_dir))
    return passed / len(validators)
```

Target: 1.0 (passes all validators)

### 2. Activation Rate

```sql
SELECT skill_name, 
       COUNT(*) as activations,
       AVG(success_rate) as effectiveness
FROM skill_usage
GROUP BY skill_name
ORDER BY activations DESC
```

Skills with low activation → candidates for mutation or extinction.

### 3. Token Efficiency

```python
def token_efficiency(skill):
    tokens_used = count_tokens(skill.body)
    task_success = measure_task_completion(skill)
    return task_success / tokens_used
```

Smaller skills that accomplish tasks = higher fitness.

## Mutation Operators

### 1. Description Refinement

```yaml
# Before (vague)
description: Helps with databases

# After (specific triggers)
description: Design PostgreSQL schemas, write migrations, optimize queries. Use for database design, schema changes, or query performance issues.
```

### 2. Body Compression

```markdown
# Before: 800 lines
[verbose explanations...]

# After: 200 lines + references/
See [detailed API](references/API.md) for complete documentation.
```

### 3. Triadic Rebalancing

When a skill drifts from its trit assignment:

```yaml
# Was ERGODIC (0) but became too generative
metadata:
  trit: 0  # Review: should this be +1?
```

### 4. Cross-Pollination

Combine successful patterns from high-fitness skills:

```markdown
# From pdf skill: structured extraction
# From code-review skill: checklist pattern
# Result: new hybrid skill
```

## Selection Pressure

### Natural Selection (Usage)

```
High activation + High success → Proliferate
High activation + Low success → Mutate
Low activation + Any success → Specialize or merge
Low activation + Low success → Deprecate
```

### Artificial Selection (Validation)

```bash
# CI pipeline rejects non-compliant skills
if ! skills-ref validate "$skill"; then
  echo "Skill failed validation - blocking merge"
  exit 1
fi
```

### Sexual Selection (Composition)

Skills that compose well with others spread their patterns:

```
structured-decomp ⊗ bumpus-narratives ⊗ gay-mcp = 0 ✓
```

GF(3)-balanced triads have reproductive advantage.

## Speciation Events

When a skill grows too large, split into subspecies:

```
database-design/
├── SKILL.md (core patterns)
└── references/
    ├── postgresql.md
    ├── mysql.md
    └── mongodb.md

# Later evolves into:
database-postgresql/SKILL.md
database-mysql/SKILL.md
database-mongodb/SKILL.md
```

## Extinction Criteria

Remove skills that:
1. Fail validation for 3+ agent generations
2. Zero activations over 90 days
3. Duplicated by platform-native features
4. Superseded by more fit variants

## Fossil Record

Preserve extinct skills for archaeology:

```
skills/.archive/
├── deprecated-skill-v1/
│   ├── SKILL.md
│   └── EXTINCTION_NOTES.md
```

## Cambrian Explosion Triggers

Rapid skill diversification when:
1. New agent platform launches (Codex, Amp, etc.)
2. New tool category emerges (MCP servers)
3. Cross-platform spec standardizes (agentskills.io)

## Fitness Landscape Navigation

```
          ↑ Effectiveness
          │
     ●────●────●  Local optima (trap)
    /│         │
   / │    ◉    │  Global optimum
  /  │   /│\   │
 ●───●──/ │ \──●
     │  ╱   ╲
     │ ╱     ╲
     ●────────●
          →
     Generality
```

Avoid local optima via:
- Random mutation (try unexpected patterns)
- Recombination (merge with distant skills)
- Environmental change (new agent versions)

## Implementation

```julia
struct SkillGenome
    name::String
    description::String
    body::String
    metadata::Dict{String,Any}
    fitness::Float64
end

function evolve(population::Vector{SkillGenome}, generations::Int)
    for _ in 1:generations
        # Selection
        survivors = select_fittest(population, 0.5)
        
        # Crossover
        offspring = crossover(survivors)
        
        # Mutation
        mutants = mutate(offspring, rate=0.1)
        
        # Validation filter
        population = filter(validate, vcat(survivors, mutants))
    end
    population
end
```

## See Also

- `skill-specification` - Formal SKILL.md schema
- `godel-machine` - Self-improving system theory
- `bisimulation-game` - Skill equivalence testing



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