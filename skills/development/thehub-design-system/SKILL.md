---
name: thehub-design-system
description: Senior PHP/Frontend engineer for TheHUB - Swedish cycling competition platform on Uppsala WebHotell. Use when JALLE asks about TheHUB development, GravitySeries, cycling events, PHP design patterns, mobile-first layouts, or component styling.
---

# TheHUB Development System

> **VIKTIGT:** Läs alltid CLAUDE.md och CLAUDE-CSS.md i projektets root FÖRE all utveckling.

## Projektöversikt

**Platform:** PHP 8.x / MySQL på Uppsala WebHotell  
**Domain:** thehub.gravityseries.se  
**Användare:** 3,000+ licensierade cyklister, arrangörer, klubbar  
**Version:** 1.0.x (inga versionsprefix!)

---

## KRITISKA DOKUMENT (Single Source of Truth)

| Fil | Syfte | Läs ALLTID |
|-----|-------|------------|
| `CLAUDE.md` | Utvecklingsregler, sidmallar, DB-schema | Före all kod |
| `CLAUDE-CSS.md` | CSS-tokens, breakpoints, färger | Före all styling |
| `.claude/rules/page-routing.md` | URL -> fil-mappning | Före sidredigering |

---

## STRIKTA FORBUD

### 1. Inga versionsprefix
```php
// FEL
HUB_V3_ROOT, HUB_V2_URL, 'v3/pages/event.php'

// RATT
HUB_ROOT, HUB_URL, 'pages/event.php'
```

### 2. Inga emojis i kod
```php
// FEL
$icon = 'emoji';

// RATT
<i data-lucide="flag"></i>
```

### 3. Inga hardkodade varden
```css
/* FEL */
padding: 16px;
color: #61CE70;

/* RATT */
padding: var(--space-md);
color: var(--color-accent);
```

### 4. Fråga innan ny CSS
Skapa ALDRIG ny CSS utan att fråga först.

---

## CSS-SYSTEM

### Laddningsordning
1. reset.css
2. tokens.css ← Design tokens
3. theme.css ← Dark/Light mode
4. layout.css
5. components.css
6. tables.css
7. pages/{page}.css ← Sidspecifik

### Spacing Tokens (OBLIGATORISKA)
```css
--space-2xs: 4px;
--space-xs: 8px;
--space-sm: 12px;
--space-md: 16px;
--space-lg: 24px;
--space-xl: 32px;
--space-2xl: 48px;
--space-3xl: 64px;
```

### Radius Tokens
```css
--radius-sm: 6px;   /* Mobil: 0 */
--radius-md: 10px;  /* Mobil: 0 */
--radius-lg: 14px;  /* Mobil: 0 */
--radius-full: 9999px;
```

### Färg-tokens (Dark Mode default)
```css
--color-bg-page: #0b131e;
--color-bg-card: #0e1621;
--color-text-primary: #f8f2f0;
--color-text-secondary: #c7cfdd;
--color-accent: #37d4d6;
--color-success: #10b981;
--color-warning: #fbbf24;
--color-error: #ef4444;
```

### Serie-färger
```css
--series-enduro: #FFE009;
--series-downhill: #FF6B35;
--series-xc: #2E7D32;
--series-ges: #EF761F;
--series-ggs: #8A9A5B;
```

---

## Breakpoints

| Namn | Range | Beskrivning |
|------|-------|-------------|
| Mobile Portrait | 0-599px | Smal mobil |
| Mobile Landscape | 600-767px | Mobil liggande |
| Tablet | 768-1023px | Surfplatta |
| Desktop | 1024px+ | Desktop |

### Edge-to-edge på mobil
```css
@media (max-width: 767px) {
  .card {
    margin-left: -16px;
    margin-right: -16px;
    border-radius: 0 !important;
    width: calc(100% + 32px);
  }
}
```

---

## SIDMALLAR

### Publik sida
```php
<?php
require_once __DIR__ . '/config/database.php';
$pageTitle = 'Sidtitel';
include __DIR__ . '/includes/header.php';
?>

<main class="container">
    <!-- INNEHÅLL -->
</main>

<?php include __DIR__ . '/includes/footer.php'; ?>
```

### Admin-sida
```php
<?php
require_once __DIR__ . '/../config/database.php';
require_once __DIR__ . '/../includes/auth.php';
requireAdmin();

$pageTitle = 'Sidtitel';
include __DIR__ . '/../includes/admin-header.php';
?>

<div class="admin-content">
    <div class="page-header">
        <h1><?= $pageTitle ?></h1>
    </div>
</div>

<?php include __DIR__ . '/../includes/admin-footer.php'; ?>
```

---

## KOMPONENTER

### Card
```html
<div class="card">
    <div class="card-header"><h3>Titel</h3></div>
    <div class="card-body">Innehåll</div>
</div>
```

### Button
```html
<button class="btn btn-primary">Primär</button>
<button class="btn btn-secondary">Sekundär</button>
<button class="btn btn-danger">Danger</button>
```

### Alert
```html
<div class="alert alert-success">OK</div>
<div class="alert alert-warning">Varning</div>
<div class="alert alert-danger">Fel</div>
```

### Badge
```html
<span class="badge badge-success">Aktiv</span>
<span class="badge badge-warning">Väntande</span>
```

### Table
```html
<div class="table-responsive">
    <table class="table">
        <thead><tr><th>Kolumn</th></tr></thead>
        <tbody><tr><td>Data</td></tr></tbody>
    </table>
</div>
```

---

## DATABAS-MONSTER

### Prepared Statements (ALLTID)
```php
$stmt = $pdo->prepare("SELECT * FROM riders WHERE id = ?");
$stmt->execute([$id]);
$rider = $stmt->fetch();
```

### Series-koppling (VIKTIGT!)
```php
// FEL - missar events i series_events
JOIN series s ON e.series_id = s.id

// RATT - anvand series_events (many-to-many)
JOIN series_events se ON se.event_id = e.id
JOIN series s ON se.series_id = s.id
```

### Output-sanitering
```php
echo htmlspecialchars($data, ENT_QUOTES, 'UTF-8');
```

---

## PROJEKTSTRUKTUR

```
thehub/
├── CLAUDE.md           # Utvecklingsregler
├── CLAUDE-CSS.md       # CSS-system
├── config/database.php
├── includes/
│   ├── header.php
│   ├── footer.php
│   ├── layout-header.php   # LAST
│   └── layout-footer.php   # LAST
├── components/
│   ├── sidebar.php         # LAST
│   └── header.php          # LAST
├── assets/css/
│   ├── tokens.css          # LAST
│   ├── theme.css           # LAST
│   ├── components.css      # LAST
│   └── pages/*.css
├── pages/              # Publika sidor
├── admin/              # Admin-sidor
├── api/                # API-endpoints
└── Tools/migrations/   # DB-migrationer
```

---

## VERKTYG

### Migrationer
- **Plats:** `/Tools/migrations/` (ENDAST där!)
- **Admin:** `/admin/migrations.php`
- **Format:** `NNN_beskrivande_namn.sql`

### Nya verktyg
- Länka ALLTID från `/admin/tools.php`
- Placera under rätt sektion

---

## CHECKLISTA VID KODANDRING

1. [ ] Läste CLAUDE.md?
2. [ ] Läste CLAUDE-CSS.md?
3. [ ] Använder CSS-variabler (inga hårdkodade värden)?
4. [ ] Testat på mobil (320px)?
5. [ ] Testat i dark mode?
6. [ ] Prepared statements för databas?
7. [ ] Inga versionsprefix?
8. [ ] Inga emojis?

---

## SVENSK TERMINOLOGI

| Engelska | Svenska |
|----------|---------|
| Event | Tävling |
| Registration | Anmälan |
| Results | Resultat |
| Rider | Åkare |
| Club | Klubb |
| Series | Serie |
| Points | Poäng |
| Class | Klass |

---

## RESPONS-FORMAT

### Kod-svar ska vara:
- Kompletta (copy-paste ready)
- Med svenska labels i UI
- Utan "..." eller ofullstandiga delar
- Max 1 fil per svar om inte annat begars

### Format:
```
filnamn.php
[KOMPLETT KOD]

Placera i: /path/to/file
```
