---
name: divek-bi-visual-audit
description: Visual compliance auditing for DiveK brand identity. Use when reviewing UI screens, component libraries, landing pages, design handoff specs, CSS tokens, or visual QA reports for alignment with DiveK color palette, typography, and cinematic-minimal style direction.
---

# DiveK BI Visual Audit

## Run Visual Audit Workflow

1. Identify input type.
   - `screen`: mockup, screenshot, page implementation.
   - `system`: design tokens, theme files, component kit.
   - `campaign`: landing page or ad creative.
2. Load sources in this order.
   - `docs/DiveK_Brand_Identity_Guidelines_EN.md`
   - `references/visual-rules.md`
3. Extract current visual tokens.
   - Background colors, accent colors, typography stacks, key spacing.
4. Evaluate five checks.
   - Palette compliance
   - Highlight behavior
   - Typography compliance
   - Minimalist-cinematic layout
   - Immersion interruptions
5. Assign severity.
   - `P0`: breaks premium direction or default dark immersion.
   - `P1`: visible mismatch in primary surfaces.
   - `P2`: polish inconsistency.
6. Deliver exact remediation with token-level instructions.

## Enforce Required Specs

- Use theater-black default base: `#121212`.
- Use accent colors for intent highlights only.
  - Neon Mint `#00FFCC`
  - Seoul Purple `#8A2BE2`
- Avoid primary-color language app look and avoid Korea-flag red/blue motifs.
- Use UI fonts `Inter` or `Outfit` for English.
- Use `Pretendard` for Korean subtitles.
- Keep screen composition minimal, focused, and low-clutter.

## Audit Highlight Behavior

- Use accent highlights for searched words and key learning moments.
- Avoid spraying accent colors across non-semantic elements.
- Keep contrast readable against dark surfaces.

## Return Visual QA Report

Use this format:

```markdown
## Visual Audit Result
Status: fail
Scope: search results screen

### Violations
- [P0] Base background changed from #121212 to #F7F8FA.
- [P1] Accent colors used on unrelated buttons, reducing highlight meaning.
- [P1] Korean subtitle font not set to Pretendard.

### Fix Instructions
1. Restore dark base surfaces to #121212 family.
2. Limit accent use to keyword highlights and active learning cues.
3. Set subtitle font stack to Pretendard fallback chain.

### Verification
- Dark mode remains default.
- Highlight density is intentional and sparse.
- Typography follows DiveK standard.
```
