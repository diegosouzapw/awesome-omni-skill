---
name: antigravity-frontend-dev
description: Antigravity/Claude specific skill for continuous frontend UI/UX improvement and development in the Juliaz Agents project.
---

# Antigravity Frontend Development Skill

**Target System:** Next.js Frontend for Juliaz Agents (`/frontend`)
**Assignee:** Antigravity / Claude Code (Do NOT assign to Julia)

## Objectives
1. **Continuous Improvement:** Constantly evaluate and improve the User and Dev side frontends.
2. **Simplified User Experience:** Maintain the User frontend (`/`) as a simplified, visually stunning interface focused on a "floating energy" chatbot entity.
3. **Advanced Dev Operations:** Maintain the Dev frontend (`/dev`) as a comprehensive dashboard for tracking agent status, tokens, chat history, and logs without clutter ("active objectives" removed).
4. **Modern Stack:** Utilize Next.js, Tailwind CSS, and Framer Motion for rich interactions and animations.

## Workflow & Constraints
- Always ensure `app/page.tsx` represents the User side and `app/dev/page.tsx` represents the Dev side.
- Use the `generate_image` or WebP generation tools if requested to design new UI mockups before implementing them.
- Avoid modifying backend logic unless necessary for the frontend feature being implemented.
- The User side entity should be designed using purely CSS animations (e.g., `animate-pulse`, `blur`, `spin`) or Framer Motion to resemble an invisible, floating core of energy. Minimal text and no complex dashboards on this route.

## Actions to Take Proactively
- If the user asks you to "improve the frontend" or "make the UI better", first review `frontend/app/globals.css`, layout files, and the current components.
- Suggest high-end aesthetics: glassmorphism, responsive grids, dark modes, and subtle micro-animations.
- Use `tailwind.config.ts` for consistent styling.

## Command Execution
When running Next.js locally, always use:
```bash
npm run dev --prefix frontend
```
