---
name: self-improvement
description: Zoe's self-improvement system - learns from corrections and user preferences
version: 1.0.0
author: zoe-team
api_only: true
priority: 1
triggers:
  - "remember that"
  - "don't forget"
  - "i prefer"
  - "from now on"
  - "always remember"
  - "never forget"
  - "i told you"
allowed_endpoints:
  - "GET /api/learnings"
  - "POST /api/learnings/*/confirm"
  - "POST /api/learnings/*/dismiss"
tags:
  - meta
  - self-improvement
---
# Self-Improvement

## When to Use
Automatically active. This skill runs in the background to detect when users
correct Zoe or express preferences. Explicit triggers like "remember that" or
"from now on" create immediate learnings.

## How It Works
1. After each conversation, scan for correction signals
2. Extract the learning (what was wrong, what is correct)
3. Store in memory with user ownership
4. Use learnings to improve future responses

## Security
- Only trusted sources can create learnings (Trust Gate integration)
- Owner corrections auto-confirm
- Trusted contact corrections need user review
- Learnings expire after 30 days if not confirmed
