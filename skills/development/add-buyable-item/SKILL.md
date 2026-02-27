---
name: add-buyable-item
description: Add a new one-time shop boost (buyable item) consistent with the app’s cat-petting theme and design-concept-reference.png. Produces the boosts.json entry, icon filename/path, accessible imageDescription, and concise icon generation instructions.
---

# Add Buyable Item

## Instructions

1. Preconditions

- This skill is for ONE-TIME purchase boosts only.
- The boost MUST conform to the existing content + validation:
  - Boost schema/types live in: src/game/types.ts and src/game/schema.ts
  - Boost content lives in: src/content/boosts/boosts.json
- If the requested behavior cannot be expressed by an existing boost type, do NOT invent fields silently:
  - Either map it to an existing supported type, or propose a new type and list the required code changes (schema + engine + UI).

2. Review existing items for progression

- ALWAYS read src/content/boosts/boosts.json to see existing items.
- Analyze the progression: prices, effects, and conceptual "upgrade" trajectory.
- New items should feel like an "upgrade" conceptually (not related, but better/fancier/more premium).
- Example progression: Warm Gloves → Auto Scratcher → (something even better).
- Suggested price should be higher than existing items to reflect progression.

3. Theme + style alignment

- Item concept must fit "cozy cat petting" (grooming, comfort, toys, treats, relaxation).
- Visual style must match the UI reference at repo root: design-concept-reference.png
  - Warm palette (beige/orange/browns), soft darker-brown outlines (not black), simple shapes, subtle 2–4 tone shading
  - Icon must work at small size, centered, no background, no text.

4. Define the new item fields (per existing schema)

- Produce: id, title, description, price, icon, type + type-specific params (per src/game/types.ts), imageDescription, and itemDetails.
- description: short, user-facing, includes the numeric effect (e.g. "Clicks give x2 pets.").
- imageDescription: 1–2 sentences describing ONLY what's visible in the icon, suitable for accessibility and for generating the icon. Do not format it like a prompt.
- itemDetails: 1–2 sentences about the item's characteristics, quality, or special features that make it feel premium/upgraded. This will be used to enhance the AI image generation.

5. Choose icon asset name

- Filename must be deterministic and match id:
  - public/icons/<id>.png
  - JSON icon path: /icons/<id>.png
- Return the exact filename to create.

6. Generate the icon automatically

- After defining all item fields, use the Bash tool to run:
  npm run generate-icon "<id>" "<imageDescription>" "<itemDetails>"
- This will automatically generate and save the icon to public/icons/<id>.png
- Wait for the generation to complete (may take 5-20 seconds)
- If generation fails (insufficient credits, API error, etc.):
  - Report the specific error
  - Provide the COMPLETE PROMPT for manual generation (imageDescription + itemDetails + base style requirements)
  - Suggest manual creation using the provided prompt

7. Update boosts.json

- Append the new item object to src/content/boosts/boosts.json.
- Keep JSON valid.
- Ensure id is unique.
- Ensure the fields exactly match the schema in src/game/schema.ts.

8. Output format (must follow)
   Return exactly these sections, in this order:
   A) New item (human spec): title, effect summary, price, why it fits theme, why it's an upgrade
   B) Item details: the itemDetails field for image generation context
   C) JSON entry: single JSON object to paste into boosts.json
   D) Asset: icon filename + icon path
   E) Icon generation:
      - If successful: confirmation that icon was generated automatically
      - If failed: error message + COMPLETE PROMPT for manual generation (see step 6)
   F) Checklist

## Examples

Example A (click multiplier)
A) New item (human spec)

- Title: Soft Brush
- Effect: Clicks give x3 pets.
- Price: 180
- Fit: Brushing is cozy cat-care and matches the core "petting" loop.
- Upgrade: More advanced than basic gloves, offers higher multiplier and premium grooming quality.

B) Item details
Premium natural bristle brush designed for gentle, effective grooming. Ergonomic beige wooden handle with soft brown trim and comfortable grip for extended petting sessions.

C) JSON entry
{
"id": "soft_brush",
"title": "Soft Brush",
"description": "Clicks give x3 pets.",
"type": "clickMultiplier",
"value": 3,
"price": 180,
"icon": "/icons/soft_brush.png",
"imageDescription": "A small cat grooming brush with rounded edges and a warm beige handle, outlined in soft brown. Subtle shading with a tiny pawprint detail, centered on a transparent background."
}

D) Asset

- Filename: public/icons/soft_brush.png
- Path: /icons/soft_brush.png

E) Icon generation
Running: npm run generate-icon "soft_brush" "A small cat grooming brush with rounded edges and a warm beige handle, outlined in soft brown. Subtle shading with a tiny pawprint detail, centered on a transparent background." "Premium natural bristle brush designed for gentle, effective grooming. Ergonomic beige wooden handle with soft brown trim and comfortable grip for extended petting sessions."

Icon generated successfully and saved to public/icons/soft_brush.png

F) Checklist

- [x] Icon generated automatically at public/icons/soft_brush.png
- [ ] Append the JSON object to src/content/boosts/boosts.json (valid JSON)
- [ ] Confirm the object matches src/game/schema.ts
- [ ] Restart dev server and verify icon appears in shop

Example B (auto click - successful generation)
A) New item (human spec)

- Title: Cozy Scratcher
- Effect: +1 pet every 1 second.
- Price: 220
- Fit: A scratcher is a natural cat item and supports passive petting.
- Upgrade: Automated passive benefit, more expensive than manual boost items, represents automation tier.

B) Item details
Sturdy cardboard scratching pad with natural texture and reinforced edges. Designed for durability and continuous use, with eco-friendly materials in warm neutral tones that complement any cat corner.

C) JSON entry
{
"id": "cozy_scratcher",
"title": "Cozy Scratcher",
"description": "+1 pet every 1 second.",
"type": "autoClick",
"value": 1,
"intervalMs": 1000,
"price": 220,
"icon": "/icons/cozy_scratcher.png",
"imageDescription": "A simple cardboard cat scratcher pad with rounded corners in warm tan tones and soft brown outlines. Light shading and a small stitched-edge detail, centered on a transparent background."
}

D) Asset

- Filename: public/icons/cozy_scratcher.png
- Path: /icons/cozy_scratcher.png

E) Icon generation
Running: npm run generate-icon "cozy_scratcher" "A simple cardboard cat scratcher pad with rounded corners in warm tan tones and soft brown outlines. Light shading and a small stitched-edge detail, centered on a transparent background." "Sturdy cardboard scratching pad with natural texture and reinforced edges. Designed for durability and continuous use, with eco-friendly materials in warm neutral tones that complement any cat corner."

Icon generated successfully and saved to public/icons/cozy_scratcher.png

F) Checklist

- [x] Icon generated automatically at public/icons/cozy_scratcher.png
- [ ] Append the JSON object to src/content/boosts/boosts.json (valid JSON)
- [ ] Confirm the object matches src/game/schema.ts
- [ ] Restart dev server and verify icon appears in shop

Example C (generation failed - manual fallback)
A) New item (human spec)

- Title: Premium Catnip Toy
- Effect: Clicks give x4 pets.
- Price: 300
- Fit: Catnip toys enhance engagement and petting sessions.
- Upgrade: Higher multiplier, premium quality item.

B) Item details
Handcrafted organic catnip-filled plush toy with premium velvet exterior in warm cream and dusty sage tones.

C) JSON entry
{
"id": "premium_catnip_toy",
"title": "Premium Catnip Toy",
"description": "Clicks give x4 pets.",
"type": "clickMultiplier",
"value": 4,
"price": 300,
"icon": "/icons/premium_catnip_toy.png",
"imageDescription": "A small plush mouse toy in warm cream color with soft brown outline, subtle sage green accent on the ears, centered on transparent background."
}

D) Asset

- Filename: public/icons/premium_catnip_toy.png
- Path: /icons/premium_catnip_toy.png

E) Icon generation

**Status**: ❌ Generation failed - 403 Forbidden (insufficient API credits)

**Manual generation prompt**:
```
A small plush mouse toy in warm cream color with soft brown outline, subtle sage green accent on the ears, centered on transparent background.

Item details: Handcrafted organic catnip-filled plush toy with premium velvet exterior in warm cream and dusty sage tones.

Style requirements: Muted, cozy 2D cartoon icons that match design-concept-reference.png: clean simple shapes, soft darker-brown outlines (not black), subtle 2–4 tone shading, and a mostly neutral base (cream/tan/beige) with at most one small muted accent color (sage, dusty blue, muted teal, soft pink, lavender-gray) that won't clash with future cat themes. Single centered object on a transparent background, readable at small size, no text, no background scene, no 3D/photorealism, no neon or highly saturated colors.
```

**Instructions**: Use the above prompt in your preferred AI image generator, save as PNG (1024x1024 recommended), and save to public/icons/premium_catnip_toy.png

F) Checklist

- [ ] Generate icon manually using the prompt above and save to public/icons/premium_catnip_toy.png
- [x] Append the JSON object to src/content/boosts/boosts.json (valid JSON)
- [x] Confirm the object matches src/game/schema.ts
- [ ] Restart dev server and verify icon appears in shop
