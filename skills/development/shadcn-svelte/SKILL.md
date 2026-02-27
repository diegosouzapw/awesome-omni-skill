---
name: shadcn-svelte
description: shadcn-svelte component library - install Button, Card, Dialog, Form, Table components with shadcn design tokens. Use for pre-built accessible UI components. For raw Tailwind CSS use tailwind skill. For UX design principles use frontend-design skill.
license: MIT
compatibility: opencode
metadata:
  author: OpenCode Community
  version: "2.0"
  source: https://github.com/huntabyte/shadcn-svelte
---

# shadcn-svelte

shadcn-svelte is a Svelte port of shadcn-ui - accessible UI components built with Tailwind CSS and Bits UI.

## When to Use

- Installing shadcn-svelte components (Button, Dialog, Card, etc.)
- Setting up theming with Tailwind CSS v4
- Configuring dark mode with mode-watcher
- Building accessible forms with Bits UI
- Troubleshooting component issues

## Quick Reference

```bash
# Install shadcn-svelte
bunx shadcn-svelte@latest init

# Add components
bunx shadcn-svelte@latest add button
bunx shadcn-svelte@latest add dialog
bunx shadcn-svelte@latest add card

# Add form components
bunx shadcn-svelte@latest add form
bunx shadcn-svelte@latest add input
bunx shadcn-svelte@latest add select
```

```svelte
<script lang="ts">
  import { Button } from "$lib/components/ui/button";
  import * as Card from "$lib/components/ui/card";
</script>

<Card.Root>
  <Card.Header>
    <Card.Title>Title</Card.Title>
    <Card.Description>Description</Card.Description>
  </Card.Header>
  <Card.Content>
    Content here
  </Card.Content>
  <Card.Footer>
    <Button>Click me</Button>
  </Card.Footer>
</Card.Root>
```

## Key Concepts

- **Components** - Installed to `$lib/components/ui/`
- **Bits UI** - Headless primitives for accessibility
- **Tailwind v4** - Styling with CSS-first config
- **Dark Mode** - Via `mode-watcher` package

## References

- [Design Tokens](references/design-tokens.md) - **CRITICAL: Read before styling**
- [Installation](references/documentation/Installation_e2993602e4.md)
- [Dark Mode Setup](references/documentation/Dark_Mode_ed9cf28a2a.md)
- [Component Categories](references/registry.json)
- [Troubleshooting](references/documentation/FAQ_18bce2f4b4.md)
- [Form Components](references/documentation/Formsnap_7e47279aa6.md)

## Related Skills

- tailwind - Styling
- svelte-kit - Framework
