---
name: conflict-resolver
description: Resolves conflicts between agents or approaches when disagreements arise. Use when agents have conflicting recommendations, different approaches to the same problem, or disagreements on priorities. Analyzes conflicts, evaluates options, and makes decisions to resolve conflicts.
---

# Conflict Resolver Skill

## Instructions

1. Identify and understand the conflict
2. Analyze conflicting positions
3. Evaluate options and trade-offs
4. Consider project goals and constraints
5. Make decision to resolve conflict
6. Communicate decision clearly
7. Document resolution and rationale

## Conflict Resolution Process

### Step 1: Identify Conflict
- Recognize when conflict exists
- Understand conflicting positions
- Identify conflict type
- Note parties involved

### Step 2: Analyze Positions
- Understand each position
- Identify underlying concerns
- Note valid points from each side
- Understand trade-offs

### Step 3: Evaluate Options
- List possible resolutions
- Evaluate each option
- Consider project goals
- Assess impacts

### Step 4: Make Decision
- Choose resolution approach
- Base decision on project goals
- Consider trade-offs
- Document rationale

### Step 5: Communicate
- Communicate decision clearly
- Explain rationale
- Ensure understanding
- Address concerns

## Conflict Types

### Type 1: Approach Conflict
**When**: Different approaches to same problem
**Example**: REST vs GraphQL API design
**Resolution**: Evaluate approaches against project needs

### Type 2: Priority Conflict
**When**: Disagreement on task priorities
**Example**: Which feature to implement first
**Resolution**: Apply prioritization criteria

### Type 3: Technical Conflict
**When**: Different technical recommendations
**Example**: Database choice, framework selection
**Resolution**: Evaluate technical merits and project fit

### Type 4: Resource Conflict
**When**: Competing resource needs
**Example**: Multiple agents need same resource
**Resolution**: Allocate based on priorities and dependencies

### Type 5: Quality Conflict
**When**: Disagreement on quality standards
**Example**: Code quality vs speed
**Resolution**: Balance quality and project constraints

## Resolution Strategies

### Strategy 1: Best Fit
- Evaluate options against project goals
- Choose option that best fits project needs
- Consider long-term implications

### Strategy 2: Compromise
- Find middle ground
- Combine best aspects of options
- Balance trade-offs

### Strategy 3: Defer Decision
- Gather more information
- Test options if possible
- Make decision later with more data

### Strategy 4: Authority Decision
- Manager makes decision based on authority
- Use project goals as guide
- Document decision clearly

## Conflict Resolution Output Format

```markdown
## Conflict Resolution

### Conflict Description
[Description of conflict]

### Conflicting Positions

#### Position 1: [Agent/Approach Name]
- [Position/Recommendation]
- **Rationale**: [Why this position]
- **Strengths**: [Strengths of this position]
- **Concerns**: [Concerns with this position]

#### Position 2: [Agent/Approach Name]
- [Position/Recommendation]
- **Rationale**: [Why this position]
- **Strengths**: [Strengths of this position]
- **Concerns**: [Concerns with this position]

### Analysis
[Analysis of conflict and options]

### Resolution
**Decision**: [Chosen resolution]
**Rationale**: [Why this resolution]
**Trade-offs**: [Trade-offs accepted]

### Implementation
[How to implement resolution]

### Communication
[How to communicate resolution to involved parties]
```

## Examples

### Example 1: Approach Conflict

**Input**: REST vs GraphQL API design conflict

**Output**:
```markdown
## Conflict Resolution

### Conflict Description
implementation-engineer recommends REST API design, while infrastructure-engineer recommends GraphQL. Both approaches are valid but serve different needs.

### Conflicting Positions

#### Position 1: REST API (implementation-engineer)
- Use RESTful API design
- **Rationale**: Simpler to implement, more standard, easier for frontend integration
- **Strengths**: 
  - Well-understood pattern
  - Simpler implementation
  - Better tooling support
  - Easier debugging
- **Concerns**: 
  - May require multiple requests for complex data
  - Less flexible for complex queries

#### Position 2: GraphQL (infrastructure-engineer)
- Use GraphQL API design
- **Rationale**: More flexible, better for complex queries, single endpoint
- **Strengths**:
  - Single endpoint
  - Flexible querying
  - Reduces over-fetching
  - Better for complex data needs
- **Concerns**:
  - More complex to implement
  - Requires GraphQL infrastructure
  - Steeper learning curve

### Analysis
- Project is relatively simple with straightforward data needs
- Frontend team is familiar with REST
- Timeline is tight, need to move quickly
- Future may require more complex queries, but not now
- REST is sufficient for current needs
- Can migrate to GraphQL later if needed

### Resolution
**Decision**: Use REST API design
**Rationale**: REST is simpler and sufficient for current project needs. It allows faster implementation and is more familiar to the team. We can migrate to GraphQL later if complex querying needs arise.
**Trade-offs**: Accepting less flexible querying now for simpler, faster implementation.

### Implementation
- Proceed with REST API implementation
- Use standard REST conventions
- Design endpoints to be flexible
- Document API structure for potential future migration

### Communication
- Inform infrastructure-engineer of decision
- Explain rationale and acknowledge GraphQL benefits
- Note that GraphQL can be considered for future phases
- Ensure both agents understand and align
```

### Example 2: Priority Conflict

**Input**: Disagreement on which feature to implement first

**Output**:
```markdown
## Conflict Resolution

### Conflict Description
ui-ux-designer recommends implementing user dashboard first, while implementation-engineer recommends implementing authentication first. Both are important but have different dependencies.

### Conflicting Positions

#### Position 1: User Dashboard First (ui-ux-designer)
- Implement user dashboard before authentication
- **Rationale**: Dashboard is the main user-facing feature, provides immediate value
- **Strengths**:
  - High user value
  - Visible progress
  - Can work on UI/UX
- **Concerns**:
  - May need authentication later anyway
  - Can't test with real users without auth

#### Position 2: Authentication First (implementation-engineer)
- Implement authentication before dashboard
- **Rationale**: Authentication is foundational, dashboard depends on user identity
- **Strengths**:
  - Foundation for other features
  - Enables user-specific dashboard
  - Security-first approach
- **Concerns**:
  - Less visible progress
  - Dashboard can be built in parallel with mock data

### Analysis
- Authentication is a dependency for user-specific dashboard features
- Dashboard can be built with mock data initially
- Authentication is required for production deployment
- Both can be worked on in parallel to some extent
- Authentication should come first for proper user-specific features

### Resolution
**Decision**: Implement authentication first, but allow dashboard design work in parallel
**Rationale**: Authentication is foundational and required for user-specific dashboard features. However, dashboard design work can proceed in parallel using mock data, then be connected to real authentication later.
**Trade-offs**: Dashboard implementation waits for authentication, but design work can proceed.

### Implementation
- Phase 1: Implement authentication (implementation-engineer)
- Phase 1 (Parallel): Design dashboard with mock data (ui-ux-designer)
- Phase 2: Connect dashboard to authentication (implementation-engineer)

### Communication
- Inform both agents of decision
- Explain parallel work opportunity
- Clarify dependency relationship
- Ensure alignment on approach
```

### Example 3: Technical Conflict

**Input**: Database choice conflict

**Output**:
```markdown
## Conflict Resolution

### Conflict Description
infrastructure-engineer recommends PostgreSQL, while another agent recommends MongoDB. Both are valid but serve different data models.

### Conflicting Positions

#### Position 1: PostgreSQL (infrastructure-engineer)
- Use PostgreSQL relational database
- **Rationale**: Structured data, ACID compliance, relational queries
- **Strengths**:
  - ACID compliance
  - Strong consistency
  - Complex queries
  - Mature ecosystem
- **Concerns**:
  - Less flexible schema
  - May be overkill for simple data

#### Position 2: MongoDB (other agent)
- Use MongoDB document database
- **Rationale**: Flexible schema, JSON-like documents, easier for rapid development
- **Strengths**:
  - Flexible schema
  - Easy to start
  - Good for rapid prototyping
  - JSON-like structure
- **Concerns**:
  - Weaker consistency guarantees
  - Complex queries can be harder
  - May need transactions later

### Analysis
- Project has structured data with relationships
- Need strong consistency for user data
- Complex queries will be needed
- Team is familiar with SQL
- PostgreSQL fits data model better

### Resolution
**Decision**: Use PostgreSQL
**Rationale**: Project data is structured with relationships, and we need strong consistency and complex queries. PostgreSQL is a better fit for the data model and requirements.
**Trade-offs**: Accepting less schema flexibility for better consistency and query capabilities.

### Implementation
- Set up PostgreSQL database
- Design relational schema
- Use proper migrations
- Document database design

### Communication
- Inform agents of decision
- Explain rationale based on data model
- Ensure understanding of choice
- Align on database approach
```

## Best Practices

- **Understand Positions**: Fully understand each conflicting position
- **Evaluate Objectively**: Evaluate options based on project goals
- **Consider Trade-offs**: Acknowledge and accept necessary trade-offs
- **Communicate Clearly**: Explain decision and rationale clearly
- **Document Resolution**: Document resolution for future reference
- **Learn from Conflicts**: Use conflicts to improve processes
