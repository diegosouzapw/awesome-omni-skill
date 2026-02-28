---
name: typst
description: Write and create Typst documents (.typ files). Use when the user asks to create, write, edit, or convert documents to Typst format, write Typst markup, create academic papers, reports, or documents in Typst, convert LaTeX to Typst, or work with .typ files. Typst is a modern markup-based typesetting system alternative to LaTeX.
---

# Typst Document Writing

Typst is a markup-based typesetting system. Files use `.typ` extension. No boilerplate required—just start writing.

## Three Modes

Typst has three syntactical modes:

| Mode | Default In | Enter Via | Exit Via |
|------|-----------|-----------|----------|
| Markup | `.typ` files | `[..]` from code | `#` for code |
| Code | After `#` | `#expr` or `{ }` | `[..]` for markup |
| Math | Never | `$..$ ` | End `$` |

## Essential Syntax

### Markup Mode (default)
```typst
= Heading 1
== Heading 2
*bold* _italic_ `code`
- bullet list
+ numbered list
/ Term: definition
@label-ref
<my-label>
```

### Code Mode (prefix with `#`)
```typst
#let x = 5
#if x > 3 [larger] else [smaller]
#for i in range(3) [Item #i. ]
#rect(width: 2cm, fill: blue)
#image("photo.png", width: 50%)
```

### Math Mode (wrap in `$`)
```typst
Inline $x^2 + y^2 = z^2$ math.

Block math (note spaces inside dollars):
$ sum_(k=1)^n k = (n(n+1))/2 $
```

## Critical Patterns

### Set Rules (configure defaults)
```typst
#set page(margin: 2cm)
#set text(font: "New Computer Modern", size: 11pt)
#set par(justify: true)
#set heading(numbering: "1.1")
```

### Show Rules (restyle elements)
```typst
// Show-set: apply set rule to specific element
#show heading: set text(navy)

// Transform: completely redefine
#show heading: it => [
  #text(blue)[#it.body]
]
```

### Function Calls
```typst
// Named arguments after positional
#rect(width: 2cm, height: 1cm, fill: aqua)

// Trailing content block (common pattern)
#figure(
  image("chart.png", width: 80%),
  caption: [Analysis results],
) <fig-results>
```

### Tables
```typst
#table(
  columns: (auto, 1fr, 1fr),
  [Header 1], [Header 2], [Header 3],
  [Row 1], [Data], [Data],
  [Row 2], [Data], [Data],
)
```

### Grids (layout, not semantic)
```typst
#grid(
  columns: (1fr, 1fr),
  gutter: 1em,
  [Left column],
  [Right column],
)
```

## Math Mode Details

### Key Differences from LaTeX
| LaTeX | Typst |
|-------|-------|
| `\frac{a}{b}` | `a/b` or `frac(a,b)` |
| `\sqrt{x}` | `sqrt(x)` |
| `\sum_{i=1}^{n}` | `sum_(i=1)^n` |
| `\alpha, \beta` | `alpha, beta` |
| `\mathbf{x}` | `bold(x)` |
| `\text{word}` | `"word"` |
| `\left( \right)` | Auto-scales, or `lr(( ))` |
| `\begin{pmatrix}` | `mat(1, 2; 3, 4)` |
| `\begin{cases}` | `cases(a "if" x, b "else")` |
| `\vec{v}` | `arrow(v)` or `vec(a, b, c)` |

### Math Syntax
```typst
$ x^2 $           // superscript
$ x_n $           // subscript  
$ x_(i+1) $       // grouped subscript
$ (a+b)/c $       // fraction
$ sqrt(x) $       // square root
$ root(3, x) $    // nth root
$ sum_(i=0)^n $   // sum with limits
$ integral_a^b $  // integral
$ mat(1, 2; 3, 4) $ // matrix (semicolon = row break)
$ vec(x, y, z) $  // column vector
$ cases(1 "if" x > 0, 0 "else") $
```

### Multi-letter Names
```typst
$ "error" = x - hat(x) $  // quotes for text
$ pi r^2 $                // single letters = variables
$ A B $                   // space = multiplication
```

## Common Pitfalls

1. **Forgetting `#` in markup**: Function calls need `#` prefix in markup mode
   - Wrong: `rect(...)` 
   - Right: `#rect(...)`

2. **Math block spacing**: Block equations need spaces inside `$`
   - Inline: `$x^2$`
   - Block: `$ x^2 $` (spaces required)

3. **Content vs strings**: Use `[content]` for markup, `"string"` for plain text
   - `#text(fill: red)[Hello]` — content with markup
   - `#lower("HELLO")` — string manipulation

4. **Semicolons in expressions**: End expression early with `;`
   - `#x;` to prevent next char joining expression

5. **Multi-letter math variables**: Wrap in quotes or they become function calls
   - Wrong: `$error$` (looks for `error` function)
   - Right: `$"error"$` or single letters `$e$`

## Document Templates

### Academic Paper
```typst
#set document(title: [Paper Title], author: "Author")
#set page(margin: 2.5cm, numbering: "1")
#set text(font: "New Computer Modern", size: 11pt)
#set par(justify: true, leading: 0.65em)
#set heading(numbering: "1.1")

#align(center)[
  #text(17pt, weight: "bold")[Paper Title]
  #v(1em)
  Author Name \
  Institution \
  #link("mailto:email@example.com")
]

#outline()

= Introduction
#lorem(50)

= Methods
#lorem(50)
```

### Report with Figures
```typst
#set page(header: [Report Title #h(1fr) #counter(page).display()])

= Results

#figure(
  table(
    columns: 3,
    [A], [B], [C],
    [1], [2], [3],
  ),
  caption: [Sample data],
) <tab-data>

As shown in @tab-data, ...
```

## Symbol Reference

Common symbols: `=>` (⇒), `->` (→), `<-` (←), `!=` (≠), `<=` (≤), `>=` (≥), `...` (…), `~` (non-breaking space)

Greek: `alpha`, `beta`, `gamma`, `delta`, `epsilon`, `theta`, `lambda`, `mu`, `pi`, `sigma`, `omega`

Variants: `arrow.r`, `arrow.l.double`, `plus.circle` — append modifiers with dots

See full list: https://typst.app/docs/reference/symbols/

## Additional References

For complex tasks, consult these reference files:
- **`references/math.md`**: Comprehensive math mode syntax, symbols, matrices, alignment
- **`references/advanced.md`**: Scripting, templates, page layout, bibliography, imports

## Compilation

```bash
typst compile document.typ           # → document.pdf
typst compile document.typ out.pdf   # custom output
typst watch document.typ             # auto-recompile on save
```
