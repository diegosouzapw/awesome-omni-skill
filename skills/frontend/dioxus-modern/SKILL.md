---
name: dioxus-modern
description: Modern Dioxus 0.7 + Tailwind CSS development guide. Use when asking about Signals, Stores, ReadSignal props, async patterns, components, or Tailwind integration in Dioxus 0.7.
argument-hint: [dioxus question, component pattern, state management, tailwind setup, async data fetching]
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
model: sonnet
user-invocable: true
version: 1.9.0
---

```jsonl
{"kind":"meta","skill":"dioxus-modern","version":"1.9.0","format":"markdown-with-embedded-jsonl","mode":"contract-first"}
{"kind":"input","arguments":"$ARGUMENTS","rule":"Infer target from current request context. Trigger on dioxus, signals, stores, rsx, tailwind, fullstack, router."}
{"kind":"mission","goal":"Guide developers in writing modern, AI-friendly Dioxus 0.7 applications using Signals, Stores, and fine-grained reactivity."}
{"kind":"rule","id":"state_management","text":"Use use_signal for atomic values, use_store with #[derive(Store)] for collections/nested state. Never use use_state in 0.7."}
{"kind":"rule","id":"props_pattern","text":"Use ReadSignal<T> for component props that receive reactive values. Accepts Signal, Memo, Resource, or primitives with auto-conversion."}
{"kind":"rule","id":"component_pattern","text":"Always use #[component] macro. Components are memoized by default and only re-render when Props change via PartialEq."}
{"kind":"rule","id":"tailwind_integration","text":"Dioxus 0.7 has automatic Tailwind integration. Use asset!(\"/assets/tailwind.css\") via document::Stylesheet and run dx serve."}
{"kind":"rule","id":"async_patterns","text":"Use use_resource for async state. Use use_server_future for hydration-safe data. Avoid waterfalls by starting all requests first."}
{"kind":"workflow","id":"dioxus_dev","steps":["Identify state container (Signal vs Store)","Define props using ReadSignal","Implement UI with rsx! macro","Style with Tailwind class strings","Add async data with use_resource"]}
{"kind":"cmd","group":"cli","commands":{"dx new":"Create new project","dx serve":"Start dev server with hot-reload","dx bundle":"Build for production","dx translate":"Convert HTML to RSX"}}
{"kind":"ref","file":"reference.md","use":"API signatures, type system, and core mechanics"}
{"kind":"ref","file":"fullstack.md","use":"Server functions, streaming, and fullstack patterns"}
{"kind":"ref","file":"templates.md","use":"Ready-to-use boilerplate for common UI and logic"}
{"kind":"ref","file":"examples.md","use":"Good vs Bad comparisons and best practices"}
{"kind":"ref","file":"migration.md","use":"Upgrading from 0.5/0.6 to 0.7"}
```

# Modern Dioxus 0.7 Guide

Dioxus 0.7 is a massive leap forward in Rust UI development, focusing on **Copy signals**, **fine-grained reactivity**, and **automatic asset handling**.

## Core Architecture

1.  **Reactivity**: Built on Signals and Stores. State is `Copy + Send + Sync`.
2.  **Fullstack**: Unified server/client code with `#[server]` functions and `use_server_future`.
3.  **Styling**: Native Tailwind CSS integration using the `dx` CLI.
4.  **Performance**: Sub-second hot-patching and WASM bundle splitting.

## Progressive Disclosure

For deep-dives into specific areas, consult the following support docs:

-   **[Reference](reference.md)**: Every hook, type, and trait you need to know.
-   **[Fullstack](fullstack.md)**: Server-side rendering, streaming, and real-time.
-   **[Templates](templates.md)**: Copy-paste code for lists, auth, websockets, and more.
-   **[Examples](examples.md)**: Learn by seeing Good vs. Bad patterns.
-   **[Migration](migration.md)**: Quick guide for moving from older versions.

## Quick CLI Cheat Sheet

```bash
# Start developing
dx serve

# Add Tailwind
# 1. Create tailwind.css with @import "tailwindcss"
# 2. Add asset!("/assets/tailwind.css") to root rsx!

# Convert HTML snippet to RSX
dx translate --raw "<div class='foo'>...</div>"
```
