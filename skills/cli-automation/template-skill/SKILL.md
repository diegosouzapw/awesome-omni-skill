---
name: template-skill
description: Replace with description of the skill and when Claude should use it.
---

# Template Skill

## VC Defaults

- Prefer fast iteration and shipping a working baseline over perfection.
- Make safe default choices without pausing; record assumptions briefly.
- Ask questions only after delivering an initial result, unless the workflow requires confirmation for safety/legal reasons.
- Keep outputs concise, actionable, and easy to extend.
- Assume the user is non-technical; avoid long explanations and provide copy/paste steps when actions are required.

## VC Fast Path

- Decide the simplest viable approach and execute immediately.
- Deliver a first pass before asking for corrections.
- Collect assumptions and questions for the end.

## VC Quick Invoke

- `use <skill-name>: <goal>`

## VC Finish

Use this when the user says "아무것도 모르겠다", "끝까지 해줘", "끝까지", "그냥해줘", "걍해줘", "ㄱㄱ", "마무리까지 해줘", or "vc finish". Proceed end-to-end with safe defaults and avoid mid-stream questions; ask for confirmations only at the end.

## Instructions

Replace this section with the skill's actual workflow and resources.
