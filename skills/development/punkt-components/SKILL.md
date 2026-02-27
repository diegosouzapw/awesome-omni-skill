---
name: punkt-components
description: Using Punkt design system components (React and Elements/Web Components). Covers component APIs, props, events, slots, and usage patterns for all Punkt UI components. Use when building UIs with Punkt components.
---

# Punkt Components

Skill for using Punkt design system components. Covers both React (`@oslokommune/punkt-react`) and Elements/Web Components (`@oslokommune/punkt-elements`).

For CSS classes, colors, typography, spacing, grid, and layout, see the **punkt-css** skill.

## Getting started

All usage patterns require Punkt CSS. See the punkt-css skill for setup instructions.

### React (NPM)

```sh
npm add @oslokommune/punkt-react
```

```jsx
import { PktButton } from '@oslokommune/punkt-react';

<PktButton skin="primary" variant="icon-left" iconName="user">
  Click me
</PktButton>
```

Punkt components load icons, SVGs, and other resources from the CDN. If your project uses a Content Security Policy (CSP), you must configure it to allow resources from `https://punkt-cdn.oslo.kommune.no/`. See the [CSP section](#content-security-policy-csp) below.

If you have unit tests, components that wrap Elements may need special setup — see individual component docs.

### Elements (NPM)

```sh
npm add @oslokommune/punkt-elements
```

```js
import '@oslokommune/punkt-elements/dist/pkt-button.js';
```

```html
<pkt-button skin="primary">
  <span>Click me</span>
</pkt-button>
```

For reactive slot content (content that changes programmatically), wrap it in a container element like `<span>` or `<div>`.

### Elements (CDN)

No build step required. Include CSS and component scripts directly:

```html
<link href="https://punkt-cdn.oslo.kommune.no/latest/css/pkt.min.css" rel="stylesheet" />
<script src="https://punkt-cdn.oslo.kommune.no/latest/elements/pkt-button.js" type="module"></script>
```

If using CSP, see the [CSP section](#content-security-policy-csp) below.

## Content Security Policy (CSP)

Punkt components load fonts, icons (SVG), and other resources from `https://punkt-cdn.oslo.kommune.no/`. If the application uses a Content Security Policy, the CSP must allow this origin. This applies to all setup methods (NPM and CDN).

**Required CSP directives:**

```
Content-Security-Policy:
  default-src 'self';
  font-src 'self' https://punkt-cdn.oslo.kommune.no/;
  img-src 'self' https://punkt-cdn.oslo.kommune.no/;
  script-src 'self' https://punkt-cdn.oslo.kommune.no/;
  style-src 'self' 'unsafe-inline' https://punkt-cdn.oslo.kommune.no/;
  connect-src 'self' https://punkt-cdn.oslo.kommune.no/;
```

**Why `'unsafe-inline'` in `style-src`?** Because of the intricacies of SVG styling and certain web component features, Punkt requires `'unsafe-inline'` in the `style-src` directive. Without it, some styles will be blocked by the browser. CSP nonces can be used as an alternative but require server-side configuration.

> **Important:** Always instruct users to configure CSP when setting up Punkt components. Missing CSP configuration is a common cause of broken fonts, missing icons, and invisible components.

## Components

1. [Accordion](accordion.md)
2. [Alert](alert.md)
3. [Backlink](backlink.md)
4. [Breadcrumbs](breadcrumbs.md)
5. [Button](button.md)
6. [Card](card.md)
7. [Checkbox](checkbox.md)
8. [Combobox](combobox.md)
9. [Consent](consent.md)
10. [Datepicker](datepicker.md)
11. [Footer](footer.md)
12. [Header](header.md)
13. [Icon](icon.md)
14. [Input Wrapper](input-wrapper.md)
15. [Link](link.md)
16. [LinkCard](linkcard.md)
17. [Loader](loader.md)
18. [Messagebox](messagebox.md)
19. [Modal](modal.md)
20. [Progressbar](progressbar.md)
21. [Radio Button](radiobutton.md)
22. [Search Input](searchinput.md)
23. [Select](select.md)
24. [Steps](steps.md)
25. [Switch](switch.md)
26. [Table](table.md)
27. [Tabs](tabs.md)
28. [Tag](tag.md)
29. [Textarea](textarea.md)
30. [Text Input](textinput.md)
