---
name: self-improving-ai
description: Understanding and using StickerNest's self-improving AI system. Use when the user asks about AI self-improvement, prompt versioning, reflection loops, AI evaluation, auto-tuning prompts, or the AI judge system. Covers AIReflectionService, stores, and the improvement loop.
---

# Self-Improving AI System for StickerNest

This skill covers StickerNest's self-improving AI system - an AI that evaluates its own generations and automatically improves its prompts over time.

## When to Use This Skill

This skill helps when you need to:
- Understand how the self-improvement loop works
- Configure the reflection system settings
- Add new AI capabilities that should self-improve
- Debug or tune the evaluation rubrics
- Extend the improvement loop to new domains

## Core Concepts

### The Improvement Loop

The self-improving AI follows this cycle:

```
[Generation] → [Track Metrics] → [Evaluate] → [Analyze] → [Improve Prompt] → [Generation]
     ↓              ↓                ↓            ↓              ↓
  Widget/Image   MetricsStore   AIReflection   Suggestions   PromptVersion
                               Service (Judge)               Store
```

### Key Components

| Component | Purpose | Location |
|-----------|---------|----------|
| `AIReflectionStore` | Stores evaluations, runs, suggestions | `src/state/useAIReflectionStore.ts` |
| `PromptVersionStore` | Version control for AI prompts | `src/state/usePromptVersionStore.ts` |
| `GenerationMetricsStore` | Tracks generation quality | `src/state/useGenerationMetricsStore.ts` |
| `AIReflectionService` | The "judge" AI that evaluates | `src/ai/AIReflectionService.ts` |
| `SkillRecommendationService` | Suggests new skills | `src/ai/SkillRecommendationService.ts` |
| `ReflectionDashboard` | Admin UI panel | `src/components/ai-reflection/ReflectionDashboard.tsx` |

### Evaluation Rubrics

The system evaluates generations against rubrics with weighted criteria:

**Widget Generation Rubric:**
- Protocol Compliance (25%) - Follows Widget Protocol v3.0
- Code Quality (20%) - Clean, readable code
- Functionality (25%) - Works correctly
- Port Design (15%) - Good input/output definitions
- User Experience (15%) - Visual design and interaction

**Image Generation Rubric:**
- Prompt Accuracy (30%) - Matches user intent
- Visual Quality (25%) - Clear, well-composed
- Style Consistency (20%) - Matches requested style
- Usability (25%) - Suitable for design use

## Step-by-Step Guide

### Step 1: Recording a Generation

When AI generates something, record it in the metrics store:

```typescript
import { useGenerationMetricsStore } from '../state/useGenerationMetricsStore';

// After generation completes
const metricsStore = useGenerationMetricsStore.getState();
const recordId = metricsStore.addRecord({
  type: 'widget', // or 'image', 'pipeline', 'skill'
  promptVersionId: currentPromptVersionId,
  userPrompt: userInput,
  result: success ? 'success' : 'failure',
  errorMessage: error?.message,
  qualityScore: validationScore, // 0-100 if available
  metadata: {
    model: 'claude-3-5-sonnet',
    provider: 'anthropic',
    durationMs: elapsed,
  },
});
```

### Step 2: Adding User Feedback

Capture user feedback on generations:

```typescript
// Thumbs up/down
metricsStore.addFeedback(recordId, 'thumbs_up');

// Star rating
metricsStore.addFeedback(recordId, 'rating', 4);

// With comment and tags
metricsStore.addFeedback(recordId, 'rating', 2, 'Output was too verbose', ['too_long', 'verbose']);
```

### Step 3: Running a Reflection

Trigger a reflection manually or let it run on schedule:

```typescript
import { reflectOnWidgetGeneration } from '../ai/AIReflectionService';

// Manual reflection
const result = await reflectOnWidgetGeneration({ forceRun: true });

console.log('Evaluation passed:', result.evaluation?.passed);
console.log('Prompt changed:', result.promptChanged);
console.log('New suggestions:', result.suggestions.length);
```

### Step 4: Managing Prompt Versions

Handle prompt version control:

```typescript
import { usePromptVersionStore } from '../state/usePromptVersionStore';

const promptStore = usePromptVersionStore.getState();

// Get current prompt for a domain
const currentPrompt = promptStore.getActivePrompt('widget_generation');

// Create a new version
const versionId = promptStore.createVersion(
  'widget_generation',
  newPromptContent,
  'Improved based on reflection',
  'ai', // created by AI
  evaluationId
);

// Revert to previous version
promptStore.revertToVersion(previousVersionId);

// Handle pending proposals
const proposals = promptStore.getPendingProposals('widget_generation');
proposals.forEach(p => {
  // Review and approve/reject
  promptStore.approveProposal(p.id);
  // or promptStore.rejectProposal(p.id);
});
```

### Step 5: Configuring the Reflection Loop

Adjust reflection settings:

```typescript
import { useAIReflectionStore } from '../state/useAIReflectionStore';

const reflectionStore = useAIReflectionStore.getState();

reflectionStore.updateConfig({
  enabled: true,
  intervalMinutes: 60,        // How often to reflect
  messagesToEvaluate: 20,     // How many records to evaluate
  scoreThreshold: 3.5,        // Pass/fail threshold (1-5)
  cooldownMinutes: 30,        // Pause after prompt update
  autoApplyChanges: false,    // Require approval for changes
  evaluateUnevaluatedOnly: true,
});
```

## Code Examples

### Example: Custom Rubric for New Domain

```typescript
import { useAIReflectionStore, type RubricCriteria } from '../state/useAIReflectionStore';

const customRubric: RubricCriteria[] = [
  {
    name: 'Accuracy',
    description: 'Output matches expected format and content',
    weight: 0.4,
    minScore: 1,
    maxScore: 5,
  },
  {
    name: 'Efficiency',
    description: 'Uses optimal approach without waste',
    weight: 0.3,
    minScore: 1,
    maxScore: 5,
  },
  {
    name: 'Maintainability',
    description: 'Easy to understand and modify',
    weight: 0.3,
    minScore: 1,
    maxScore: 5,
  },
];

const reflectionStore = useAIReflectionStore.getState();
reflectionStore.setWidgetRubric(customRubric);
```

### Example: Tracking Skill Gaps

```typescript
import { analyzeSkillGaps, generateSkillFromGap } from '../ai/SkillRecommendationService';

// Analyze patterns for potential new skills
const gaps = analyzeSkillGaps();

// Find high-priority gaps
const criticalGaps = gaps.filter(g => g.priority === 'critical' || g.priority === 'high');

// Generate a skill template for a gap
if (criticalGaps.length > 0) {
  const template = generateSkillFromGap(criticalGaps[0].id);
  console.log('Suggested skill:', template?.name);
  console.log('Content:', template?.content);
}
```

### Example: Using the Reflection Dashboard

```tsx
import { ReflectionDashboard } from '../components/ai-reflection';
import { useState } from 'react';

function MyComponent() {
  const [showDashboard, setShowDashboard] = useState(false);

  return (
    <>
      <button onClick={() => setShowDashboard(true)}>
        Open AI Dashboard
      </button>
      <ReflectionDashboard
        isOpen={showDashboard}
        onClose={() => setShowDashboard(false)}
      />
    </>
  );
}
```

## Common Patterns

### Pattern: Adding Self-Improvement to a New AI Feature

1. Add a prompt domain to `PromptVersionStore`
2. Track generations in `GenerationMetricsStore`
3. Create a rubric for evaluation
4. Add reflection trigger to `AIReflectionService`

### Pattern: Manual Prompt Improvement

When you want to update a prompt based on observations:

```typescript
const promptStore = usePromptVersionStore.getState();

// Create proposal for review
promptStore.createProposal(
  'widget_generation',
  improvedPromptContent,
  'User requested more concise outputs',
  ['User feedback: too verbose', 'Multiple complaints about length'],
  'manual-review'
);
```

### Pattern: Exporting Data for Analysis

```typescript
const metricsStore = useGenerationMetricsStore.getState();

const analysisData = metricsStore.exportForReflection('widget', {
  limit: 100,
  includeFailuresOnly: true,
});

console.log('Failure rate:', 100 - analysisData.metrics.successRate);
console.log('Common issues:', analysisData.metrics.commonIssues);
```

## Reference Files

| Category | File |
|----------|------|
| **Reflection Store** | `src/state/useAIReflectionStore.ts` |
| **Prompt Versions** | `src/state/usePromptVersionStore.ts` |
| **Generation Metrics** | `src/state/useGenerationMetricsStore.ts` |
| **Reflection Service** | `src/ai/AIReflectionService.ts` |
| **Skill Recommendations** | `src/ai/SkillRecommendationService.ts` |
| **Dashboard UI** | `src/components/ai-reflection/ReflectionDashboard.tsx` |

## Troubleshooting

### Issue: Reflection loop not running
**Cause**: Cooldown period active or no unevaluated records
**Fix**: Check `isInCooldown()` and `getUnevaluatedRecords()`. Use `forceRun: true` to bypass cooldown.

### Issue: Prompts changing too frequently
**Cause**: Score threshold too high or auto-apply enabled
**Fix**: Lower `scoreThreshold`, disable `autoApplyChanges`, increase `cooldownMinutes`

### Issue: AI judge too lenient
**Cause**: Rubric weights favor passing, or system prompt too forgiving
**Fix**: Modify rubric weights, update the `reflection_judge` prompt in `PromptVersionStore`

### Issue: Missing evaluations
**Cause**: Generations not being recorded to metrics store
**Fix**: Ensure `addRecord()` is called after every generation with proper metadata
