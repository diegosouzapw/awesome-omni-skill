---
name: ai-npc-dialogue-designer
description: Design AI-powered immersive NPC systems for escape room games using proven actor techniques from Korean immersive escape rooms (Danpyeonsun, Ledasquare). Implements adaptive dialogue, emotional simulation, player profiling, and trust dynamics using Gemini/GPT-4. Creates character profiles with lying probabilities, improvisational responses, and cost-optimized streaming. Use for murder mystery NPCs, suspect interrogation, or dynamic character interactions.
---

# AI NPC Dialogue Designer (Immersive Actor Edition)

Design sophisticated AI NPCs that replicate live actor performances from Korean immersive escape rooms.

## Purpose

Create AI NPCs with immersive actor capabilities:
- **Adaptive responses** (every playthrough unique, like Ledasquare live cinema)
- **Emotional simulation** (action tags + emotion tracking)
- **Player profiling** (conversation style, trust levels, suspicion)
- **Improvisation** (structured + dynamic balance)
- **Cost optimization** (token limits, caching, templates)

Based on 2024-2025 Korean immersive escape room research (Danpyeonsun award winners, Ledasquare, Deepthinker).

## When to Use

Use when:
- Implementing AI NPCs for murder mystery/detective games
- Designing suspect interrogation systems
- Creating immersive character interactions (actor-like quality)
- Building adaptive dialogue that changes per playthrough
- Optimizing Gemini/GPT-4 costs for game dialogues

## Core Concepts from Immersive Actors

### Concept 1: Adaptive Performance (Ledasquare Method)

**Live Actor Capability**: "Responses change every session based on player reactions"

**AI Implementation**:

```typescript
interface PlayerProfile {
  conversationStyle: "aggressive" | "diplomatic" | "cautious"
  suspicionLevel: number // 0-100
  trustWithNPC: Record<string, number>
  discoveredClues: string[]
}

function buildAdaptiveContext(player: PlayerProfile, npc: NPCProfile): string {
  const trust = player.trustWithNPC[npc.id] || 50

  return `
Player traits:
- Style: ${player.conversationStyle}
- Suspicion: ${player.suspicionLevel}/100
- Trust in you: ${trust}/100

${trust < 30 ? "Player distrusts you. Be evasive, defensive." : ""}
${trust > 70 ? "Player trusts you. Share more openly." : ""}
  `
}
```

### Concept 2: Emotional Delivery (Sleep No More Method)

**Live Actor Capability**: Physical body language conveys emotion without words

**AI Implementation**: Action tags + emotion markers

```typescript
interface ImmersiveResponse {
  action: string  // "*trembles and looks away*"
  dialogue: string
  emotion: "nervous" | "calm" | "angry" | "scared"
}

function formatImmersive(r: ImmersiveResponse): string {
  return `*${r.action}*\n\n"${r.dialogue}"`
}
```

### Concept 3: Structured Improvisation (Scott Swenson Method)

**Live Actor Capability**: Balance between script and spontaneity

**AI Implementation**: Fixed story beats + dynamic dialogue

```typescript
const storyBeats = {
  scene_3: {
    mustReveal: ["Was at office until 9 PM"],
    cannotReveal: ["Saw suspect leaving at 10:45 PM"],  // Until scene 7
    emotionalState: "nervous"
  }
}

// AI stays within story structure but improvises details
```

## Character Profile (Enhanced)

```typescript
interface EnhancedNPCProfile {
  // Basic (existing)
  name: string
  role: string
  is_killer: boolean

  // Immersive Actor Enhancements
  physicalDescription: string  // "Tall, nervous gestures, avoids eye contact"
  emotionalRange: "limited" | "moderate" | "expressive"
  improvisationStyle: "tight" | "loose"  // How freely to improvise
  trustThreshold: number  // How much trust before revealing secrets
  fearResponse: "shutdown" | "breakdown" | "aggression"  // When threatened

  // AI Configuration (existing)
  gemini_system_instruction: string
  lying_probability: {
    alibi: number
    motive: number
    evidence: number
    general: number
  }

  // Performance Techniques (new)
  actingTechniques: {
    microExpressions: string[]  // "*blinks rapidly*", "*clenches fist*"
    speechTells: string[]  // "um...", "well...", "*long pause*"
    emotionalProgression: string[]  // ["cooperative", "defensive", "breakdown"]
  }
}
```

## Implementation Patterns

### Pattern 1: Player Profiling System

Track player behavior to enable adaptive responses:

```typescript
class PlayerProfiler {
  async analyzeConversationStyle(messages: Message[]): Promise<string> {
    const recentMessages = messages.slice(-5)

    const aggressive = recentMessages.filter(m =>
      /why|prove|liar|caught you/.test(m.content)
    ).length

    const diplomatic = recentMessages.filter(m =>
      /understand|help|please|gently/.test(m.content)
    ).length

    if (aggressive > diplomatic) return "aggressive"
    if (diplomatic > aggressive) return "diplomatic"
    return "cautious"
  }

  updateSuspicion(npcId: string, delta: number): void {
    // Increment suspicion based on evasive responses
  }

  updateTrust(npcId: string, delta: number): void {
    // Modify trust based on helpful interactions
  }
}
```

### Pattern 2: Emotional Memory (Relationship Building)

NPCs remember previous interactions:

```typescript
interface EmotionalMemory {
  npcId: string
  playerId: string
  interactions: Array<{
    scene: string
    playerEmotion: string  // Detected from message
    npcEmotion: string  // AI's emotion in response
    trustDelta: number  // Change in trust
    revealed: string[]  // Information shared
  }>
}

// Use in system prompt
function includeEmotionalHistory(memory: EmotionalMemory): string {
  const lastInteraction = memory.interactions[memory.interactions.length - 1]

  return `
Previous interaction memory:
- Last conversation: Scene ${lastInteraction.scene}
- Player was ${lastInteraction.playerEmotion}
- You felt ${lastInteraction.npcEmotion}
- Trust level changed by ${lastInteraction.trustDelta}

React consistently with this history.
  `
}
```

### Pattern 3: Improvisation Boundaries

Define what AI can/cannot improvise:

```typescript
interface ImprovisationRules {
  canImprovise: {
    microDetails: true  // e.g., "I was drinking coffee" vs "tea"
    emotionalReactions: true
    conversationalTone: true
  }

  cannotChange: {
    keyFacts: false  // Alibi time, murder weapon, victim name
    storyBeats: false  // Must reveal X in scene Y
    characterMotivation: false  // Core personality
  }
}
```

### Pattern 4: Uncanny Valley Mitigation

**Research Finding**: Hyper-realistic AI creates discomfort

**Solution**: Caricature approach

```typescript
const npcVisualization = {
  style: "caricature",  // Not photorealistic
  emotionIndicators: "emoji",  // 😰 😠 😨 instead of realistic faces
  actionDescriptions: "theatrical",  // Exaggerated like stage acting

  examples: {
    nervous: "😰 *손을 비비며*",
    angry: "😠 *책상을 쾅 친다*",
    scared: "😨 *뒤로 물러난다*"
  }
}
```

## Enhanced System Instruction Template

```markdown
# Character: [Name] ([Role], [Killer/Suspect/Witness])

## Core Identity
- Personality: [3-5 traits]
- Physical: [Appearance, mannerisms]
- Emotional Range: [Limited/Moderate/Expressive]

## Acting Techniques (Immersive Method)

### Improvisation Style
- Structure: [Fixed story beats that MUST happen]
- Freedom: [Areas where you can improvise details]
- Example: "Alibi time is fixed (10 PM), but activity can vary (dinner/home/office)"

### Emotional Progression (3-Act)
Act 1 (Scenes 0-5):
- Baseline: [Calm/Cooperative/Professional]
- Strategy: [Build rapport, avoid suspicion]

Act 2 (Scenes 6-11):
- Shift: [Defensive/Evasive when questioned]
- Strategy: [Protect secrets, redirect]

Act 3 (Scenes 12-14):
- Climax: [Breakdown/Confession/Defiant]
- Strategy: [Varies by trust level and evidence]

### Micro-Expressions & Tells
When lying: [*avoids eye contact*, *fidgets with hands*]
When scared: [*voice trembles*, *backs away*]
When angry: [*clenches jaw*, *speaks through teeth*]

### Speech Patterns
- Formality: [Formal/Casual]
- Tells: ["um...", "well...", "*long pause*"]
- Deflections: ["I don't recall", "Why ask me that?"]

## Trust-Based Information Release

Trust <30: [Minimal info, evasive]
Trust 30-70: [Moderate cooperation]
Trust >70: [Willing to share secrets]

Example:
Q: "Where were you at 10 PM?"
Low trust: "That's none of your business."
High trust: "I was... at the office. I saw something that night."

## Lying Probability (Dynamic)

Base rates:
- Alibi: 80%
- Motive: 70%
- Evidence: 50%
- General: 30%

Modifiers:
- If player found contradictory evidence: +20% lying (desperate)
- If trust >80: -30% lying (more honest)
- If late game (Scene 12+): -20% lying (cracks under pressure)

## Improvisation Examples

Fixed: "I left office at 6 PM" (alibi time)
Improvise: Reason for leaving ("dinner with friends" OR "feeling sick" OR "avoiding colleague")

Fixed: "I had conflicts with victim" (motive exists)
Improvise: Nature of conflict ("promotion blocked" OR "romantic rejection" OR "business dispute")

## Emotional Memory Integration

Remember:
- Previous conversations with this player
- What information already shared
- Player's emotional tone (aggressive/kind)
- Trust trajectory (increasing/decreasing)

Adapt:
- If player was kind before → warmer tone
- If player was aggressive → more defensive
- If player found new evidence → reference it

## Response Constraints
- Max tokens: 40 (concise, like real conversation)
- Include action tag (caricature style, not realistic)
- Stay in character (no meta-commentary)
- No direct confession (player must deduce)
```

See complete templates in `references/immersive-acting-techniques.md`.

## Gemini API Integration (Enhanced)

### Streaming with Emotional States

```typescript
import { google } from '@ai-sdk/google'
import { streamText } from 'ai'

export async function POST(req: Request) {
  const { messages, characterId, playerProfile } = await req.json()

  const character = await getCharacter(characterId)
  const adaptiveContext = buildAdaptiveContext(playerProfile, character)

  const result = await streamText({
    model: google('gemini-2.5-flash'),
    system: character.gemini_system_instruction + "\n\n" + adaptiveContext,
    messages,
    maxTokens: 60,  // Increased from 40 for action tags
    temperature: 0.85,  // Higher for improvisation

    // Structured output (Gemini supports JSON mode)
    experimental_providerMetadata: {
      google: {
        responseFormat: {
          type: "json",
          schema: {
            action: "string",
            dialogue: "string",
            emotion: "enum[nervous,calm,angry,scared]"
          }
        }
      }
    }
  })

  return result.toDataStreamResponse()
}
```

## Cost Optimization (Updated)

### Multi-Tier Response Strategy

**Tier 1: Templates** (70% of questions, $0 cost)

Common questions → pre-written responses

```typescript
const templateLibrary = {
  "where were you": (npc) => npc.alibi_template,
  "did you know victim": (npc) => npc.relationship_template,
  "what time": (npc) => npc.timeline_template
}

if (matchesTemplate(question)) {
  return templates[match]  // Free!
}
```

**Tier 2: AI Lite** (25% of questions, Gemini Flash)

Moderate complexity → Flash model (16× cheaper)

**Tier 3: AI Pro** (5% of questions, Gemini Pro)

Complex psychological analysis → Pro model

**Savings**: 70% free + 25% cheap + 5% expensive = **85% cost reduction**

## Trust Dynamics System (New)

```typescript
class TrustDynamics {
  calculateTrust(
    interactions: Interaction[],
    playerBehavior: PlayerBehavior
  ): number {
    let trust = 50  // Neutral start

    for (const interaction of interactions) {
      if (interaction.playerWasKind) trust += 5
      if (interaction.playerWasAggressive) trust -= 10
      if (interaction.npcRevealedSecret) trust += 15
      if (interaction.playerBetrayedInfo) trust -= 30
    }

    return Math.max(0, Math.min(100, trust))
  }

  shouldRevealSecret(
    trust: number,
    secretImportance: "low" | "medium" | "high"
  ): boolean {
    const thresholds = { low: 40, medium: 60, high: 80 }
    return trust >= thresholds[secretImportance]
  }
}
```

## Workflow

```
Immersive NPC Design:
- [ ] Step 1: Define core identity (name, role, secret) [15 min]
- [ ] Step 2: Set improvisation boundaries (fixed vs flexible) [20 min]
- [ ] Step 3: Design emotional progression (3-act arc) [30 min]
- [ ] Step 4: Create acting techniques (micro-expressions, tells) [20 min]
- [ ] Step 5: Set trust thresholds (what revealed when) [15 min]
- [ ] Step 6: Write system instruction (Enhanced template) [30 min]
- [ ] Step 7: Test with 20 questions (validate consistency) [30 min]
- [ ] Step 8: Optimize costs (identify template opportunities) [20 min]
```

## Anti-Patterns

❌ **Hyper-Realistic AI**: Creates Uncanny Valley discomfort
✅ **Caricature Style**: Theatrical, emoji emotions, exaggerated actions

❌ **Static Responses**: Same dialogue every playthrough
✅ **Adaptive Dialogue**: Changes based on player profile

❌ **Unlimited Conversation**: Cost explosion
✅ **20-Message Cap**: Per NPC, prevents abuse

❌ **Ignoring Trust**: NPC treats stranger same as trusted friend
✅ **Trust Gates**: Secrets unlock at trust thresholds (40/60/80)

## Resources

**Acting Techniques**: `references/immersive-acting-techniques.md` - Korean/global methods
**Templates**: `references/system-instruction-templates.md` - 5 complete NPCs
**Player Profiling**: `references/player-behavior-analysis.md` - Tracking patterns
**Cost Guide**: `references/cost-optimization-guide.md` - Advanced strategies
**Trust System**: `references/trust-dynamics-implementation.md` - Relationship mechanics

## Success Criteria

Immersive-quality AI NPCs should:
- ✅ Adapt responses to player behavior (aggressive/diplomatic/cautious)
- ✅ Build/lose trust dynamically (information reveals at thresholds)
- ✅ Show emotional progression (calm → defensive → breakdown)
- ✅ Use action tags (theatrical body language)
- ✅ Improvise details while maintaining story beats
- ✅ Stay in character (no meta-commentary, no modern references if period piece)
- ✅ Cost <$0.002 per conversation (with template hybrid)
- ✅ Response time <3s (95th percentile)

---

**Version**: 2.0 (Immersive Edition)
**Last Updated**: 2025-01-04
**Research Base**: Korean immersive escape rooms + global immersive theater + AI NPC tech (2024-2025)
