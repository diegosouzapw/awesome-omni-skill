---
name: state
description: Create representation of current world state for a domain. Use when modeling system state, building world models, capturing entity relationships, or establishing baseline snapshots.
argument-hint: "[scope] [schema] [timestamp]"
disable-model-invocation: false
user-invocable: true
allowed-tools: Read, Grep
context: fork
agent: explore
layer: MODEL
---

## Intent

Create a structured representation of the current state of a domain, system, or entity. This is the foundation for world modeling, enabling tracking of entities, relationships, and properties over time.

**Success criteria:**
- State captured in structured, queryable format
- Entities and relationships clearly identified
- Uncertainty and confidence explicitly represented
- Evidence anchors for all state assertions

**Compatible schemas:**
- `schemas/output_schema.yaml`
- `reference/world_state_schema.yaml`

## Inputs

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `scope` | Yes | string | What domain/system to model (e.g., "user authentication", "payment processing") |
| `schema` | No | object | State schema defining expected structure |
| `timestamp` | No | string | Point in time for state (default: now) |
| `depth` | No | string | Modeling depth: surface, detailed, comprehensive |

## Procedure

1) **Define scope boundaries**: Clarify what is included in the state model
   - Identify system or domain boundaries
   - Determine entity types to include
   - Establish relationship types to capture

2) **Identify entities**: Enumerate entities within scope
   - Extract entity references from sources
   - Assign unique identifiers
   - Capture entity properties/attributes

3) **Map relationships**: Document connections between entities
   - Identify relationship types (contains, depends_on, references)
   - Note relationship directionality
   - Capture relationship properties

4) **Capture current values**: Record current state of each entity
   - Document property values with evidence
   - Note uncertainty where values are unclear
   - Record observation timestamp

5) **Apply schema**: Structure state according to schema if provided
   - Validate against schema constraints
   - Note schema violations or extensions needed
   - Ensure required fields are populated

6) **Ground state claims**: Attach evidence to all assertions
   - Reference source files, commands, or observations
   - Note confidence level for each assertion

## Output Contract

Return a structured object:

```yaml
state:
  world_id: string  # Unique identifier for this state snapshot
  scope: string  # Domain being modeled
  timestamp: string  # When state was captured
  entities:
    - id: string  # Unique entity identifier
      type: string  # Entity classification
      properties: object  # Entity attributes
      confidence: number  # Confidence in entity state
  relationships:
    - from: string  # Source entity ID
      to: string  # Target entity ID
      type: string  # Relationship type
      properties: object  # Relationship attributes
  meta:
    version: string  # State schema version
    completeness: number  # 0-1 estimate of coverage
confidence: 0..1  # Overall state model confidence
evidence_anchors: ["file:line", "observation:id"]
assumptions: []
```

### Field Definitions

| Field | Type | Description |
|-------|------|-------------|
| `state.world_id` | string | Unique identifier for state snapshot |
| `state.scope` | string | Domain or system being modeled |
| `state.entities` | array | Identified entities with properties |
| `state.relationships` | array | Connections between entities |
| `state.meta.completeness` | number | Estimated model completeness |
| `confidence` | number | Overall confidence in state accuracy |

## Examples

### Example 1: Model Authentication System State

**Input:**
```yaml
scope: "user authentication system"
depth: "detailed"
```

**Output:**
```yaml
state:
  world_id: "auth-state-20240115-001"
  scope: "user authentication system"
  timestamp: "2024-01-15T10:30:00Z"
  entities:
    - id: "auth-service"
      type: "service"
      properties:
        status: "running"
        version: "2.3.1"
        port: 8080
      confidence: 0.95
    - id: "user-db"
      type: "database"
      properties:
        type: "PostgreSQL"
        connection_pool: 20
        active_connections: 12
      confidence: 0.90
    - id: "session-cache"
      type: "cache"
      properties:
        type: "Redis"
        memory_used: "256MB"
        keys: 15420
      confidence: 0.90
  relationships:
    - from: "auth-service"
      to: "user-db"
      type: "depends_on"
      properties:
        connection_type: "direct"
        required: true
    - from: "auth-service"
      to: "session-cache"
      type: "depends_on"
      properties:
        connection_type: "direct"
        required: true
  meta:
    version: "1.0"
    completeness: 0.85
confidence: 0.88
evidence_anchors:
  - "config/services.yml:15-45"
  - "command:docker ps"
  - "command:redis-cli info"
assumptions:
  - "Service discovery reflects actual running state"
  - "Configuration matches deployed state"
```

### Example 2: Model Code Module State

**Input:**
```yaml
scope: "payment processing module"
schema:
  required_entities: ["class", "method", "dependency"]
```

**Output:**
```yaml
state:
  world_id: "payments-code-20240115"
  scope: "payment processing module"
  timestamp: "2024-01-15T11:00:00Z"
  entities:
    - id: "PaymentProcessor"
      type: "class"
      properties:
        file: "src/services/payment_processor.rb"
        lines: 145
        methods: 8
        complexity: 24
      confidence: 0.95
    - id: "process_payment"
      type: "method"
      properties:
        class: "PaymentProcessor"
        visibility: "public"
        params: ["order", "payment_method"]
        complexity: 12
      confidence: 0.95
    - id: "stripe-gem"
      type: "dependency"
      properties:
        name: "stripe"
        version: "8.0.0"
        usage: ["PaymentProcessor"]
      confidence: 0.90
  relationships:
    - from: "PaymentProcessor"
      to: "stripe-gem"
      type: "depends_on"
      properties:
        import_type: "require"
    - from: "process_payment"
      to: "PaymentProcessor"
      type: "member_of"
      properties:
        visibility: "public"
  meta:
    version: "1.0"
    completeness: 0.75
confidence: 0.85
evidence_anchors:
  - "src/services/payment_processor.rb:1-145"
  - "Gemfile:42"
assumptions:
  - "Static analysis reflects runtime behavior"
  - "No dynamic method definitions"
```

## Verification

- [ ] State includes world_id and timestamp
- [ ] All entities have unique IDs and types
- [ ] Relationships reference valid entity IDs
- [ ] Confidence scores present for entities
- [ ] Evidence anchors support state assertions

**Verification tools:** Read (to verify file references)

## Safety Constraints

- `mutation`: false
- `requires_checkpoint`: false
- `requires_approval`: false
- `risk`: low

**Capability-specific rules:**
- Do not modify state while modeling it
- Note when state may be stale or dynamic
- Flag entities with low confidence
- Do not invent entities without evidence

## Composition Patterns

**Commonly follows:**
- `observe` - Observations feed into state modeling
- `retrieve` - Retrieved data informs state
- `integrate` - Merged data forms state

**Commonly precedes:**
- `transition` - State enables transition modeling
- `compare` - States can be compared (diff)
- `simulate` - State is starting point for simulation

**Anti-patterns:**
- Never use state for predictions (use `predict`)
- Avoid state for single-value measurement (use `measure`)

**Workflow references:**
- See `reference/workflow_catalog.yaml#world_model_build` for state in world modeling
- See `reference/workflow_catalog.yaml#digital_twin_sync_loop` for state in digital twins
