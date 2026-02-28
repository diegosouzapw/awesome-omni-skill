---
name: aiproofing-text
description: Analyzes narrative text and files to identify and remove AI-generated signals while preserving authentic voice and style. Runs input through comprehensive protocols covering vocabulary, syntax, character voice, emotional depth, and readability. Use this skill when you need to proof narrative content against AI detection patterns or humanize AI-assisted writing.
version: 1.0
---

## Purpose

This skill provides a complete AI proofing workflow for narrative Markdown files of any length and genre. It operates without requiring pre-existing metadata or manual annotations, automatically extracting narrative context and applying a systematic 16-task protocol to improve human authenticity while preserving voice.

## When to Use This Skill

Invoke this skill when you need to:
- Analyze narrative text for AI-generated signals and patterns
- Remove repetitive vocabulary, formulaic phrasing, or mechanical construction
- Enhance character voice consistency and differentiation
- Improve emotional depth and sensory grounding
- Increase syntactic variety and natural burstiness
- Verify narrative coherence before publication
- Humanize text that may have been assisted by AI

## How to Use

### Basic Invocation

```
/aiproofing-text "Analyze and revise this manuscript" path/to/manuscript.md
```

Or provide text inline:

```
/aiproofing-text "AI proof this text" "Your narrative text here"
```

### Workflow Phases

The skill executes a structured 6-phase protocol:

1. **Intake and Baseline** – Extracts narrative structure, POV, tone, characters, and establishes baseline metrics
2. **Lexical Depth** – Analyzes vocabulary diversity, idioms, and overused patterns
3. **Syntax and Grammar Flexibility** – Evaluates sentence structure, part-of-speech balance, and modal variety
4. **Readability and Flow** – Checks complexity calibration, formulaic patterns, and rhythmic burstiness
5. **Voice and Emotion** – Assesses character voice consistency, emotional intensity, and figurative language
6. **Quality Assurance** – Validates consistency, continuity, and publication readiness

### Understanding the Output

The skill generates:
- **Revised manuscript** with improvements across all categories
- **Analysis report** summarizing findings for each protocol area
- **Specific recommendations** for any remaining concerns
- **Publication readiness verdict** (Ready / Ready with tweaks / Hold)

## Protocols Included

This skill includes 20 specialized analysis protocols:

- **AIproof_plan.md** – Master workflow organizing all 16 tasks
- **manuscript_analysis.md** – Automated intake and context extraction
- **automation_playbook.md** – Detailed agent execution guide
- **AIproofcheck.md** – Quick verification checklist
- **ai_tell_checklist.md** – Fast scan for common AI signals

**Category Guides:**
- vocabulary_analysis.md – Lexical variety and clustering
- idiomatic_analysis.md – Phrase authenticity
- overused_vocabulary_analysis.md – Bureaucratic/tech drift removal
- sentence_structure_analysis.md – SVO pattern breaking
- part_of_speech_analysis.md – Noun/verb/adjective balance
- modal_epistemic_analysis.md – Uncertainty and perspective nuance
- readability_analysis.md – Density and comprehension calibration
- formulaic_pattern_analysis.md – Template disruption
- burstiness_analysis.md – Controlled stylistic surprise
- character_voice_analysis.md – Voice differentiation and consistency
- emotional_intensity_analysis.md – Sensory and emotional grounding
- metaphor_analysis.md – Figurative language and cliché replacement
- consistency_check.md – Continuity and tone cohesion
- final_analysis.md – Publication-readiness validation

## Key Features

- **Neutral and Genre-Agnostic** – Works with any narrative length, genre, tense, or POV
- **No Metadata Required** – Auto-derives context from the source text alone
- **Systematic and Thorough** – 16 sequential tasks covering linguistic, structural, and stylistic dimensions
- **Voice-Preserving** – Enhances authenticity without flattening character or narrative style
- **Agent-Ready** – Designed for autonomous execution with clear inputs and outputs
- **Iterative-Friendly** – Allows refinement and rollback between phases

## Example Workflow

**Input:** An unpublished 5,000-word short story with some AI-assisted passages

**Processing:**
1. Extract 4 characters, 3 locations, present tense, third-person limited POV
2. Scan vocabulary against overuse clusters; flag 12 repetition hotspots
3. Analyze sentence rhythms; identify 3 formulaic openings and 5 SVO patterns
4. Review emotional intensity; add sensory grounding to 4 climactic moments
5. Check voice consistency across 4 POV sections; differentiate narrator tone
6. Verify all changes integrate; confirm readability metrics match intent

**Output:** Revised manuscript + detailed report addressing each of 16 task categories

## Tips for Best Results

- Provide the full narrative context (chapters, scenes, backstory references) for most accurate analysis
- Allow the skill to complete all 6 phases before drawing conclusions
- Review and selectively accept recommendations; preserve intentional stylistic choices
- Use the "Ready with tweaks" verdict to identify areas for human refinement
- Keep pre-edit snapshots for comparison and rollback

## Technical Details

The skill includes a Python orchestration script (`scripts/aiproof_runner.py`) that helps sequence tasks and manage outputs if running via automation.

Protocol files reference each other; the skill maintains cross-references so you can jump to relevant analyses.

---

**Version:** 1.0
**Status:** Production Ready
**Last Updated:** 2025-12-20
