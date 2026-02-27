---
name: tailwind-ecommerce
description: Use when building ecommerce interfaces, product pages, shopping carts, checkout flows, order management, or store navigation with Tailwind CSS. Covers product cards, product lists, product detail pages, category filters, shopping cart, checkout forms, order summaries, order history, pricing displays, promo sections, reviews, ratings, store navigation, category previews, inventory badges, wishlist buttons, compare features.
---

# Ecommerce UI with Tailwind CSS

Patterns for product display, cart, checkout, orders, and store navigation.

---

## 1. Product Display Patterns

### Card Variants

| Variant | Key Classes | Use Case |
|---------|-------------|----------|
| Simple grid card | `aspect-[3/4] w-full object-cover` + text below image | Category pages, minimal listings |
| Card with details | `rounded-lg border bg-white shadow-sm` + price, rating, badge | Product listings, search results |
| Horizontal card | `flex flex-row gap-4` + image left, details right | Cart items, wishlists, comparisons |
| Quick view overlay | `group relative` + `opacity-0 group-hover:opacity-100` overlay CTA | Browse-and-add flows |
| Featured card | `col-span-2 row-span-2` + larger image | Homepage hero products |

### Image Handling

| Concern | Pattern | Classes |
|---------|---------|---------|
| Aspect ratio | Fixed ratio container | `aspect-square`, `aspect-[3/4]`, `aspect-[4/3]` |
| Fill behavior | Cover without distortion | `object-cover w-full h-full` |
| Responsive sizing | Fluid width, fixed aspect | `w-full` on container, aspect ratio on `<img>` |
| Placeholder | Background color fallback | `bg-gray-100` on container |
| Hover zoom | Scale on group hover | `transition-transform duration-300 group-hover:scale-105` |
| Lazy loading | Native attribute | `loading="lazy"` on `<img>` |

### Price Display

| Scenario | Pattern |
|----------|---------|
| Regular price | `<span class="text-lg font-semibold text-gray-900">$49.99</span>` |
| Sale price | Original: `line-through text-gray-500 text-sm` / Sale: `text-lg font-semibold text-red-600` |
| Price range | `From $29.99` with `text-sm text-gray-500` prefix |
| Currency symbol | Keep symbol adjacent, no wrapping: `whitespace-nowrap` |

### Rating Display

| Element | Classes |
|---------|---------|
| Star icon (filled) | `size-4 text-yellow-400 fill-current` |
| Star icon (empty) | `size-4 text-gray-300 fill-current` |
| Half star | Clip filled star at 50% with `overflow-hidden w-2` wrapper |
| Review count | `text-sm text-gray-500 ml-1` next to stars |
| Container | `flex items-center gap-0.5` |

---

## 2. Product List Layouts

### Responsive Grid

```html
<div class="grid grid-cols-1 gap-6 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
  <!-- Product cards -->
</div>
```

### Layout Patterns

| Layout | Classes | When to Use |
|--------|---------|-------------|
| Grid (default) | `grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6` | Category pages, search results |
| Two-column wide | `grid grid-cols-1 md:grid-cols-2 gap-8` | Featured products, comparisons |
| List view | `flex flex-col divide-y` with horizontal cards | Detail-heavy browsing |
| Masonry-style | `columns-2 lg:columns-3 gap-4` | Visual-heavy catalogs |

### Loading and Pagination

| Pattern | Implementation |
|---------|---------------|
| Skeleton card | `animate-pulse` with `bg-gray-200 rounded` blocks matching card layout |
| Skeleton image | `aspect-[3/4] bg-gray-200 rounded-lg animate-pulse` |
| Skeleton text | `h-4 bg-gray-200 rounded w-3/4 animate-pulse` |
| Load more button | `w-full py-3 border rounded-lg text-center` below grid |
| Pagination | `flex items-center gap-2` with numbered buttons, active: `bg-indigo-600 text-white` |
| Infinite scroll | Intersection observer on sentinel div at grid end |
| Empty state | `text-center py-16` with icon, heading, and CTA |

---

## 3. Category and Filtering

### Sidebar Filter Panel (Desktop)

| Element | Classes |
|---------|---------|
| Panel container | `w-64 shrink-0 hidden lg:block` |
| Filter group | `border-b py-4` with heading + options |
| Group heading | `font-medium text-sm text-gray-900 mb-3` |
| Checkbox + label | `flex items-center gap-2 text-sm text-gray-600` |
| Color swatch | `size-6 rounded-full border-2` + `ring-2 ring-offset-2 ring-indigo-500` when active |
| Price range | Two `input[type=number]` with `w-20` in a `flex gap-2 items-center` row |
| Apply button | `w-full mt-3 py-2 bg-indigo-600 text-white rounded-lg text-sm` |

### Mobile Filter (Slide-out Dialog)

| Element | Classes |
|---------|---------|
| Trigger button | `lg:hidden flex items-center gap-2 border rounded-lg px-4 py-2` |
| Backdrop | `fixed inset-0 bg-black/25 z-40` |
| Panel | `fixed inset-y-0 right-0 w-full max-w-sm bg-white z-50 overflow-y-auto` |
| Header | `flex items-center justify-between px-4 py-3 border-b` |
| Footer | `sticky bottom-0 bg-white border-t px-4 py-3` with Apply/Clear buttons |

### Active Filters and Sorting

| Element | Classes |
|---------|---------|
| Filter chip | `inline-flex items-center gap-1 rounded-full bg-gray-100 px-3 py-1 text-sm` |
| Remove button | `size-4 text-gray-400 hover:text-gray-600` (X icon inside chip) |
| Clear all link | `text-sm text-indigo-600 hover:text-indigo-800` |
| Sort dropdown | `select` or custom dropdown, `text-sm border rounded-lg px-3 py-2` |
| Result count | `text-sm text-gray-500` (e.g., "128 products") |
| Filter count badge | `size-5 rounded-full bg-indigo-600 text-white text-xs flex items-center justify-center` |

---

## 4. Shopping Cart

### Cart Item Row

```html
<div class="flex gap-4 py-4 border-b">
  <img class="size-20 rounded-lg object-cover shrink-0" src="..." alt="..." />
  <div class="flex flex-1 flex-col justify-between">
    <div class="flex justify-between">
      <div>
        <h3 class="text-sm font-medium text-gray-900">Product Name</h3>
        <p class="mt-1 text-sm text-gray-500">Color / Size</p>
      </div>
      <p class="text-sm font-medium text-gray-900">$49.99</p>
    </div>
    <div class="flex items-center justify-between">
      <!-- Quantity selector -->
      <div class="flex items-center rounded-lg border">
        <button class="px-3 py-1 text-gray-600">-</button>
        <span class="px-3 py-1 border-x text-sm">1</span>
        <button class="px-3 py-1 text-gray-600">+</button>
      </div>
      <button class="text-sm text-red-600 hover:text-red-800">Remove</button>
    </div>
  </div>
</div>
```

### Cart Sections

| Section | Classes | Content |
|---------|---------|---------|
| Cart list | `divide-y` | Cart item rows |
| Summary panel | `rounded-lg bg-gray-50 p-6 lg:w-80 lg:shrink-0` | Subtotal, shipping, tax, total |
| Summary row | `flex justify-between text-sm text-gray-600 py-2` | Label + amount |
| Total row | `flex justify-between text-base font-semibold text-gray-900 border-t pt-4 mt-4` | Total label + amount |
| Checkout CTA | `w-full mt-6 py-3 bg-indigo-600 text-white rounded-lg font-medium` | "Proceed to Checkout" |
| Continue shopping | `text-sm text-indigo-600 hover:text-indigo-800 mt-4 text-center block` | Link back |

### Cart States

| State | Pattern |
|-------|---------|
| Empty cart | `text-center py-16` with cart icon, "Your cart is empty" heading, shop CTA |
| Mini cart / drawer | `fixed inset-y-0 right-0 w-full max-w-md bg-white shadow-xl z-50` |
| Cart badge | `absolute -top-2 -right-2 size-5 rounded-full bg-indigo-600 text-white text-xs flex items-center justify-center` |
| Updating state | `opacity-50 pointer-events-none` on item row during async update |

---

## 5. Checkout Flow

### Progress Indicator

| Step State | Classes |
|------------|---------|
| Completed | `size-8 rounded-full bg-indigo-600 text-white` with check icon |
| Current | `size-8 rounded-full border-2 border-indigo-600 text-indigo-600` |
| Upcoming | `size-8 rounded-full border-2 border-gray-300 text-gray-400` |
| Connector (done) | `h-0.5 w-full bg-indigo-600` between circles |
| Connector (pending) | `h-0.5 w-full bg-gray-300` between circles |
| Container | `flex items-center justify-between max-w-lg mx-auto mb-8` |

### Form Layout

| Section | Structure |
|---------|-----------|
| Two-column layout | `grid grid-cols-1 lg:grid-cols-5 gap-8` (3 cols form + 2 cols summary) |
| Field group | `space-y-4` container |
| Two-field row | `grid grid-cols-2 gap-4` (first/last name, city/zip) |
| Full-width field | `col-span-2` or standalone |
| Label | `block text-sm font-medium text-gray-700 mb-1` |
| Input | `w-full rounded-lg border border-gray-300 px-3 py-2 text-sm focus:border-indigo-500 focus:ring-1 focus:ring-indigo-500` |
| Error input | `border-red-500 focus:border-red-500 focus:ring-red-500` |
| Error message | `text-sm text-red-600 mt-1` |

### Payment Fields

| Element | Pattern |
|---------|---------|
| Card number | Full-width input with card icon right-aligned |
| Expiry + CVC | `grid grid-cols-2 gap-4` side by side |
| Card group | `rounded-lg border divide-y` wrapping all card fields as one visual unit |
| Saved cards | Radio group: `flex items-center gap-3 p-4 border rounded-lg` per card |
| Security note | `flex items-center gap-2 text-sm text-gray-500` with lock icon |

### Checkout Patterns

| Pattern | Implementation |
|---------|---------------|
| Guest vs account | Toggle or tabs at top: `flex rounded-lg border` with two options |
| Order review | Summary card: item thumbnails, quantities, line totals in `bg-gray-50 rounded-lg p-6` |
| Place order CTA | `w-full py-3 bg-indigo-600 text-white rounded-lg font-medium text-lg` |
| Terms checkbox | `flex items-start gap-2` with checkbox and label text |
| Loading on submit | Button with `opacity-75 cursor-not-allowed` + spinner icon |

---

## 6. Order Management

### Order History Table

| Viewport | Pattern |
|----------|---------|
| Desktop | Standard `<table>` with columns: Order #, Date, Status, Total, Actions |
| Mobile | Stacked cards: `divide-y` with each order as a `flex flex-col gap-2 py-4` block |
| Table header | `text-left text-sm font-medium text-gray-500 pb-3 border-b` |
| Table cell | `py-4 text-sm text-gray-900` |

### Order Status Badges

| Status | Classes |
|--------|---------|
| Pending | `bg-yellow-100 text-yellow-800 rounded-full px-2.5 py-0.5 text-xs font-medium` |
| Processing | `bg-blue-100 text-blue-800 rounded-full px-2.5 py-0.5 text-xs font-medium` |
| Shipped | `bg-indigo-100 text-indigo-800 rounded-full px-2.5 py-0.5 text-xs font-medium` |
| Delivered | `bg-green-100 text-green-800 rounded-full px-2.5 py-0.5 text-xs font-medium` |
| Cancelled | `bg-red-100 text-red-800 rounded-full px-2.5 py-0.5 text-xs font-medium` |
| Refunded | `bg-gray-100 text-gray-800 rounded-full px-2.5 py-0.5 text-xs font-medium` |

### Order Detail Page

| Section | Pattern |
|---------|---------|
| Status timeline | Vertical: `relative pl-8 space-y-6` with `absolute left-0` circles and connecting line |
| Timeline dot (done) | `size-4 rounded-full bg-indigo-600` |
| Timeline dot (current) | `size-4 rounded-full border-2 border-indigo-600 bg-white` |
| Timeline line | `absolute left-2 top-4 bottom-0 w-0.5 bg-gray-200` |
| Items list | Horizontal cards with image, name, qty, price |
| Totals section | Same pattern as cart summary |
| Shipping address | `bg-gray-50 rounded-lg p-4` card |
| Invoice layout | `max-w-2xl mx-auto` with logo header, address block, items table, totals |

---

## 7. Pricing and Promotions

### Price Patterns

| Pattern | Implementation |
|---------|---------------|
| Original + sale | `<span class="line-through text-gray-500 text-sm">$99.99</span> <span class="text-lg font-semibold text-red-600">$49.99</span>` |
| Save amount | `<span class="text-sm text-green-600 font-medium">Save $50.00 (50%)</span>` |
| Per-unit price | `<span class="text-xs text-gray-500">$2.50/oz</span>` below main price |

### Discount Badge (on Product Image)

```html
<div class="relative">
  <img class="aspect-[3/4] w-full rounded-lg object-cover" src="..." alt="..." />
  <span class="absolute top-2 left-2 rounded-full bg-red-600 px-2.5 py-0.5 text-xs font-semibold text-white">
    -50%
  </span>
</div>
```

### Pricing Tier Cards

| Element | Classes |
|---------|---------|
| Card container | `rounded-xl border p-6 text-center` |
| Featured card | `border-indigo-600 ring-2 ring-indigo-600 relative` with "Popular" badge |
| Price | `text-4xl font-bold text-gray-900` |
| Period | `text-sm text-gray-500` (e.g., "/month") |
| Feature list | `space-y-3 text-sm text-gray-600` with check icons |
| CTA button | `w-full mt-6 py-2.5 rounded-lg font-medium` |

### Promo and Incentive Sections

| Pattern | Classes |
|---------|---------|
| Full-width banner | `bg-indigo-600 text-white py-3 text-center text-sm font-medium` |
| Dismissable banner | Add `flex items-center justify-between px-4` with close button |
| Trust icons row | `grid grid-cols-2 md:grid-cols-4 gap-6 py-8 text-center` |
| Trust icon item | `flex flex-col items-center gap-2 text-sm text-gray-600` with icon above |
| Coupon input | `flex` with `flex-1 rounded-l-lg border px-3 py-2` input + `rounded-r-lg bg-gray-900 text-white px-4 py-2` button |

---

## 8. Store Navigation

### Store Header

| Element | Classes |
|---------|---------|
| Header bar | `sticky top-0 z-30 bg-white border-b` |
| Inner layout | `flex items-center justify-between h-16 px-4 max-w-7xl mx-auto` |
| Logo | `shrink-0` |
| Search bar | `flex-1 max-w-lg mx-8 hidden md:block` with `rounded-full bg-gray-100 px-4 py-2` |
| Icon group | `flex items-center gap-4` for account, wishlist, cart icons |
| Cart icon + badge | `relative` button with `absolute -top-1.5 -right-1.5` badge |

### Mega Menu

| Element | Classes |
|---------|---------|
| Trigger | `group` on nav item |
| Dropdown | `invisible group-hover:visible opacity-0 group-hover:opacity-100 transition-all absolute left-0 w-full bg-white shadow-lg border-t` |
| Inner grid | `grid grid-cols-4 gap-8 max-w-7xl mx-auto p-8` |
| Column heading | `font-medium text-sm text-gray-900 mb-3` |
| Column links | `space-y-2 text-sm text-gray-600` |
| Featured image | `rounded-lg overflow-hidden` in last column |

### Category Previews and Breadcrumbs

| Element | Classes |
|---------|---------|
| Category card | `group relative rounded-xl overflow-hidden` |
| Category image | `aspect-[4/3] w-full object-cover transition-transform group-hover:scale-105` |
| Category overlay | `absolute inset-0 bg-gradient-to-t from-black/60 to-transparent` |
| Category label | `absolute bottom-4 left-4 text-lg font-semibold text-white` |
| Breadcrumb nav | `flex items-center gap-2 text-sm text-gray-500` |
| Breadcrumb separator | `size-4 text-gray-400` (chevron icon) |
| Breadcrumb current | `text-gray-900 font-medium` (last item, no link) |

---

## 9. Accessibility for Ecommerce

| Element | Requirement | Implementation |
|---------|-------------|----------------|
| Product image | Descriptive alt text | `alt="Red leather crossbody bag"` not `alt="product"` |
| Decorative image | Skip for screen readers | `alt=""` and `aria-hidden="true"` |
| Sale price | Announce was/now | `aria-label="Sale price $49.99, originally $99.99"` on container |
| Cart update | Announce change | `aria-live="polite"` region for "Item added" / "Cart updated" messages |
| Cart count badge | Announce count | `aria-label="Cart, 3 items"` on cart button |
| Quantity selector | Label buttons | `aria-label="Decrease quantity"` / `aria-label="Increase quantity"` |
| Star rating | Text alternative | `aria-label="4 out of 5 stars"` on rating container |
| Filter change | Announce results | `aria-live="polite"` on product count ("128 results") |
| Form errors | Link to field | `aria-describedby="email-error"` on input, matching `id` on error text |
| Checkout steps | Current step | `aria-current="step"` on active step indicator |
| Remove from cart | Confirm action | Identify item in label: `aria-label="Remove Red Bag from cart"` |
| Sort dropdown | Label purpose | `aria-label="Sort products by"` on select element |
| Color swatch | Name the color | `aria-label="Color: Navy Blue"` on swatch button |
| Dialog filters | Trap focus | `role="dialog"` + `aria-modal="true"` + focus trap on mobile filter panel |

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Fixed pixel image heights | Use `aspect-*` utilities with `object-cover` for responsive images |
| Cart with no empty state | Always handle zero items with illustration and CTA |
| Price without `whitespace-nowrap` | Currency and amount should not wrap to separate lines |
| Filter panel always visible on mobile | Hide with `hidden lg:block`, use dialog for mobile |
| No loading state during cart updates | Show `opacity-50 pointer-events-none` or skeleton on async actions |
| Checkout form in single column | Use `grid grid-cols-2 gap-4` for name, city/zip, expiry/CVC pairs |
| Status badges with only color | Add text labels; color alone fails WCAG |
| Missing `aria-live` on cart | Cart count and summary changes must announce to screen readers |
| Mega menu not keyboard-navigable | Ensure focus management and Escape key closes the menu |
| Discount badge over product text | Position badge on image (`absolute top-2 left-2`), not over card text |

## MCP Component Library

The `frontend-components` MCP server provides ecommerce-related components:

- **HyperUI** (`hyperui`): Product cards, product collections, pricing tables, forms, badges, stats
- **DaisyUI** (`daisyui`): Card, badge, stat, table, modal, rating, countdown class references
- **FlyonUI** (`flyonui`): Card, badge, stat, table CSS components + datatable JS plugin

**Ecommerce search:** `search_components(query: "product", framework: "hyperui")` or `search_components(query: "pricing")` for pricing components across all libraries.
