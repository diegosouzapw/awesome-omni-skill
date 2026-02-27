---
name: agent-guidelines
description: When you need to understand the project's core mandate, operational rules, or "Constitution". Use this skill to align with the project's identity and strict coding standards.
---

# Agent Guidelines & Operational Rules

## 1. The "Constitution" & Project Identity

**Project Name:** NextBlock CMS
**Mandate:** Build a premium, Open-Core Next.js CMS.
**Core Stack:** Next.js (App Router), Supabase, Tailwind CSS, Tiptap v3.

### Critical Rules

1.  **Strict Separation:** `libs/ui` and `libs/db` must be publishable as standalone packages. They cannot depend on `apps/nextblock`.
2.  **Open-Core Model:** The core is open-source; premium extensions are private.
3.  **Distribution:** Users get a standalone app via `npm create nextblock`.

## 2. Operational Rules (Global Context)

- **Context First:** Before answering complex questions, always check `docs/README.md` and relevant linked docs.
- **Strict Types:** Always use `strict: true` TypeScript. No `any` unless absolutely unavoidable and documented.
- **Target the App, Not the Template:** NEVER edit files in `apps/create-nextblock/templates/nextblock-template` directly. Always make changes in `apps/nextblock` (the core app). The template is synced from the core app via scripts.

## 3. Maintenance Rule: 'Ghost Module Synchronization'

> [!IMPORTANT]
> **CRITICAL:** Whenever you modify the exports of `libs/ecommerce` (the private library), you **MUST** immediately update `tools/stubs/libs/ecommerce/index.ts` to export the same names (as stubs).
>
> **Why?** Failure to do this will break the public open-source build which relies on these stubs when the private library is not present.

## 4. Documentation Access

- Use the **`context7` MCP tool** to fetch the latest documentation for Next.js, Supabase, Nx, Tailwind, etc.
- Do not guess about API updates; verify with `context7` if unsure.
