---
name: chatgpt-app
description: Guidance for building the chatgpt-app (Vite + React + @openai/apps-sdk-ui) with the MCP-friendly single-file output.
---

# ChatGPT App Skill

This skill covers work in `/chatgpt-app`, the Vite + React TypeScript app that ships a single-file (inline HTML/CSS/JS) bundle consumed by the MCP server.

## Tech Stack & Build

- Vite + React 19 + TypeScript.
- Tailwind CSS v4 for utilities; base CSS imports `@openai/apps-sdk-ui/css`.
- Single-file build via `vite-plugin-singlefile` and `html-minifier-terser` (see `package.json` scripts).

## Design System

- All design and components come from **@openai/apps-sdk-ui**.
- Official reference: https://openai.github.io/apps-sdk-ui/?path=/docs/overview-introduction--docs
- Deep dives: prefer the **deepwiki-mcp** (if available) to consult available components, tokens, styles, icons, and colors before guessing.
- Design tokens are exposed in `node_modules/@openai/apps-sdk-ui`; CSS imports use `@source "../node_modules/@openai/apps-sdk-ui";` as in `src/index.css`.

## Icons

- Import icons from the SDK Icon component:
  ```tsx
  import { IconName } from "@openai/apps-sdk-ui/components/Icon";
  ```
- Use SDK-documented icon names; check the docs or deepwiki-mcp for the catalog.

## Styling & Tokens

- Base CSS (`src/index.css`) pulls tokens and fonts:
  ```css
  @import "tailwindcss";
  @import "@openai/apps-sdk-ui/css";
  @source "../node_modules/@openai/apps-sdk-ui";
  :root {
    font-family: var(--font-sans);
    color: var(--color-text);
    background: var(--color-surface);
  }
  ```
- Prefer SDK tokens/variables (`--font-sans`, `--color-text`, `--color-surface`, etc.) and SDK components over custom styles.

## Practices

- Reach for SDK components first; only add bespoke styles when the SDK lacks a suitable primitive.
- Keep the bundle single-file safe: avoid external asset references that won’t inline cleanly.
- When uncertain about component props, tokens, or icons, consult the Apps SDK UI docs and deepwiki-mcp instead of guessing.
