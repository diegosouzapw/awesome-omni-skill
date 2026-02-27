---
name: libpdf-helper
description: Work with @libpdf/core - modern TypeScript PDF library for parsing, modifying, and generating PDFs. Use when (1) starting new @libpdf/core project, (2) migrating from pdf-lib/pdf.js/pdfkit, (3) understanding @libpdf/core API, (4) solving PDF tasks (forms, signatures, encryption, merging, text extraction), or (5) choosing between PDF libraries.
---

# LibPDF Helper

Comprehensive guidance for using **@libpdf/core** (v0.2.5), a modern TypeScript PDF library that parses, modifies, and generates PDFs.

## What is @libpdf/core?

A production-ready PDF library combining:
- **Lenient parsing** like PDFBox and PDF.js (opens malformed PDFs)
- **Intuitive API** like pdf-lib (TypeScript-first, clean)
- **Complete features**: encryption, digital signatures (PAdES), incremental saves, forms, text extraction

**Key differentiators:**
- Opens documents other libraries reject (brute-force recovery fallback)
- Incremental saves preserve digital signatures
- Full PAdES signature support (B-B, B-T, B-LT, B-LTA)
- Universal runtime (Node.js 20+, Bun, browsers)

## When to Use vs Alternatives

| Task | Use @libpdf/core | Use pdf-lib | Use pdf.js | Use pdfkit |
|------|------------------|-------------|------------|------------|
| Parse malformed PDFs | ✓ Best | ✗ Strict | ✓ Good | ✗ No parsing |
| Fill/flatten forms | ✓ | ✓ | ✗ | ✗ |
| Digital signatures | ✓ PAdES | ✗ | ✗ | ✗ |
| Incremental saves | ✓ | ✗ | ✗ | ✗ |
| Text extraction | ✓ | Limited | ✓ Best | ✗ |
| Generate PDFs | ✓ | ✓ | ✗ | ✓ Best |
| Browser rendering | Use pdf.js | Use pdf.js | ✓ Best | ✗ |
| Server-side | ✓ Best | ✓ | ✓ | ✓ |

**Migrate when:**
- pdf-lib fails on malformed PDFs
- Need digital signatures or incremental saves
- Need complete feature set (encryption, attachments, text extraction)

## Workflows

### 1. Migrating from Another Library

Read the appropriate migration guide:
- **From pdf-lib**: `references/migration-from-pdf-lib.md` (most common)
- **From pdf.js**: `references/migration-from-pdfjs.md` (when adding modification)
- **From pdfkit**: `references/migration-from-pdfkit.md` (when adding parsing)

Guides show side-by-side code comparisons for easy translation.

### 2. Finding Solutions for Specific Tasks

**Option A:** Browse by category in `references/use-cases-by-category.md`
- 12 categories matching common PDF tasks
- Each entry: when to use + code snippet + example reference

**Option B:** Check the [examples directory](https://github.com/LibPDF-js/core/tree/main/examples) with working implementations:
```
01-basic/              Creating, loading, saving
02-pages/              Add, remove, copy, rotate pages
03-forms/              Fill, flatten, create fields
04-drawing/            Text, shapes, paths, custom content
05-images-and-fonts/   Embed images, fonts, subsetting
06-metadata/           Title, author, keywords, dates
07-signatures/         Digital signatures, PAdES, LTV
08-attachments/        Embed and extract files
09-merging-and-splitting/ Combine or split PDFs
10-security/           Encryption, permissions, passwords
11-advanced/           Incremental saves, low-level API
12-text-extraction/    Extract and search text
```

### 3. Quick API Reference

See `references/api-quick-reference.md` for:
- Common patterns by task
- Minimal working examples
- Key method signatures

## Key Concepts

### Two API Layers

**High-level** (use 95% of the time):
```typescript
const pdf = await PDF.load(bytes);
const form = pdf.getForm();
form.fill({ name: "Jane" });
await pdf.save();
```

**Low-level** (for full control):
```typescript
const catalog = pdf.context.catalog;
const pagesDict = catalog.get("Pages") as PdfDict;
// Work with PDF objects directly (PdfDict, PdfArray, PdfStream)
```

### Incremental Saves

Critical for preserving digital signatures:
```typescript
const signed = await pdf.save({ incremental: true });
```

Without `incremental: true`, signatures become invalid. The library automatically uses incremental saves when modifying signed PDFs.

### Lenient Parsing

Opens malformed PDFs that strict parsers reject:
- Recovers xref tables by scanning entire file
- Tolerates incorrect offsets and object numbers
- Handles missing or malformed trailers

Falls back to brute-force when standard parsing fails.

## Installation

```bash
npm install @libpdf/core
# or
bun add @libpdf/core
```

**Requirements:**
- Node.js 20+ (or Bun)
- TypeScript 5+ (recommended)
- Modern browser with Web Crypto API (for browser usage)

## Quick Examples

**Load and inspect:**
```typescript
import { PDF } from "@libpdf/core";
const pdf = await PDF.load(bytes);
console.log(`${pdf.getPageCount()} pages`);
```

**Fill form:**
```typescript
const form = pdf.getForm();
form.fill({ name: "Jane", email: "jane@example.com", agreed: true });
await pdf.save();
```

**Sign document:**
```typescript
import { P12Signer } from "@libpdf/core";
const signer = await P12Signer.create(p12Bytes, "password");
const signed = await pdf.sign({ signer, reason: "I approve" });
```

**Merge PDFs:**
```typescript
const merged = await PDF.merge([pdf1Bytes, pdf2Bytes, pdf3Bytes]);
```

**Extract text:**
```typescript
const text = pdf.getPage(0).extractText();
const results = pdf.getPage(0).findText(/invoice #\d+/gi);
```

## Resources

- **Documentation**: https://libpdf.dev
- **GitHub**: https://github.com/LibPDF-js/core
- **Examples**: [examples/](https://github.com/LibPDF-js/core/tree/main/examples) (12 categories)
- **Issues**: https://github.com/LibPDF-js/core/issues

## Known Limitations

- Signature verification not yet implemented (signing works)
- TrueType Collections (.ttc) not supported
- JBIG2 and JPEG2000 images passthrough only (preserved but not decoded)
- Certificate encryption not supported (password encryption works)
- JavaScript form calculations ignored

These limitations affect edge cases; core features are production-ready.
