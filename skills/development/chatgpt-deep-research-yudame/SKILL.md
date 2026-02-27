---
name: chatgpt-deep-research
description: DEPRECATED - Use gpt-researcher skill instead. This browser automation approach has been replaced with the local GPT-Researcher framework using OpenAI o1.
user-invocable: false
---

# ChatGPT Deep Research - DEPRECATED

**Status:** This skill is deprecated and no longer maintained.

**Replacement:** Use the `gpt-researcher` skill instead.

## Why Deprecated?

Browser automation for ChatGPT Deep Research had several limitations:
- **Fragile:** UI changes break automation
- **Requires browser:** Chrome with remote debugging
- **Requires subscription:** ChatGPT Plus/Team ($200/mo)
- **Limited control:** Can't choose model or parameters
- **High maintenance:** Selectors need constant updates

## Migration to GPT-Researcher

The `gpt-researcher` skill provides all the same capabilities with better reliability:

### Old Way (Deprecated Browser Automation)
```bash
# Complex browser automation with Chrome DevTools
# Requires ChatGPT Plus subscription
# 5-10 minutes per research
# Fragile UI automation
```

### New Way (GPT-Researcher with OpenAI GPT-5.2)
```bash
cd /Users/valorengels/src/research/podcast/tools
python gpt_researcher_run.py --file prompt.txt --output results.md
```

**Benefits:**
- ✅ No browser required
- ✅ Uses OpenAI GPT-5.2 (latest flagship model, 2025)
- ✅ Pay-per-use ($0.27-2 per search)
- ✅ 100+ sources analyzed
- ✅ Fully scriptable and reproducible
- ✅ Works in any environment
- ✅ Knowledge cutoff: August 2025 (most current)

## See Also

- **gpt-researcher skill:** `.claude/skills/gpt-researcher/SKILL.md`
- **Script location:** `podcast/tools/gpt_researcher_run.py`

## Historical Reference

This skill previously automated ChatGPT Deep Research via Chrome DevTools browser automation. It has been superseded by the local GPT-Researcher framework which provides better reliability, control, and cost-effectiveness.
