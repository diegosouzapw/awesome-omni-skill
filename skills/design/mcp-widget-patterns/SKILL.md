---
name: MCP Widget Patterns
description: This skill should be used when the user asks to "design a widget", "what widget pattern should I use", "inline card design", "carousel widget", "fullscreen mode", "picture in picture", "widget layout", "card design for ChatGPT", or needs guidance on specific widget patterns and implementations for OpenAI Apps SDK.
version: 0.1.0
---

# MCP Widget Patterns for OpenAI Apps SDK

## Overview

Widgets are visual components that render inline with ChatGPT conversations. This skill catalogs proven widget patterns with implementation guidance for the OpenAI Apps SDK.

## Display Modes

The Apps SDK supports four display modes, each suited to different use cases:

| Mode | Use When | Example |
|------|----------|---------|
| **Inline** | Quick confirmations, simple data | Order confirmation card |
| **Inline Carousel** | Comparing similar items | Product search results |
| **Fullscreen** | Complex workflows, rich interaction | Document editor, maps |
| **Picture-in-Picture** | Persistent parallel activity | Video call, game |

## Inline Card Patterns

### Simple Confirmation Card

**Use for:** Action confirmations, status updates, receipts

```html
<div class="card">
    <div class="card-header">
        <span class="icon">✓</span>
        <span class="title">Booking Confirmed</span>
    </div>
    <div class="card-body">
        <div class="detail-row">
            <span class="label">Restaurant</span>
            <span class="value">Chez Pierre</span>
        </div>
        <div class="detail-row">
            <span class="label">Date</span>
            <span class="value">Tonight, 7:00 PM</span>
        </div>
        <div class="detail-row">
            <span class="label">Party Size</span>
            <span class="value">2 guests</span>
        </div>
    </div>
    <div class="card-actions">
        <button class="btn-secondary">Modify</button>
        <button class="btn-primary">Add to Calendar</button>
    </div>
</div>
```

**Design rules:**
- Maximum 2 primary actions at bottom
- 3-5 detail rows maximum
- No deep navigation within card

### Status Timeline Card

**Use for:** Order tracking, process status, delivery updates

```html
<div class="timeline-card">
    <div class="timeline-header">Order #12345</div>
    <div class="timeline">
        <div class="step completed">
            <div class="marker">✓</div>
            <div class="content">
                <div class="step-title">Confirmed</div>
                <div class="step-time">2:30 PM</div>
            </div>
        </div>
        <div class="step current">
            <div class="marker">●</div>
            <div class="content">
                <div class="step-title">Preparing</div>
                <div class="step-time">Estimated 15 min</div>
            </div>
        </div>
        <div class="step pending">
            <div class="marker">○</div>
            <div class="content">
                <div class="step-title">Out for Delivery</div>
            </div>
        </div>
    </div>
</div>
```

### Data Card with Badge

**Use for:** Account info, subscription status, quick stats

```html
<div class="data-card">
    <div class="card-header">
        <span class="title">Account Balance</span>
        <span class="badge badge-success">Active</span>
    </div>
    <div class="card-value">$1,234.56</div>
    <div class="card-subtitle">Available balance</div>
    <div class="card-actions">
        <button class="btn-primary">Transfer</button>
    </div>
</div>
```

## Carousel Patterns

### Product Carousel

**Use for:** Search results, recommendations, similar items

```html
<div class="carousel">
    <div class="carousel-item">
        <img src="product1.jpg" alt="Product" class="item-image">
        <div class="item-content">
            <div class="item-title">Wireless Headphones</div>
            <div class="item-price">$149.99</div>
            <div class="item-rating">★★★★☆ (234)</div>
        </div>
        <button class="btn-primary">View Details</button>
    </div>
    <!-- Repeat for 3-8 items -->
</div>
```

**Design rules:**
- 3-8 items maximum
- Consistent visual hierarchy across cards
- Up to 3 lines of metadata per card
- Single primary action per card

### Media Carousel

**Use for:** Image galleries, video thumbnails, portfolio items

```html
<div class="media-carousel">
    <div class="media-item">
        <div class="media-thumbnail">
            <img src="photo1.jpg" alt="Gallery image">
            <div class="media-overlay">
                <span class="duration">2:30</span>
            </div>
        </div>
        <div class="media-title">Beach Sunset</div>
    </div>
</div>
```

### List Carousel (Ranked)

**Use for:** Top results, favorites, prioritized lists

```html
<div class="ranked-carousel">
    <div class="ranked-item">
        <div class="rank">1</div>
        <div class="item-content">
            <div class="item-title">Best Match Restaurant</div>
            <div class="item-meta">Italian • $$ • 0.3 mi</div>
            <div class="item-rating">4.8 ★</div>
        </div>
        <button class="btn-icon favorite">♡</button>
    </div>
</div>
```

## Fullscreen Patterns

### Editor Canvas

**Use for:** Document editing, code editing, design tools

```javascript
// Request fullscreen mode
await window.openai.requestDisplayMode('fullscreen');

// Widget structure
<div class="fullscreen-editor">
    <header class="editor-toolbar">
        <div class="toolbar-left">
            <button>Bold</button>
            <button>Italic</button>
        </div>
        <div class="toolbar-right">
            <button class="btn-secondary">Cancel</button>
            <button class="btn-primary">Save</button>
        </div>
    </header>
    <main class="editor-canvas">
        <!-- Editable content -->
    </main>
</div>
```

### Map View

**Use for:** Location selection, route display, area exploration

```html
<div class="fullscreen-map">
    <div class="map-container" id="map"></div>
    <div class="map-controls">
        <button class="btn-icon">🔍+</button>
        <button class="btn-icon">🔍-</button>
        <button class="btn-icon">📍</button>
    </div>
    <div class="map-info-panel">
        <div class="location-name">Selected Location</div>
        <div class="location-address">123 Main St</div>
        <button class="btn-primary">Confirm Location</button>
    </div>
</div>
```

### Multi-Step Wizard

**Use for:** Complex forms, onboarding, guided processes

```html
<div class="fullscreen-wizard">
    <header class="wizard-progress">
        <div class="step completed">1</div>
        <div class="step current">2</div>
        <div class="step">3</div>
    </header>
    <main class="wizard-content">
        <!-- Current step content -->
    </main>
    <footer class="wizard-actions">
        <button class="btn-secondary">Back</button>
        <button class="btn-primary">Continue</button>
    </footer>
</div>
```

## Picture-in-Picture Patterns

### Video Player

**Use for:** Video calls, tutorials, live streams

```javascript
// Request PiP mode
await window.openai.requestDisplayMode('pip');

<div class="pip-video">
    <video id="player" autoplay></video>
    <div class="pip-controls">
        <button class="btn-icon">⏸</button>
        <button class="btn-icon">🔊</button>
        <button class="btn-icon close">✕</button>
    </div>
</div>
```

### Live Session

**Use for:** Games, collaborative editing, real-time monitoring

```html
<div class="pip-session">
    <div class="session-status">
        <span class="live-indicator">●</span>
        <span>Live Session</span>
    </div>
    <div class="session-content">
        <!-- Live content -->
    </div>
    <button class="pip-close">End Session</button>
</div>
```

## Styling Guidelines

### Color System

Use ChatGPT's semantic colors:

```css
:root {
    /* Text */
    --text-default: #000000;
    --text-secondary: #666666;
    --text-subtle: #999999;

    /* Surfaces */
    --surface-primary: #ffffff;
    --surface-secondary: #f5f5f5;
    --surface-elevated: #ffffff;

    /* Actions */
    --action-primary: #0066cc;
    --action-secondary: #e0e0e0;

    /* Status */
    --status-success: #22c55e;
    --status-warning: #f59e0b;
    --status-error: #ef4444;

    /* Borders */
    --border-default: #e0e0e0;
    --border-subtle: #f0f0f0;
}

/* Dark mode */
.dark {
    --text-default: #ffffff;
    --text-secondary: #a0a0a0;
    --surface-primary: #1a1a1a;
    --surface-secondary: #2d2d2d;
    --border-default: #404040;
}
```

### Typography

Inherit system fonts:

```css
body {
    font-family: system-ui, -apple-system, BlinkMacSystemFont,
                 'Segoe UI', Roboto, sans-serif;
    font-size: 14px;
    line-height: 1.5;
}

.heading-lg { font-size: 18px; font-weight: 600; }
.heading-md { font-size: 16px; font-weight: 600; }
.body { font-size: 14px; }
.body-small { font-size: 12px; }
.caption { font-size: 11px; color: var(--text-secondary); }
```

### Spacing

Consistent spacing scale:

```css
--space-xs: 4px;
--space-sm: 8px;
--space-md: 12px;
--space-lg: 16px;
--space-xl: 24px;
--space-2xl: 32px;
```

### Border Radius

Match ChatGPT's rounded corners:

```css
--radius-sm: 4px;
--radius-md: 8px;
--radius-lg: 12px;
--radius-full: 9999px;
```

## Pattern Selection Guide

| User Need | Pattern | Display Mode |
|-----------|---------|--------------|
| Confirm an action | Confirmation Card | Inline |
| Compare 3-8 options | Carousel | Inline |
| Track progress | Timeline Card | Inline |
| View account status | Data Card | Inline |
| Edit a document | Editor Canvas | Fullscreen |
| Select a location | Map View | Fullscreen |
| Complete complex form | Wizard | Fullscreen |
| Watch video content | Video Player | PiP |
| Join live session | Live Session | PiP |

## Additional Resources

### Reference Files

For detailed implementation:
- **`references/css-variables.md`** - Complete CSS variable reference
- **`references/responsive-patterns.md`** - Mobile/desktop adaptations

### Example Files

Working examples in `examples/`:
- **`examples/inline-card.html`** - Basic inline card template
- **`examples/carousel.html`** - Product carousel template
- **`examples/fullscreen-editor.html`** - Fullscreen editor template

### Official Resources

- UI Kit: https://github.com/openai/apps-sdk-ui
- UI Guidelines: https://developers.openai.com/apps-sdk/concepts/ui-guidelines/
- Examples: https://github.com/openai/openai-apps-sdk-examples
