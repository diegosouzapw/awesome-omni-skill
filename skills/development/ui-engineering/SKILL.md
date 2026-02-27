---
name: ui-engineering
created_by: "云中江树 (Yunzhong Jiangshu)"
description: Expert system for creating high-quality, brand-aligned user interfaces. Use this skill when asked to design websites, create components, or implement specific visual styles. It contains specifications for major tech companies, design trends, and specific site types like blogs or presentations.

license: MIT
---

# UI Engineering Skill

## Overview

This skill provides a comprehensive library of **UI Engineering Specifications**. Each specification acts as a "stylesheet for logic", defining the exact colors, typography, interactions, and mental models for a specific style.

**When to use:**
- "Design a website like Apple"
- "Create a personal blog"
- "Build a dashboard like Stripe"
- "Make a PPT-style website"

## How to Use

### 1. Identify the Style
First, determine the most appropriate style for the user's request.
- Consult [Registry](references/_meta_registry.md) for a complete list of 60+ styles.

### 2. Read the Spec
Once a style is identified (e.g., `stripe`, `personal_blog`), read the corresponding reference file.
- Example: If the style is `stripe`, read `references/stripe.md`.

### 3. Apply the Rules
Use the Design Tokens, Component Recipes, and Tech Stack defined in the spec `references/[spec_name].md` to generate code. **Do not deviate** from the spec's defined hex codes or physics unless explicitly asked.

## Available References

### Core Registry
- [Meta Registry](references/_meta_registry.md): The master index of all styles, categorized by industry and vibe.

### High-Frequency Styles
- **SaaS / Fintech**: [Stripe](references/stripe.md), [Linear](references/linear.md)
- **Content / Blog**: [Personal Blog](references/personal_blog.md), [Notion](references/notion.md)
- **Presentation**: [Presentation Scroll](references/presentation_scroll.md)
- **Big Tech**: [Apple](references/apple.md), [Google](references/google.md), [Microsoft](references/microsoft.md)

## Skill Structure
```
ui-engineering/
├── SKILL.md
└── references/
    ├── _meta_registry.md  <- START HERE
    ├── apple.md
    ├── stripe.md
    ├── personal_blog.md
    ├── ... (60+ specs)
```
