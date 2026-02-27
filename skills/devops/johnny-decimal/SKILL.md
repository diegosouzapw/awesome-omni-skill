---
name: johnny-decimal
description: Johnny Decimal + PARA organization
metadata:
  globs: ""
---

# Johnny Decimal + PARA Organization System

A hybrid system combining [Johnny Decimal](https://johnnydecimal.com/) structure with [PARA](https://fortelabs.com/blog/para/) methodology.

## Structure

```
AC.ID — Title
│
├── AC   = Area (10-99) + Category (X0-X9)
└── ID   = Unique identifier within category (01-99)
```

## PARA Mapping

| JD Area | PARA      | Purpose                                 |
| ------- | --------- | --------------------------------------- |
| `10-19` | Projects  | Active projects with a defined outcome  |
| `20-29` | Areas     | Ongoing responsibilities to maintain    |
| `30-39` | Resources | Reference materials, topics of interest |
| `90-99` | Archives  | Completed or inactive items             |
| `Inbox` | —         | Items to process and organize           |

## Project Numbering

Projects use a year-based ID format:

```
10.{YY}{NNN} — Project Title
     │  │
     │  └── NNN = Sequential number (001, 002, ...)
     └───── YY  = Year (25 for 2025)
```

Example: `10.25001 — Website Redesign` (first project of 2025)

## Rules

- **Categories** subdivide areas (11, 12, 13...) — max 10 per area
- **IDs** are unique items within a category (11.01, 11.02...)
- **Projects** use `10.{YY}{NNN}` format with year prefix
- Keep **one location** per ID — no duplicates
- Use **descriptive titles** after the number
- Move completed projects to **90-99 Archives**

## File Naming Convention

Files inherit their prefix from the folder path:

```
{Category} - {Subcategory} - {Date} - {Description}.{ext}
    │            │             │           │
    │            │             │           └── Free-form description
    │            │             └── ISO date YYYY-MM-DD (optional)
    │            └── From parent folder name (e.g., "27.05 - Fiscalité" → "Fiscalité")
    └── From grandparent folder name (e.g., "27 - Finances" → "Finances")
```

### Examples

| Folder Path                            | File Name                                                  |
| -------------------------------------- | ---------------------------------------------------------- |
| `27 - Finances/27.05 - Fiscalité/`     | `Finances - Fiscalité - 2024-04-15 - Déclaration 2042.pdf` |
| `23 - Sport/23.01 - Tir à l'arc/`      | `Sport - Tir à l'arc - 2024-09-14 - Licence FFTA.pdf`      |
| `34 - Administratif/34.03 - Identité/` | `Administratif - Identité - Passeport.pdf`                 |

### Rules

- **Separator**: `-` (space-dash-space)
- **Date**: Optional, ISO format `YYYY-MM-DD`
- **Description**: Title Case, accents allowed
- **No date** for reference documents (rules, templates)

## Best Practices

- Start simple — add categories only when needed
- Use consistent naming conventions across all items
- Prefer flat structures over deep nesting
- Keep **Inbox at zero** — process and file items regularly

## Index Location

The Johnny Decimal index is maintained in **Apple Notes**. Use the `mcp_apple-notes` tools to search or consult the index when needed.
