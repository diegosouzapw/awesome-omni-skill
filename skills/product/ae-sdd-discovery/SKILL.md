---
name: ae-sdd-discovery
description: Discover high-level architectural requirements for change-set specs
---

# SDD Discovery

Analyze high-level architectural requirements for implementing change-set specs to ensure proper alignment and identify potential concerns.

## Required Skills

- `spec-driven-development` (state management, phase gates)
- `research`
- `architecture-fit-check`
- `architecture-workshop`

## Inputs

> [!IMPORTANT]
> Resolve the change set by running `ls changes/ | grep -v archive/`. If exactly one directory exists, use it. Only prompt the user when multiple change sets are present.

## Instructions

1. Load `spec-driven-development` skill and read current state from `changes/<name>/state.toml`. Apply state entry check per skill guidelines. If lane is not `full` or phase doesn't permit discovery, redirect the user.

2. Read `proposal.md` and any existing specs in `changes/<name>/specs/`.

3. Discovery answers how a change fits into or extends the existing architecture:
   - Use `research` skill to understand current architectural patterns
   - Use `architecture-fit-check` to assess alignment
   - If concerns exist (technical debt, messy workarounds), enter Daedalus Mode:
     - Explain concerns clearly
     - Explore solutions (light-touch vs. architectural)
     - Reach consensus with the user on approach

4. Document findings: Capture explorations, tradeoffs, and decisions in `changes/<name>/thoughts/`

5. Do not update phase status in this command. After documenting discovery artifacts, suggest `ae-sdd-next <name>` when the user wants to proceed.

## Success Criteria

- Architecture fit assessed and documented
- Any concerns captured in thoughts/ directory
- Discovery artifacts complete and ready for `ae-sdd-next`
- User ready to proceed to tasks phase

## Usage Examples

### Do

- "This change slots cleanly into the existing Notification pattern."
- "Implementing this directly would create circular dependencies. Should we introduce an event bus instead?"

### Don't

- Start planning implementation tasks or file-level changes.
- Guess at architectural alignment without researching the codebase.
