---
name: project-frontend-design
description: comprehensive design guide for the Meatballs project, combining high-quality frontend principles with the specific "Craft & Chao" design system. Use this skill for all frontend work in this project.
---

This skill defines the specific design language for the **Meatballs** project, merging the "Craft & Chao" aesthetic with high-quality production standards. Avoid generic "AI slop" by strictly adhering to these specific theme patterns.

## 🎨 Core Design Philosophy: "Craft & Chao"

Commit to this bold aesthetic direction:
- **Warm & Authentic**: Evoke appetite and warmth using rich orange-red gradients. The site should feel "cooked", not sterile.
- **Premium Quality**: Use refined typography, generous spacing, and glass-morphism effects (`backdrop-blur`).
- **Mobile-First**: Designs must be optimized for single-column, touch-friendly interactions (min 44px targets).

## 📐 Theme & Aesthetics Guidelines

### 1. Color System
Do not use generic colors. Use the specific project palette:
- **Primary**: `orange-500` to `red-600` gradients. Use for CTAs, price highlights.
- **Background**: `#FDFBF7` (Warm Off-White). **Never** use pure white `#FFFFFF` for the main page background.
- **Neutral**: `slate-50` to `slate-900` for text and borders.
- **Semantic**: `green` for success, `red` for warning.
- **Technique**: Use opacity layers (`/90`, `/50`) and gradients to create depth and sophistication.

### 2. Typography
Avoid generic font stacks.
- **UI & Body**: `font-sans` (Clean, readable).
- **Accents & Prices**: `font-serif` (Premium, editorial feel).
- **Hierarchy**:
  - Hero: `text-4xl`, Extrabold.
  - Prices: `text-5xl` (Success page), `text-xl` (Cards).
  - Labels: Uppercase, `text-xs`, Bold, wide tracking.

### 3. Component Patterns
- **Buttons**:
  - Primary: Gradient `from-orange-500 to-red-600`, `rounded-full`, with hover scale effects.
  - Secondary: `orange-50` background, `orange-700` text.
- **Cards**:
  - White background, `rounded-3xl` (Very rounded), `shadow-sm` transitioning to `shadow-xl` on hover.
- **Inputs**: `rounded-xl`, `bg-slate-50`, focus ring in `orange-500`.

### 4. Motion & Interaction
Create a living interface, not a static page.
- **Micro-interactions**: Use `active:scale-95` on clickable elements.
- **Transitions**: Default to `duration-300`. Use `duration-700` for entrances.
- **Feedback**: Visual state must reflect user actions immediately (e.g., pulse animations).

## 🔧 Implementation Strategy

- **Tailwind CSS**: Use utility classes for almost everything.
- **Icons**: Lucide React (`size={14}` to `size={20}`).
- **Images**: High-quality food photography with gradient overlays for text readability.
- **Internationalization (i18n)**: All user-facing text must be internationalized using `next-intl`. **Never** hardcode strings; use translation keys (e.g., `t('menu.title')`).

## 🚫 Anti-Patterns (What to Avoid)
- **Generic Aesthetics**: Do not use "startup blue" or default Bootstrap/Material styles.
- **Sterility**: Avoid cold grays or pure whites without the warm off-white base.
- **Flatness**: Always use shadows, gradients, or blur to add depth to flat elements.
- **Tiny Targets**: Never make touch targets smaller than 44px.

**Remember**: The goal is a "Premium, Warm, Food-Focused" experience. Implementation must be meticulous and polished.
