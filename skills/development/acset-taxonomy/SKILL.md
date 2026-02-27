---
name: acset-taxonomy
description: Taxonomy of ACSet skills with morphisms to semantically similar categorical/relational skills
license: MIT
metadata:
  trit: 0
  color: '#4A90D9'
  gf3_role: ERGODIC
  version: 1.0.0
---

# ACSet Skill Taxonomy

**Trit**: 0 (ERGODIC - coordinates between schema validation and data generation)

## Overview

This skill maps the ecosystem of ACSet-based skills and their morphisms to semantically similar categorical/relational skills.

```
                    ┌─────────────────────────────────────────┐
                    │         ACSET SKILL UNIVERSE            │
                    └─────────────────────────────────────────┘
                                      │
           ┌──────────────────────────┼──────────────────────────┐
           │                          │                          │
    ┌──────▼──────┐           ┌───────▼───────┐          ┌───────▼───────┐
    │  EXPLICIT   │           │  DOMAIN-      │          │  SEMANTICALLY │
    │  ACSET      │           │  SPECIFIC     │          │  SIMILAR      │
    │  CORE       │           │  ACSET        │          │  CATEGORICAL  │
    └─────────────┘           └───────────────┘          └───────────────┘
         (3)                       (12)                       (12)
```

---

## I. EXPLICIT ACSET CORE (3 skills)

Foundation skills that define ACSet theory and operations:

| Skill | Description | Schema Pattern | Trit |
|-------|-------------|----------------|------|
| [acsets](file:///Users/bob/.claude/skills/acsets/SKILL.md) | Algebraic databases with Specter navigation | `Ob::, Hom::, Attr::` | 0 |
| [acsets-relational-thinking](file:///Users/bob/.claude/skills/acsets-relational-thinking/SKILL.md) | Functors X: C → Set, DPO rewriting | Category → Set | -1 |
| [specter-acset](file:///Users/bob/.claude/skills/specter-acset/SKILL.md) | Bidirectional navigation, inline caching | Lenses/Prisms | +1 |

**GF(3) Triad**: `acsets-relational-thinking (-1) ⊗ acsets (0) ⊗ specter-acset (+1) = 0 ✓`

---

## II. DOMAIN-SPECIFIC ACSET (12 skills)

ACSet schemas instantiated for specific domains:

### A. Google Workspace ACSet Family

| Skill | Domain | Schema Objects | MCP Equivalence |
|-------|--------|----------------|-----------------|
| [calendar-acset](file:///Users/bob/.claude/skills/calendar-acset/SKILL.md) | Google Calendar | Event, Attendee, Recurrence | calendar-mcp |
| [docs-acset](file:///Users/bob/.claude/skills/docs-acset/SKILL.md) | Google Docs/Sheets | Document, Comment, Cell | docs-mcp |
| [drive-acset](file:///Users/bob/.claude/skills/drive-acset/SKILL.md) | Google Drive | File, Folder, Permission | drive-mcp |
| [tasks-acset](file:///Users/bob/.claude/skills/tasks-acset/SKILL.md) | Google Tasks | Task, TaskList, Due | tasks-mcp |

**Common Pattern**: Transform API operations → GF(3)-typed Interactions → triadic queues

### B. Data Import/Export ACSet Family

| Skill | Domain | Schema Objects | Source |
|-------|--------|----------------|--------|
| [chatgpt-export-acset](file:///Users/bob/.claude/skills/chatgpt-export-acset/SKILL.md) | ChatGPT exports | Conversation, Message, Author | JSON export |
| [openai-acset](file:///Users/bob/.claude/skills/openai-acset/SKILL.md) | OpenAI API | Request, Response, Token | API calls |
| [browser-history-acset](file:///Users/bob/.claude/skills/browser-history-acset/SKILL.md) | Browser history | Visit, Page, Domain | SQLite |

### C. Infrastructure ACSet Family

| Skill | Domain | Schema Objects | Use Case |
|-------|--------|----------------|----------|
| [nix-acset-worlding](file:///Users/bob/.claude/skills/nix-acset-worlding/SKILL.md) | Nix store | Derivation, StorePath, Dep | GC analysis |
| [protocol-acset](file:///Users/bob/.claude/skills/protocol-acset/SKILL.md) | P2P protocols | Message, Peer, Channel | Interop design |
| [rg-flow-acset](file:///Users/bob/.claude/skills/rg-flow-acset/SKILL.md) | Ripgrep flows | Match, File, Pattern | Code search |

### D. Language Bridge ACSet Family

| Skill | Domain | Schema Objects | Bridge |
|-------|--------|----------------|--------|
| [lispsyntax-acset](file:///Users/bob/.claude/skills/lispsyntax-acset/SKILL.md) | S-expressions | Sexp, Atom, List | Julia ↔ Lisp |
| [compositional-acset-comparison](file:///Users/bob/.claude/skills/compositional-acset-comparison/SKILL.md) | Algorithm analysis | Query, Result, Schema | DuckDB ↔ LanceDB |

---

## III. SEMANTICALLY SIMILAR CATEGORICAL (12 skills)

Skills that share categorical/relational structure but don't explicitly use ACSets:

### A. Sheaf/Cohomology Family

| Skill | Structure | Relation to ACSet | Morphism |
|-------|-----------|-------------------|----------|
| [sheaf-cohomology](file:///Users/bob/.claude/skills/sheaf-cohomology/SKILL.md) | Čech cohomology | Sheaves on schemas | H^n(C, F) |
| [structured-decomp](file:///Users/bob/.claude/skills/structured-decomp/SKILL.md) | Tree decompositions | Sheaves on trees | FPT algorithms |
| [persistent-homology](file:///Users/bob/.claude/skills/persistent-homology/SKILL.md) | Persistence modules | Filtered ACSets | Barcodes |

**Morphism**: ACSet schema C → Sheaf F: C^op → Set

### B. Fibration/Extension Family

| Skill | Structure | Relation to ACSet | Morphism |
|-------|-----------|-------------------|----------|
| [covariant-fibrations](file:///Users/bob/.claude/skills/covariant-fibrations/SKILL.md) | Dependent types over 2 | Fibered ACSets | π: E → B |
| [kan-extensions](file:///Users/bob/.claude/skills/kan-extensions/SKILL.md) | (L ⊣ R) adjunctions | Schema change functors | Lan_F, Ran_F |
| [ordered-locale](file:///Users/bob/.claude/skills/ordered-locale/SKILL.md) | Frames with preorder | Directed ACSets | ≪ relation |

**Morphism**: Schema morphism F: C → D induces Lan_F: Set^C → Set^D

### C. Computational/Rewriting Family

| Skill | Structure | Relation to ACSet | Morphism |
|-------|-----------|-------------------|----------|
| [datalog-fixpoint](file:///Users/bob/.claude/skills/datalog-fixpoint/SKILL.md) | Bottom-up iteration | Relational queries | IDB/EDB |
| [interaction-nets](file:///Users/bob/.claude/skills/interaction-nets/SKILL.md) | Lafont nets | Graph rewriting | DPO rules |
| [open-games](file:///Users/bob/.claude/skills/open-games/SKILL.md) | Compositional games | Wiring diagrams | Play/Coplay |

**Morphism**: ACSet rewriting via DPO (Double Pushout)

### D. Distributed/Temporal Family

| Skill | Structure | Relation to ACSet | Morphism |
|-------|-----------|-------------------|----------|
| [topos-catcolab](file:///Users/bob/.claude/skills/topos-catcolab/SKILL.md) | Double theories | Collaborative ACSets | Automerge CRDT |
| [crdt-vterm](file:///Users/bob/.claude/skills/crdt-vterm/SKILL.md) | LWW/G-Counter | Mergeable ACSets | ⊔ lattice |
| [temporal-coalgebra](file:///Users/bob/.claude/skills/temporal-coalgebra/SKILL.md) | Final coalgebras | Stream of ACSets | νF bisimulation |

**Morphism**: ACSet × Time → Temporal ACSet (presheaf on ℕ^op)

---

## IV. MORPHISM DIAGRAM

```
                         ┌────────────────┐
                         │  acsets (0)    │
                         │  Core Functor  │
                         │   X: C → Set   │
                         └───────┬────────┘
                                 │
            ┌────────────────────┼────────────────────┐
            │                    │                    │
    ┌───────▼────────┐   ┌───────▼────────┐   ┌───────▼────────┐
    │ Domain-Specific│   │ Sheaf/Cohom    │   │ Computation    │
    │ (Workspace,    │   │ (structure on  │   │ (rewriting,    │
    │  Protocol)     │   │  schema)       │   │  queries)      │
    └───────┬────────┘   └───────┬────────┘   └───────┬────────┘
            │                    │                    │
            │    forgetful       │    forgetful       │
            │    functor U       │    functor U       │
            │                    │                    │
            └────────────────────┼────────────────────┘
                                 │
                         ┌───────▼────────┐
                         │  specter-acset │
                         │  Navigation    │
                         │  Lenses/Prisms │
                         └────────────────┘
```

---

## V. GF(3) TRIADIC ORGANIZATION

### Core Triad
```
acsets-relational-thinking (-1) ⊗ acsets (0) ⊗ specter-acset (+1) = 0 ✓
```

### Sheaf Triad
```
sheaf-cohomology (-1) ⊗ structured-decomp (0) ⊗ persistent-homology (+1) = 0 ✓
```

### Workspace Triad
```
calendar-acset (-1) ⊗ docs-acset (0) ⊗ drive-acset (+1) = 0 ✓
```

### Computation Triad
```
datalog-fixpoint (-1) ⊗ interaction-nets (0) ⊗ open-games (+1) = 0 ✓
```

### Temporal Triad
```
temporal-coalgebra (-1) ⊗ crdt-vterm (0) ⊗ topos-catcolab (+1) = 0 ✓
```

---

## VI. SCHEMA PATTERNS

### Generic ACSet Schema Template
```julia
@present SchDomainACSet(FreeSchema) begin
    # Objects (Ob)
    Entity::Ob
    Relation::Ob
    Attribute::Ob
    
    # Morphisms (Hom)
    source::Hom(Relation, Entity)
    target::Hom(Relation, Entity)
    
    # Attributes (Attr)
    Name::AttrType
    Value::AttrType
    entity_name::Attr(Entity, Name)
    attr_value::Attr(Attribute, Value)
end
```

### Common Schema Transformations

| Transform | From | To | Functor |
|-----------|------|----|---------|
| Forget | WorkspaceACSet | ACSet | Forgetful U |
| Embed | ACSet | SheafACSet | Yoneda y |
| Temporal | ACSet | StreamACSet | Δ^* (nerve) |
| Distributed | ACSet | CRDTACSet | Free(Lattice) |

---

## VII. USAGE PATTERNS

### Cross-ACSet Query
```julia
# Query across multiple domain ACSets
using ACSets, Catlab

# Define morphism between schemas
F = SchemaMap(CalendarSchema, TasksSchema, 
    Event => Task,
    due_date => due_date
)

# Pull back tasks to calendar view
calendar_tasks = Δ(F, my_tasks)
```

### Sheaf Consistency Check
```julia
# Verify local-to-global consistency
using StructuredDecompositions

decomp = tree_decomposition(my_acset)
consistent = verify_sheaf_condition(decomp)
```

### Temporal Stream
```julia
# Create temporal ACSet stream
using TemporalCoalgebra

stream = unfold(initial_acset) do x
    (observe(x), step(x))
end
```

---

## VIII. INVARIANTS

```yaml
invariants:
  - name: functor_preservation
    predicate: "F(id_A) = id_{F(A)} and F(g∘f) = F(g)∘F(f)"
    scope: per_morphism
    
  - name: schema_pullback
    predicate: "Δ(F, X) is right adjoint to Σ(F, X)"
    scope: per_schema_map
    
  - name: gf3_triad_conservation
    predicate: "Σ trit = 0 for each triad"
    scope: per_skill_group
    
  - name: crdt_convergence
    predicate: "a ⊔ b = b ⊔ a (commutativity)"
    scope: per_distributed_acset
```

---

## IX. REFERENCES

- [Catlab.jl](https://github.com/AlgebraicJulia/Catlab.jl)
- [ACSets.jl](https://github.com/AlgebraicJulia/ACSets.jl)
- [StructuredDecompositions.jl](https://github.com/AlgebraicJulia/StructuredDecompositions.jl)
- [Relational Thinking Book](https://www.relationalthinking.dev/)
- [CatColab](https://catcolab.org/)


---

## Autopoietic Marginalia

> **The interaction IS the skill improving itself.**

Every use of this skill is an opportunity for worlding:
- **MEMORY** (-1): Record what was learned
- **REMEMBERING** (0): Connect patterns to other skills  
- **WORLDING** (+1): Evolve the skill based on use



*Add Interaction Exemplars here as the skill is used.*
