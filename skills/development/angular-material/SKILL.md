---
name: angular-material
description: "ALWAYS use when working with Angular Material components, CDK, or Material Design in Angular applications."
metadata:
  version: 21.0.3
  generated_by: oguzhancart
  generated_at: 2026-02-19
---

# @angular/material & @angular/cdk

**Version:** 21.0.3 (Feb 2026)
**Tags:** Material Design, CDK, Components, UI Library

**References:** [Docs](https://material.angular.dev) — components, guides • [CDK Docs](https://material.angular.dev/cdk) • [GitHub Issues](https://github.com/angular/components/issues) • [Changelog](https://github.com/angular/components/blob/main/CHANGELOG.md)

## API Changes

This section documents recent version-specific API changes.

- NEW: Angular Aria — New low-level component library for accessible, headless components that can be styled custom [source](https://blog.angular.dev/announcing-angular-v21)

- NEW: CDK overlays now use browser's built-in popovers for improved accessibility [source](https://blog.angular.dev/announcing-angular-v21)

- NEW: Material Design system tokens — Use utility classes to apply Material tokens directly in templates [source](https://blog.angular.dev/announcing-angular-v21)

- NEW: CDK Drag & Drop improvements — Allow copying items between lists [source](https://blog.angular.dev/announcing-angular-v21)

- NEW: Angular v21 — Full support for new Angular features including zoneless change detection

## Best Practices

- Use standalone components — Import Material components directly without NgModule

```ts
import { Component } from '@angular/core';
import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';

@Component({
  standalone: true,
  imports: [MatButtonModule, MatCardModule],
  // ...
})
export class ExampleComponent {}
```

- Use CDK for custom components — Use CDK (Component Dev Kit) for behavior primitives like drag-drop, overlays, menus without Material styling

- Use CDK Virtual Scroll for large lists — Improve performance with `cdkVirtualScrollViewport` instead of rendering all items

```html
<cdk-virtual-scroll-viewport itemSize="50" class="viewport">
  <div *cdkVirtualFor="let item of items">{{item.name}}</div>
</cdk-virtual-scroll-viewport>
```

- Use `trackBy` with `ngFor` — Prevent unnecessary DOM re-renders

- Customize themes with SCSS — Use Material's theming system for custom colors and typography

- Use `ChangeDetectionStrategy.OnPush` — Improve performance with default change detection strategy

- Follow accessibility guidelines — Use ARIA labels, keyboard navigation, and focus management provided by CDK components
