---
name: how-to-build-chatgpt-sidebar
description: Use when asked to build a sidebar experience similar to ChatGPT.com / OpenAI
---

# How to Build a ChatGPT-Style Sidebar

This skill documents the **reusable patterns, DOM structure, layout, animations, and information architecture** reverse-engineered from ChatGPT.com's sidebar. Use this when building a collapsible sidebar that follows the same UX.

Reference screenshots and HTML snapshots live in `references/` — consult `references/README.md` for the full inventory.

---

## Overall Page Layout

The page is a full-viewport flex column. The sidebar and main content sit side-by-side inside a flex row.

```
body
  └── div.flex.h-svh.w-screen.flex-col
        └── div.relative.z-0.flex.min-h-0.w-full.flex-1       ← flex row
              ├── #sidebar-container (shrink-0, border-e)      ← sidebar
              └── @container/main (flex-1, min-w-0)             ← main content
```

Key layout rules:

- The **sidebar is `shrink-0`** so it never compresses; the main content area uses `flex-1 min-w-0` to fill remaining space.
- The sidebar has a right border (`border-e` for logical properties / `border-r` for LTR).
- The sidebar is `h-full` and `overflow-hidden`.
- The sidebar is `z-21` (above main content, below modals/tooltips at z-50).
- The sidebar is **hidden on mobile** via `max-md:hidden` and `print:hidden`.

---

## Sidebar States

### State 1: Desktop Collapsed (Rail)

**Container**: `width: var(--sidebar-rail-width)` (~52px), `background-color: var(--sidebar-bg, var(--bg-primary))`.

The container holds **two overlapping layers** stacked via absolute positioning:

1. **Tiny bar** (`#stage-sidebar-tiny-bar`): `absolute inset-0`, `opacity-100`, interactive.
2. **Expanded panel**: `opacity-0`, `pointer-events-none`, `inert` — present in DOM but invisible/inert.

The **tiny bar** is a vertical flex column:

```
div#stage-sidebar-tiny-bar.flex.h-full.w-[--sidebar-rail-width].flex-col.items-start
  ├── div.h-header-height.flex.items-center.justify-center    ← logo / toggle button
  ├── div.mt-[--sidebar-section-first-margin-top]             ← icon-only nav items
  │     ├── a (New Chat icon)
  │     ├── div (Search icon)
  │     └── a (Library icon)
  ├── div.pointer-events-none.flex-grow                        ← spacer pushes profile down
  └── div.mb-1                                                 ← profile avatar
        └── div.__menu-item[data-size="large"] (profile button)
```

**Icon swap on hover**: The toggle button contains two SVG icons. The logo icon has `group-hover/tiny-bar:hidden` and the panel-toggle icon has `hidden group-hover/tiny-bar:block`. When hovering anywhere on the tiny bar (`group/tiny-bar`), the logo swaps to a panel icon. This is pure CSS, no JS.

### State 2: Desktop Expanded

**Container**: `width: var(--sidebar-width)` (260px), `background-color: var(--sidebar-bg, var(--bg-elevated-secondary))`.

The same two layers exist, but visibility is swapped:

1. **Tiny bar**: `opacity-0`, `pointer-events-none`, `inert`.
2. **Expanded panel**: `opacity-100`, interactive.

The **expanded panel** is a scrollable full nav:

```
div.opacity-100.h-full.w-[--sidebar-width].overflow-y-auto
  ├── h2 (screen-reader only: "Chat history")
  └── nav[aria-label="Chat history"].flex.h-full.flex-col.overflow-y-auto
        ├── div.sticky.top-0.z-30 (sidebar header)
        │     └── div#sidebar-header.h-header-height.flex.items-center.justify-between
        │           ├── a[aria-label="Home"] (logo icon, links to /)
        │           └── div.flex
        │                 └── button[aria-label="Close sidebar"] (toggle)
        ├── aside (action items: New Chat, Search, Library — with text labels)
        ├── [scrollable content sections]
        └── [footer / profile at bottom of scroll]
```

### State 3: Mobile Closed

- **No sidebar container in DOM** — the `#stage-slideover-sidebar` element does not exist at mobile breakpoints.
- The main content area fills the full width.
- The **header** contains a hamburger button:

```
button[data-testid="open-sidebar-button"][aria-controls="stage-popover-sidebar"]
  ├── span.sr-only "Open sidebar"
  └── svg (hamburger icon)
```

### State 4: Mobile Open (Dialog/Popover)

The sidebar renders as a **fixed dialog** with a backdrop overlay:

```
div.fixed.inset-0.z-10.bg-gray-50/50.dark:bg-black/50         ← scrim/backdrop
div#stage-popover-sidebar[role="dialog"].fixed.start-0.top-0.z-50.h-full.w-[--sidebar-width].max-w-xs
  ├── h2 (screen-reader only: "Sidebar")
  ├── h2 (screen-reader only: "Chat history")
  └── nav (same internal structure as desktop expanded panel)
```

Key mobile dialog styles:

- `border-e border-gray-200 dark:border-gray-800`
- `bg-(--sidebar-moweb-bg, var(--sidebar-surface-primary))`
- `shadow-[0_0_64px_0_rgba(0,0,0,0.07)]`
- `pb-[env(safe-area-inset-bottom,0px)]` for iOS safe area
- The close button inside the sidebar `aria-controls="stage-popover-sidebar"`

---

## CSS Custom Properties

```css
:root {
  --sidebar-width: 260px;
  --sidebar-rail-width: calc(13 * var(--spacing)); /* ~52px */
  --header-height: calc(13 * var(--spacing)); /* ~52px */
  --sidebar-section-margin-top: 1.25rem;
  --sidebar-section-first-margin-top: 0.5rem;
  --sidebar-expanded-section-margin-bottom: 1.25rem;

  /* Light mode */
  --sidebar-bg: #ffffff29;
  --sidebar-mask-bg: transparent;
  --sidebar-surface-primary: var(--gray-50);
  --sidebar-surface-secondary: var(--gray-100);
}

/* Touch devices get larger targets */
@media (pointer: coarse) {
  :root {
    --sidebar-rail-width: calc(14 * var(--spacing)); /* ~56px */
    --header-height: calc(14 * var(--spacing)); /* ~56px */
  }
}

/* Dark mode */
.dark {
  --sidebar-bg: #00000029;
  --sidebar-surface-primary: var(--gray-900);
  --sidebar-surface-secondary: var(--gray-800);
}
```

---

## Open/Close Animation

The sidebar does **NOT** animate width. Instead, it uses a **cross-fade between two overlapping layers**:

1. The container `width` changes instantly via CSS variable (`--sidebar-rail-width` ↔ `--sidebar-width`).
2. The **tiny bar** and **expanded panel** cross-fade using opacity transitions:
   - `motion-safe:transition-opacity motion-safe:duration-150 motion-safe:ease-linear`
3. The tiny bar uses a stepped easing: `motion-safe:ease-[steps(1,start)]` — it snaps instantly rather than fading, preventing a ghostly half-state.
4. The non-active layer gets `pointer-events-none` and the `inert` HTML attribute to prevent focus and interaction.

This approach avoids layout shift during animation and is significantly simpler than animating width.

---

## Menu Item System (`.__menu-item`)

This is the core primitive for all sidebar items: nav actions, chat history entries, profile button.

### Base styles

```css
.__menu-item {
  min-height: var(--menu-item-height);
  padding-inline: calc(var(--spacing) * 2.5);
  padding-block: calc(var(--spacing) * 1.5);
  font-size: var(--text-sm);
  border-radius: 10px;
  display: flex;
  align-items: center;
  cursor: pointer;
  user-select: none;
  margin-inline: calc(var(--spacing) * 1.5); /* gutter from edges */
}

.__menu-item[data-fill] {
  max-width: calc(100% - 3 * var(--spacing)); /* constrained width */
}

.__menu-item[data-size="large"] {
  min-height: calc(var(--spacing) * 10); /* profile button */
}
```

### Hover/active states

```css
.__menu-item {
  --menu-item-highlighted: var(--interactive-bg-secondary-hover);
  --menu-item-active: var(--interactive-bg-secondary-press);
}

.__menu-item.hoverable:hover {
  background-color: var(--menu-item-highlighted);
}

.__menu-item:active:not(:has([data-trailing-button]:hover)) {
  background-color: var(--menu-item-active);
}
```

### Trailing action pattern

Menu items use a **grid overlay** to show/hide trailing action buttons on hover:

```html
<a class="group __menu-item hoverable" data-fill>
  <div class="flex min-w-0 grow items-center gap-2.5">
    <div class="truncate"><span>Chat Title</span></div>
  </div>
  <div class="trailing-pair">
    <div class="trailing highlight">
      <!-- Visible on hover: options button -->
      <button class="__menu-item-trailing-btn">...</button>
    </div>
    <div class="trailing">
      <!-- Default trailing content (hidden on hover) -->
    </div>
  </div>
</a>
```

CSS mechanics:

- `.trailing-pair` uses `display: inline-grid` with both children in the same grid cell (overlap).
- `.trailing.highlight` starts at `opacity: 0` (or `clip-path: inset(50%)` for `[data-fill]` items).
- On hover/focus/open: `.trailing.highlight` becomes visible, `.trailing:not(.highlight)` hides.
- The trailing button itself uses negative margins to extend clickable area to the menu item edge.

---

## Sidebar Content Sections

### Action Items (sticky section)

Pinned at the top of the scrollable area using a sticky `<aside>`:

```
aside.pt-[--sidebar-section-first-margin-top].tall:sticky.tall:top-header-height.tall:z-20
  ├── New chat (a.__menu-item with icon + label + keyboard shortcut)
  ├── Search chats (div.__menu-item with icon + label + keyboard shortcut)
  └── Library/Images (a.__menu-item with icon + label)
```

### Chat History

A flat container of conversation links — no `<ol>/<li>` structure:

```
div#history
  ├── a.__menu-item.hoverable[draggable][href="/c/..."]
  ├── a.__menu-item.hoverable[draggable][href="/c/..."]
  └── ...
```

Each conversation item has:

- `data-sidebar-item="true"` for sidebar-specific behavior
- `draggable="true"` for drag-and-drop reordering
- A truncated title with `dir="auto"` for bidirectional text
- A `.trailing-pair` with an options menu button (revealed on hover)

### Profile (bottom)

Located in the tiny bar at the bottom via `flex-grow` spacer:

```
div.mb-1
  └── div.__menu-item[data-size="large"][data-testid="accounts-profile-button"]
        └── div.icon-lg
              └── div.rounded-full.h-6.w-6 > img (avatar)
```

In expanded mode, the profile shows next to the avatar: name + subscription tier.

---

## Tooltip Pattern

Toggle buttons use Radix UI tooltips:

```html
<span data-state="instant-open|delayed-open|closed">
  <button aria-label="Open sidebar" aria-describedby="radix-...">
    <!-- icon -->
  </button>
</span>

<!-- Portal-rendered tooltip -->
<div
  data-radix-popper-content-wrapper
  style="position: fixed; transform: translate(Xpx, Ypx); z-index: 50;"
>
  <div
    data-side="right|bottom"
    data-align="center"
    class="dark bg-black rounded-lg px-2 py-1 max-w-xs"
  >
    <div class="text-xs font-semibold text-center">Open sidebar</div>
  </div>
</div>
```

Tooltip positioning:

- Rail button tooltip: `data-side="right"` (appears to the right of the rail)
- Expanded close button tooltip: `data-side="bottom"` (appears below the button)

---

## Responsive Strategy

| Breakpoint     | Sidebar Behavior                                        | Toggle Mechanism                                          |
| -------------- | ------------------------------------------------------- | --------------------------------------------------------- |
| `md` and above | Inline rail/expanded panel (`#stage-slideover-sidebar`) | Logo icon click (rail→expand), close button (expand→rail) |
| Below `md`     | Dialog/popover (`#stage-popover-sidebar`) with backdrop | Hamburger button in header opens, X/scrim closes          |

Key responsive classes:

- `max-md:hidden` — hides the inline sidebar on mobile
- `md:hidden` — hides mobile-only elements on desktop
- `touch:h-10 touch:w-10` — larger touch targets (44px) on coarse pointer devices
- `touch:px-1.5` — adjusted padding for touch

---

## Accessibility

- `<nav aria-label="Chat history">` wraps the sidebar content
- `<h2>` (visually hidden) provides heading structure for screen readers
- Toggle buttons use `aria-expanded`, `aria-controls`, and `aria-label`
- Mobile sidebar uses `role="dialog"` with `aria-labelledby`/`aria-describedby`
- `sr-only` / clip-rect pattern for screen-reader-only text
- `keyboard-focused:bg-token-surface-hover` for keyboard-specific focus styles (separate from mouse hover)
- RTL support: `data-rtl-flip` on directional icons, `rtl:cursor-w-resize`, `:dir(ltr)/:dir(rtl)` selectors
- `inert` attribute on hidden layers to prevent screen reader traversal

---

## Key Reusable Patterns (Summary)

1. **Rail + Expand via opacity cross-fade**: Two absolutely-positioned layers inside a container whose width changes via CSS variable. No width animation — just opacity transition + `inert`.

2. **Icon swap on hover**: Use Tailwind `group-hover/{name}:hidden` / `group-hover/{name}:block` on two sibling SVGs to swap icons (e.g., logo → panel toggle) without JS.

3. **Menu item primitive**: A flex row with `border-radius: 10px`, hover highlight, and a trailing-pair grid for overlay action buttons that appear on hover.

4. **Mobile dialog + backdrop**: At mobile breakpoints, render sidebar as a `position: fixed` dialog with `role="dialog"`, a semi-transparent backdrop (`bg-gray-50/50`), and safe-area inset padding.

5. **Sticky action section**: Use `<aside>` with `sticky top-[header-height]` for pinned action items (New Chat, Search) above the scrollable chat history.

6. **Touch-adaptive sizing**: Use `touch:` variant (pointer: coarse media query) to increase hit targets from 36px to 40px.

7. **`pointer-events-none` + `inert`**: Always pair these on hidden layers to prevent both mouse and keyboard/screen-reader interaction.
