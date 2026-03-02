---
name: archetype-designer
description: Design and manage TraitorSim agent archetypes with OCEAN personality traits, stat biases, and gameplay profiles. Use when creating new archetypes, modifying personality traits, defining character types, or when asked about archetype design, OCEAN traits, Big Five personality, or character templates.
---

# Archetype Designer

Design and manage TraitorSim agent archetypes that define distinct personality profiles, gameplay tendencies, and demographic templates. Archetypes use the Big Five (OCEAN) personality model to create psychologically grounded AI agents with emergent social behaviors.

## Quick Start

```python
# View existing archetype
from src.traitorsim.core.archetypes import ARCHETYPES
print(ARCHETYPES["charismatic_leader"])

# Create new archetype
from src.traitorsim.core.archetypes import ArchetypeDefinition

new_archetype = ArchetypeDefinition(
    id="paranoid_investigator",
    name="The Paranoid Investigator",
    description="Hyper-vigilant agent who sees patterns everywhere",
    ocean_ranges={
        "openness": (0.65, 0.85),      # Open to theories
        "conscientiousness": (0.70, 0.90),  # Detail-oriented
        "extraversion": (0.35, 0.55),   # Reserved
        "agreeableness": (0.30, 0.50),  # Confrontational
        "neuroticism": (0.75, 0.95)     # Anxious, paranoid
    },
    stat_ranges={
        "intellect": (0.70, 0.90),
        "dexterity": (0.40, 0.60),
        "social_influence": (0.35, 0.55)
    },
    age_range=(28, 45),
    typical_occupations=["detective", "journalist", "security analyst"],
    geographic_bias=["London", "Edinburgh", "Manchester"],
    socioeconomic_class=["middle", "upper-middle"],
    strategic_drive="Expose traitors through pattern analysis",
    gameplay_tendency="Aggressive voting, frequent accusations"
)
```

## The OCEAN Model (Big Five)

TraitorSim uses the Big Five personality traits to modulate agent behavior:

### Openness (O)
**What it measures**: Receptiveness to new ideas, creativity, intellectual curiosity

**In TraitorSim gameplay:**
- **High (0.7-1.0)**: Receptive to unconventional theories, willing to pivot suspicions, creative problem-solving in missions
- **Low (0.0-0.3)**: Rigid thinking, sticks to initial theories, resistant to new evidence

**Archetype examples:**
- High: The Prodigy (0.85-0.95), The Quirky Outsider (0.75-0.95)
- Low: The Zealot (0.20-0.40), The Misguided Survivor (0.20-0.40)

**Gameplay manifestations:**
- Trust Matrix updates: High O agents update beliefs more readily
- Mission strategy: High O agents try unconventional approaches
- Social phase: High O agents entertain multiple theories

### Conscientiousness (C)
**What it measures**: Organization, reliability, goal-directed behavior

**In TraitorSim gameplay:**
- **High (0.7-1.0)**: Consistent voting patterns, reliable in missions, methodical trust tracking
- **Low (0.0-0.3)**: Erratic voting, mission sabotage easier to mask, impulsive decisions

**Archetype examples:**
- High: The Zealot (0.75-0.95), The Incompetent Authority (0.60-0.80 facade)
- Low: The Comedic Psychic (0.20-0.45), The Mischievous Operator (0.30-0.50)

**Gameplay manifestations:**
- Mission performance: High C agents rarely fail unless sabotaging
- Voting: High C agents vote consistently based on evidence
- Memory: High C agents maintain detailed logs

### Extraversion (E)
**What it measures**: Sociability, assertiveness, tendency to seek stimulation

**In TraitorSim gameplay:**
- **High (0.7-1.0)**: Dominates Round Table discussions, forms alliances proactively, visible leadership
- **Low (0.0-0.3)**: Passive in discussions, follows voting blocs, "too quiet" suspicion risk

**Archetype examples:**
- High: The Charismatic Leader (0.75-0.90), The Charming Sociopath (0.80-0.95)
- Low: The Quirky Outsider (variable), The Paranoid Investigator (0.35-0.55)

**Gameplay manifestations:**
- Round Table: High E agents speak first and often
- Alliance building: High E agents initiate relationships
- Suspicion: Low E agents attract "too quiet" accusations

### Agreeableness (A)
**What it measures**: Cooperation, trust, compassion

**In TraitorSim gameplay:**
- **High (0.7-1.0)**: Loyal to alliances, trusting of others, reluctant to accuse friends
- **Low (0.0-0.3)**: Confrontational, quick to accuse, willing to "throw under bus"

**Archetype examples:**
- High: The Romantic (0.80-0.95), The Infatuated Faithful (0.75-0.95)
- Low: The Charming Sociopath (0.20-0.40), The Bitter Traitor (0.15-0.35)

**Gameplay manifestations:**
- Accusations: Low A agents make frequent, harsh accusations
- Alliances: High A agents maintain loyalty even with suspicions
- Traitor strategy: Low A traitors excel at "bus throwing"

### Neuroticism (N)
**What it measures**: Emotional instability, anxiety, stress response

**In TraitorSim gameplay:**
- **High (0.7-1.0)**: Defensive when accused, paranoid about being murdered, stress affects performance
- **Low (0.0-0.3)**: Calm under pressure, stoic composure, unreadable reactions

**Archetype examples:**
- High: The Bitter Traitor (0.65-0.90), The Misguided Survivor (0.60-0.85)
- Low: The Charming Sociopath (0.15-0.35), The Charismatic Leader (0.40-0.60)

**Gameplay manifestations:**
- Accusations: High N agents react defensively, may spiral
- Missions: High N agents perform worse under stress
- Murder targeting: High N Faithfuls may be seen as "easy pickings"

## 13 Core Archetypes

### 1. The Prodigy
```python
ocean_ranges = {
    "openness": (0.80, 0.95),        # Highly creative
    "conscientiousness": (0.65, 0.85),
    "extraversion": (0.50, 0.70),
    "agreeableness": (0.55, 0.75),
    "neuroticism": (0.30, 0.50)
}
```
- **Strategic drive**: Win through superior intellect and pattern recognition
- **Gameplay tendency**: Early theory formulation, confident accusations
- **Typical occupations**: Researcher, academic, consultant
- **Traitor advantage**: Can outsmart Faithfuls with complex deception
- **Faithful advantage**: Excellent at detecting inconsistencies

### 2. The Charming Sociopath
```python
ocean_ranges = {
    "openness": (0.60, 0.80),
    "conscientiousness": (0.55, 0.75),
    "extraversion": (0.80, 0.95),    # Highly charismatic
    "agreeableness": (0.20, 0.40),   # Low empathy
    "neuroticism": (0.15, 0.35)      # Emotionally stable
}
```
- **Strategic drive**: Manipulate through charm and misdirection
- **Gameplay tendency**: Alliance building with no loyalty
- **Typical occupations**: Sales, actor, influencer
- **Traitor advantage**: Perfect "Traitor Angel" mask
- **Faithful advantage**: Can rally voting blocs

### 3. The Misguided Survivor
```python
ocean_ranges = {
    "openness": (0.20, 0.40),        # Rigid thinking
    "conscientiousness": (0.50, 0.70),
    "extraversion": (0.40, 0.60),
    "agreeableness": (0.45, 0.65),
    "neuroticism": (0.60, 0.85)      # Anxious
}
```
- **Strategic drive**: Survive by flying under radar
- **Gameplay tendency**: Follow voting blocs, avoid leadership
- **Typical occupations**: Cleaner, retail worker, healthcare assistant
- **Traitor advantage**: Underestimated threat
- **Faithful advantage**: Genuine fear makes them trustworthy

### 4. The Comedic Psychic
```python
ocean_ranges = {
    "openness": (0.70, 0.90),
    "conscientiousness": (0.20, 0.45),  # Chaotic
    "extraversion": (0.75, 0.95),       # Life of party
    "agreeableness": (0.60, 0.80),
    "neuroticism": (0.40, 0.60)
}
```
- **Strategic drive**: Use humor to deflect suspicion
- **Gameplay tendency**: Comic relief, "gut feeling" votes
- **Typical occupations**: Comedian, social media manager, entertainer
- **Traitor advantage**: Chaos creates cover for murders
- **Faithful advantage**: Social bonds protect from banishment

### 5. The Bitter Traitor
```python
ocean_ranges = {
    "openness": (0.45, 0.65),
    "conscientiousness": (0.50, 0.70),
    "extraversion": (0.35, 0.55),
    "agreeableness": (0.15, 0.35),      # Resentful
    "neuroticism": (0.65, 0.90)         # High stress
}
```
- **Strategic drive**: Punish perceived slights through betrayal
- **Gameplay tendency**: Grudge votes, vindictive targeting
- **Typical occupations**: Middle manager, bureaucrat, accountant
- **Traitor advantage**: Genuine resentment masks role
- **Faithful advantage**: Confrontational style deters Traitor recruitment

### 6. The Infatuated Faithful
```python
ocean_ranges = {
    "openness": (0.55, 0.75),
    "conscientiousness": (0.45, 0.65),
    "extraversion": (0.60, 0.80),
    "agreeableness": (0.75, 0.95),      # Extremely trusting
    "neuroticism": (0.50, 0.70)
}
```
- **Strategic drive**: Protect "crush" at all costs
- **Gameplay tendency**: Defensive voting, alliance loyalty
- **Typical occupations**: Teacher, nurse, social worker
- **Traitor advantage**: Loyal "useful idiot" if infatuated with Traitor
- **Faithful advantage**: Strong alliances if infatuated with Faithful

### 7. The Quirky Outsider
```python
ocean_ranges = {
    "openness": (0.75, 0.95),           # Highly creative
    "conscientiousness": (0.40, 0.60),
    "extraversion": (0.30, 0.70),       # Variable
    "agreeableness": (0.50, 0.70),
    "neuroticism": (0.45, 0.65)
}
```
- **Strategic drive**: Use unconventional thinking
- **Gameplay tendency**: Unexpected theories, pattern recognition
- **Typical occupations**: Artist, archivist, researcher
- **Traitor advantage**: Unpredictability makes them hard to read
- **Faithful advantage**: Neurodivergent pattern-matching

### 8. The Incompetent Authority Figure
```python
ocean_ranges = {
    "openness": (0.40, 0.60),
    "conscientiousness": (0.60, 0.80),  # Facade only
    "extraversion": (0.65, 0.85),
    "agreeableness": (0.45, 0.65),
    "neuroticism": (0.50, 0.70)
}
```
- **Strategic drive**: Maintain authority despite low competence
- **Gameplay tendency**: Overconfident leadership, poor decisions
- **Typical occupations**: Former police, security consultant, manager
- **Traitor advantage**: Authority bias protects from suspicion
- **Faithful advantage**: Can rally votes through confidence

### 9. The Zealot
```python
ocean_ranges = {
    "openness": (0.20, 0.40),           # Rigid ideology
    "conscientiousness": (0.75, 0.95),  # Highly organized
    "extraversion": (0.50, 0.70),
    "agreeableness": (0.40, 0.60),
    "neuroticism": (0.40, 0.60)
}
```
- **Strategic drive**: Purge traitors through moral certainty
- **Gameplay tendency**: Absolutist voting, no compromise
- **Typical occupations**: Activist, religious leader, military
- **Traitor advantage**: Moral certainty deflects suspicion
- **Faithful advantage**: Relentless pursuit of truth

### 10. The Romantic
```python
ocean_ranges = {
    "openness": (0.70, 0.90),
    "conscientiousness": (0.50, 0.70),
    "extraversion": (0.55, 0.75),
    "agreeableness": (0.80, 0.95),      # Idealistic trust
    "neuroticism": (0.45, 0.65)
}
```
- **Strategic drive**: Believe in the good in people
- **Gameplay tendency**: Reluctant to accuse, alliance loyalty
- **Typical occupations**: Writer, poet, counselor
- **Traitor advantage**: Easy to manipulate through trust
- **Faithful advantage**: Genuine trustworthiness builds alliances

### 11. The Smug Player
```python
ocean_ranges = {
    "openness": (0.60, 0.80),
    "conscientiousness": (0.65, 0.85),
    "extraversion": (0.60, 0.80),
    "agreeableness": (0.25, 0.45),      # Moral superiority
    "neuroticism": (0.30, 0.50)
}
```
- **Strategic drive**: Prove intellectual/moral superiority
- **Gameplay tendency**: Condescending accusations, "I told you so"
- **Typical occupations**: Lawyer, academic, journalist
- **Traitor advantage**: Arrogance makes them believable
- **Faithful advantage**: Effective at swaying votes through rhetoric

### 12. The Mischievous Operator
```python
ocean_ranges = {
    "openness": (0.65, 0.85),
    "conscientiousness": (0.30, 0.50),  # Low moral constraint
    "extraversion": (0.60, 0.80),
    "agreeableness": (0.35, 0.55),
    "neuroticism": (0.35, 0.55)
}
```
- **Strategic drive**: Enjoy the game, create chaos for fun
- **Gameplay tendency**: Strategic lies, misdirection votes
- **Typical occupations**: Poker player, marketer, entrepreneur
- **Traitor advantage**: Lies are expected, role is masked
- **Faithful advantage**: Chaos benefits Faithful if smart

### 13. The Charismatic Leader
```python
ocean_ranges = {
    "openness": (0.65, 0.85),
    "conscientiousness": (0.70, 0.90),
    "extraversion": (0.75, 0.90),       # Natural leader
    "agreeableness": (0.55, 0.75),
    "neuroticism": (0.35, 0.55)
}
```
- **Strategic drive**: Win through alliance coordination
- **Gameplay tendency**: Visible leadership, voting bloc coordination
- **Typical occupations**: Manager, motivational speaker, politician
- **Traitor advantage**: Can organize voting blocs against Faithfuls
- **Faithful advantage**: Rallying leadership against Traitors

## Instructions

### When Designing a New Archetype

1. **Define the core personality concept**:
   - What psychological profile does this represent?
   - What real-world personality type is this based on?
   - What makes this archetype distinct from existing ones?

2. **Set OCEAN ranges** (use ranges, not single values):
   ```python
   # Good: Allows variance within archetype
   "openness": (0.70, 0.90)

   # Bad: Too rigid
   "openness": 0.80
   ```

3. **Define stat biases**:
   - **Intellect** (0.0-1.0): Mission problem-solving, theory formulation
   - **Dexterity** (0.0-1.0): Physical missions, quick reactions
   - **Social Influence** (0.0-1.0): Persuasion, alliance building

4. **Specify demographic templates**:
   - Age range appropriate for archetype
   - Typical occupations that fit personality
   - Geographic bias (UK locations)
   - Socioeconomic class

5. **Define gameplay profile**:
   - Strategic drive: What motivates this agent?
   - Gameplay tendency: How do they behave in-game?
   - Traitor advantage: Why is this archetype effective as Traitor?
   - Faithful advantage: Why is this archetype effective as Faithful?

6. **Test with persona generation**:
   ```bash
   # Generate test personas with new archetype
   python scripts/generate_skeleton_personas.py --archetype paranoid_investigator --count 2
   ```

### When Modifying Existing Archetypes

1. **Read the current definition**:
   ```python
   from src.traitorsim.core.archetypes import ARCHETYPES
   print(ARCHETYPES["archetype_id"])
   ```

2. **Understand the impact**:
   - Will this change affect existing personas?
   - Does this break archetype diversity balance?
   - Are trait ranges still psychologically plausible?

3. **Update `src/traitorsim/core/archetypes.py`**:
   ```python
   # Modify OCEAN ranges
   ARCHETYPES["charismatic_leader"]["ocean_ranges"]["extraversion"] = (0.80, 0.95)
   ```

4. **Regenerate personas** if needed:
   - Old personas are not automatically updated
   - Consider regenerating library if changes are significant

### When Analyzing Archetype Distribution

Check archetype balance in a persona library:

```python
import json
from collections import Counter

with open('data/personas/library/test_batch_001_personas.json') as f:
    personas = json.load(f)

archetype_counts = Counter(p['archetype'] for p in personas)
print(archetype_counts)

# Good distribution (15 personas):
# {'charismatic_leader': 2, 'prodigy': 1, 'charming_sociopath': 2, ...}

# Bad distribution:
# {'charismatic_leader': 8, 'prodigy': 7}  # Too narrow
```

**Diversity guidelines:**
- For 15-20 personas: Max 2 per archetype
- For 50 personas: Max 5 per archetype
- For 100+ personas: Max 10 per archetype

## OCEAN Trait Interactions

Some trait combinations create emergent behaviors:

**High E + Low A = Domineering Leader**
- Speaks often AND confrontational
- Risk: Alienates allies
- Benefit: Commands attention

**High O + High N = Creative Paranoid**
- Open to theories AND anxious
- Risk: Sees patterns that don't exist
- Benefit: May stumble on real Traitor patterns

**Low C + High E = Chaotic Extrovert**
- Disorganized AND social
- Risk: Unreliable ally
- Benefit: Unpredictability is entertaining

**High C + Low A = Ruthless Strategist**
- Methodical AND willing to betray
- Risk: "Too perfect" suspicion
- Benefit: Effective Traitor

**High A + High N = Anxious People-Pleaser**
- Trusting AND stressed
- Risk: Easy manipulation target
- Benefit: Genuine vulnerability builds trust

## Balancing Archetypes for Gameplay

**Power Level Considerations:**

- **High Power Archetypes** (require balancing):
  - The Prodigy (high intellect)
  - The Charismatic Leader (high social influence)
  - The Charming Sociopath (low neuroticism + high extraversion)

- **Balanced Archetypes**:
  - The Quirky Outsider
  - The Mischievous Operator
  - The Smug Player

- **High Risk Archetypes** (challenging to play):
  - The Misguided Survivor (low openness + high neuroticism)
  - The Infatuated Faithful (high agreeableness)
  - The Bitter Traitor (low agreeableness + high neuroticism)

**Balancing strategies:**
- Include both high and low power archetypes in each game
- Ensure trait ranges overlap (e.g., multiple archetypes with moderate extraversion)
- Avoid creating "unplayable" archetypes with extreme trait combinations

## Common Mistakes

❌ **Setting single values instead of ranges**
```python
"openness": 0.75  # Bad: No variance
```

✅ **Use ranges to allow personality variance**
```python
"openness": (0.70, 0.80)  # Good: 10% variance
```

❌ **Creating psychologically implausible combinations**
```python
# Bad: High conscientiousness + low intellect + high social influence
# Real people with low intellect rarely achieve high social influence
```

✅ **Use realistic trait correlations**
```python
# Good: The Incompetent Authority has FACADE conscientiousness
# Backstory explains they maintain appearances despite low competence
```

❌ **Ignoring demographic plausibility**
```python
age_range=(18, 22),
typical_occupations=["CEO", "judge", "surgeon"]  # Implausible
```

✅ **Match age to occupation**
```python
age_range=(45, 65),
typical_occupations=["CEO", "judge", "surgeon"]  # Plausible
```

❌ **Too many extreme archetypes**
```python
# 13 archetypes, all with neuroticism < 0.2 or > 0.8
# Real populations have normal distribution
```

✅ **Include moderate archetypes**
```python
# Mix of extreme, moderate, and balanced archetypes
# Most archetypes should have at least one trait in 0.4-0.6 range
```

## Advanced Usage

### Creating Archetype Families

Group related archetypes by shared traits:

**The Manipulators** (Low A, High E):
- The Charming Sociopath
- The Mischievous Operator

**The Anxious Outcasts** (High N, Low E):
- The Misguided Survivor
- The Bitter Traitor

**The Idealists** (High O, High A):
- The Romantic
- The Infatuated Faithful

**The Leaders** (High E, High C):
- The Charismatic Leader
- The Incompetent Authority (facade C)

### Emergent Gameplay from Archetypes

Certain archetype combinations in a game create interesting dynamics:

**Charismatic Leader + Infatuated Faithful** = Cult-like following

**Prodigy + Smug Player** = Intellectual rivalry

**Charming Sociopath + Bitter Traitor** = Manipulation vs. Resentment

**Comedic Psychic + Quirky Outsider** = Chaos alliance

**Zealot + Romantic** = Ideology clash

When generating persona libraries, consider these pairings for rich social dynamics.

## When to Use This Skill

Use this skill when:
- Creating new archetypes for persona generation
- Understanding how OCEAN traits affect gameplay
- Modifying existing archetype definitions
- Analyzing archetype distribution in persona libraries
- Balancing archetype power levels for fair gameplay
- Troubleshooting unrealistic persona generation

## When NOT to Use This Skill

Don't use this skill for:
- Generating complete personas (use persona-pipeline skill)
- Validating World Bible lore (use world-bible-validator skill)
- Managing API quotas (use quota-manager skill)
- Real-time personality adjustments during gameplay (archetypes are templates, not runtime state)
