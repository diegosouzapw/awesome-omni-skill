---
name: agent-extend
description: Extend the Strategy Coach agent with new capabilities. Use when user says "add coaching phase", "new research pillar", "extend agent", "add strategic output", or asks to enhance the AI coaching methodology.
---

# Strategy Coach Agent Extension Skill

Extends the Strategy Coach agent following the Product Strategy Research Playbook methodology.

## When to Use

Activate when user requests:
- "add coaching phase"
- "new research pillar"
- "extend agent"
- "add strategic output"
- Enhancing AI coaching methodology

## Agent Location

All agent files are in `src/lib/agents/strategy-coach/`:

| File | Purpose |
|------|---------|
| `index.ts` | Main agent interface, Claude API integration |
| `framework-state.ts` | State tracking, progress calculations |
| `system-prompt.ts` | Methodology documentation, coaching tone |
| `client-context.ts` | Organization context loading |

## Current Methodology

### Four Coaching Phases

1. **Discovery** - Understanding context and goals
2. **Research** - Exploring the three pillars
3. **Synthesis** - Developing strategic hypotheses
4. **Planning** - Creating actionable outputs

### Three Research Pillars

1. **Macro Market** - Industry trends, competition, technology
2. **Customer** - Segments, JTBD, pain points
3. **Colleague** - Internal capabilities, constraints

### Strategic Bets Format

```
We believe [trend/customer need]
Which means [opportunity/problem space]
So we will explore [hypothesis/initiative direction]
Success looks like [leading indicator metric]
```

## Extension Patterns

### Adding a New Phase

1. Update `framework-state.ts`:

```typescript
export interface FrameworkState {
  phase: 'discovery' | 'research' | 'synthesis' | 'planning' | 'new_phase';
  // ...
}

export function calculateProgress(state: FrameworkState): number {
  // Add new phase to progress calculation
}
```

2. Update `system-prompt.ts`:

```typescript
const NEW_PHASE_GUIDANCE = `
## New Phase: {Phase Name}

### Purpose
{What this phase accomplishes}

### Key Activities
- {Activity 1}
- {Activity 2}

### Transition Criteria
Move to next phase when:
- {Criterion 1}
- {Criterion 2}
`;
```

### Adding a Research Pillar

1. Update `framework-state.ts`:

```typescript
export interface ResearchProgress {
  macroMarket: PillarProgress;
  customer: PillarProgress;
  colleague: PillarProgress;
  newPillar: PillarProgress;  // Add new pillar
}

interface PillarProgress {
  started: boolean;
  completed: boolean;
  insights: string[];
}
```

2. Update `system-prompt.ts`:

```typescript
const NEW_PILLAR_SECTION = `
### Pillar 4: {Pillar Name}

#### Core Questions
- {Question 1}
- {Question 2}

#### Key Areas to Explore
| Area | Focus |
|------|-------|
| {Area 1} | {Focus description} |
| {Area 2} | {Focus description} |

#### Insight Format
Capture insights as:
- **Observation**: What was discovered
- **Implication**: What it means for strategy
- **Evidence**: Supporting data points
`;
```

### Adding Strategic Output Types

1. Update `framework-state.ts`:

```typescript
export interface CanvasCompletion {
  marketReality: boolean;
  customerInsights: boolean;
  organizationalContext: boolean;
  strategicSynthesis: boolean;
  strategicContext: boolean;
  newOutputType: boolean;  // Add new output
}
```

2. Update `system-prompt.ts`:

```typescript
const NEW_OUTPUT_TEMPLATE = `
## {Output Name} Template

### Purpose
{What this output provides}

### Structure
1. **Section 1**: {Description}
2. **Section 2**: {Description}

### Example
{Concrete example of the output}
`;
```

## Coaching Tone Guidelines

When extending the agent, maintain:

- **Socratic questioning** - Guide through questions, don't prescribe
- **Empathetic acknowledgment** - Validate challenges
- **Structured exploration** - Follow methodology phases
- **Actionable synthesis** - End with clear next steps

Example prompt patterns:

```typescript
const QUESTIONING_PATTERNS = [
  "What patterns are you seeing in...",
  "How might this affect...",
  "What would need to be true for...",
  "Who else in your organization might have insight on...",
];
```

## State Persistence

State is stored in `framework_state` JSONB column in `conversations` table.

Update state through the agent:

```typescript
export function updateFrameworkState(
  currentState: FrameworkState,
  update: Partial<FrameworkState>
): FrameworkState {
  return {
    ...currentState,
    ...update,
    lastUpdated: new Date().toISOString(),
  };
}
```

## Testing Extensions

Add tests in `tests/unit/lib/agents/strategy-coach/`:

```typescript
describe('New Phase', () => {
  it('should calculate progress correctly', () => {
    const state = createStateWithNewPhase();
    expect(calculateProgress(state)).toBe(expectedProgress);
  });

  it('should transition from previous phase', () => {
    // Test phase transition logic
  });
});
```
