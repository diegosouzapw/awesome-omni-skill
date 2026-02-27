---
name: ad-assessment-documents
description: Professional document generator for Active Directory health assessments. Creates executive-quality DOCX reports with consistent branding, proper data visualization, severity-based formatting, and actionable remediation guidance. Use when generating assessment reports from OpsIdentity JSON data or improving document output quality.
---

# AD Assessment Document Generator Skill

Generate **professional, executive-ready** DOCX assessment reports from Active Directory health data.

> "The document should look like it came from a Big 4 consulting firm, not a script output."

## When to Use This Skill

| Trigger | Action |
|---------|--------|
| "Generate assessment document" | Load [📄 Document Structure](./references/document-structure.md) |
| "Improve report formatting" | Check [🎨 Design System](./references/design-system.md) |
| "Fix table layouts" | Reference [📊 Table Templates](./references/table-templates.md) |
| "Format findings section" | Use [🔍 Finding Cards](./references/finding-cards.md) |

## Design Philosophy

### Professional Standards

| Principle | Implementation |
|-----------|----------------|
| **Executive First** | Lead with business impact, technical details follow |
| **Visual Hierarchy** | Score card → Summary table → Detailed findings |
| **Actionable** | Every finding includes copy-paste remediation |
| **Consistent** | Same styling throughout, no visual surprises |
| **Error-Free** | Never show `undefined`, `null`, or `[object Object]` |

### Color Palette (Severity-Based)

| Severity | Fill Color | Text Color | Usage |
|----------|------------|------------|-------|
| Critical | `#FEE2E2` | `#991B1B` | Red badge, immediate action |
| High | `#FEF3C7` | `#92400E` | Orange badge, priority action |
| Medium | `#FEF9C3` | `#854D0E` | Yellow badge, scheduled action |
| Low | `#DBEAFE` | `#1E40AF` | Blue badge, optimization |
| Info | `#F3F4F6` | `#374151` | Gray badge, informational |
| Success | `#D1FAE5` | `#065F46` | Green badge, compliant |

### Typography System

```
Font Family: Arial (universal compatibility)

Title:           28pt, Bold, #1E3A5F (Navy)
Heading 1:       18pt, Bold, #1E3A5F
Heading 2:       14pt, Bold, #374151 (Gray-700)
Heading 3:       12pt, Bold, #4B5563 (Gray-600)
Body:            11pt, Regular, #111827 (Gray-900)
Caption:         9pt, Regular, #6B7280 (Gray-500)
Code:            10pt, Consolas, #1F2937, Background #F3F4F6
```

## Document Structure

### Required Sections (In Order)

1. **Cover Page**
   - Logo placeholder (centered)
   - Document title: "Active Directory Health Assessment"
   - Subtitle: "Security and Configuration Report"
   - Client domain name (prominent)
   - Assessment date
   - Confidentiality notice

2. **Executive Summary** (1 page max)
   - Health score gauge (0-100)
   - Risk distribution chart (text-based)
   - Top 3-5 critical findings (bullet summary)
   - Immediate action items

3. **Environment Overview**
   - Forest/Domain properties table
   - Domain Controllers status table
   - FSMO roles health table
   - Sites and subnets summary

4. **Findings by Category**
   - Grouped by: Critical → High → Medium → Low
   - Each finding uses the Finding Card template
   - Include affected objects count

5. **Detailed Remediation**
   - Step-by-step instructions
   - PowerShell commands (properly formatted)
   - Validation steps
   - External documentation links

6. **Appendices**
   - Full affected objects lists
   - Technical metrics raw data
   - Glossary of terms

## Data Validation Rules

### CRITICAL: Never Output These

```javascript
// Before generating any field, validate:
if (value === undefined || value === null || value === '') {
  return 'No disponible';  // Or appropriate default
}
if (typeof value === 'object' && !Array.isArray(value)) {
  return JSON.stringify(value);  // Or extract specific field
}
if (value === 'undefined ms' || value === 'undefined') {
  return 'N/A';
}
```

### Health Score Calculation

```javascript
// Score should NEVER be 0 if systems are operational
function calculateHealthScore(findings) {
  let score = 100;
  
  findings.forEach(f => {
    switch(f.severity) {
      case 'CRITICAL': score -= 15; break;
      case 'HIGH': score -= 10; break;
      case 'MEDIUM': score -= 5; break;
      case 'LOW': score -= 2; break;
    }
  });
  
  return Math.max(0, score);  // Floor at 0
}
```

## Quick Reference: docx-js Patterns

### Severity Badge Cell

```javascript
function createSeverityCell(severity) {
  const colors = {
    'CRITICAL': { fill: 'FEE2E2', text: '991B1B' },
    'HIGH': { fill: 'FEF3C7', text: '92400E' },
    'MEDIUM': { fill: 'FEF9C3', text: '854D0E' },
    'LOW': { fill: 'DBEAFE', text: '1E40AF' }
  };
  const c = colors[severity] || colors['LOW'];
  
  return new TableCell({
    shading: { fill: c.fill, type: ShadingType.CLEAR },
    children: [new Paragraph({
      alignment: AlignmentType.CENTER,
      children: [new TextRun({ 
        text: severity, 
        bold: true, 
        color: c.text,
        size: 20 
      })]
    })]
  });
}
```

### Code Block Paragraph

```javascript
function createCodeBlock(code) {
  return new Paragraph({
    shading: { fill: 'F3F4F6', type: ShadingType.CLEAR },
    spacing: { before: 120, after: 120 },
    indent: { left: 360, right: 360 },
    children: [new TextRun({
      text: code,
      font: 'Consolas',
      size: 20,
      color: '1F2937'
    })]
  });
}
```

## Reference Files

Load these for specific implementation details:

- [📄 Document Structure](./references/document-structure.md) - Complete section templates with docx-js code
- [🎨 Design System](./references/design-system.md) - Full color palette, typography, spacing
- [📊 Table Templates](./references/table-templates.md) - Pre-built table layouts for each section
- [🔍 Finding Cards](./references/finding-cards.md) - Individual finding presentation format

## Anti-Patterns to Avoid

| ❌ Don't | ✅ Do Instead |
|----------|---------------|
| Show `[object Object]` | Extract specific property or stringify |
| Use `undefined` in tables | Use "N/A" or "No disponible" |
| Mix languages randomly | Consistent Spanish or English throughout |
| Escape PowerShell chars (`\$`) | Use raw code, handle escaping in generation |
| "Ver detalles en otra sección" | Include details inline or specific page ref |
| Score of 0 with operational DCs | Calculate proper weighted score |
| Inconsistent table widths | Use fixed column width templates |
| Unicode bullets in lists | Use proper docx-js numbering config |

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2025-12-27 | Initial skill creation |
