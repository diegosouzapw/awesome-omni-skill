---
name: latex
description: This skill should be used when the user asks to "create a LaTeX document", "convert markdown to LaTeX", "make a PDF from markdown", "format as LaTeX", "write LaTeX", or wants to produce beautifully typeset documents. Handles LaTeX generation from any input source with professional typography, and optionally compiles to PDF.
allowed-tools: Read, Write, Bash(pdflatex:*), Bash(xelatex:*), Bash(lualatex:*), Bash(latexmk:*), Bash(biber:*), Bash(bibtex:*), Bash(which:*), Bash(uname:*)
---

# LaTeX Document Creator

Create beautifully formatted LaTeX documents from any input source (markdown files, plain text, structured data, or user instructions). Produce professional-quality typeset output following modern LaTeX best practices.

## Workflow

1. **Analyze input** - Read the source material, identify structure (headings, lists, tables, figures, math, code, citations)
2. **Determine document class** - Select appropriate class based on content type and length
3. **Generate LaTeX** - Produce a complete `.tex` file with proper preamble and body
4. **Compile to PDF** (if requested) - Detect platform and compile using `latexmk`

If the user provides additional formatting instructions, apply them. User instructions override defaults below.

## Document Class Selection

| Content Type | Class | When |
|---|---|---|
| Short documents, articles, memos | `scrartcl` | No chapters needed |
| Reports, theses, long documents | `scrreprt` | Chapter-level structure |
| Books, manuals | `scrbook` | Front/back matter, parts |
| Presentations | `beamer` | Slides |

Use KOMA-Script classes (`scrartcl`, `scrreprt`, `scrbook`) over standard classes for superior typography defaults and built-in customization.

## Standard Preamble

Organize the preamble in this order. Include only packages the document actually needs.

```latex
\documentclass[a4paper, 11pt]{scrartcl}

% --- Typography ---
\usepackage[T1]{fontenc}
\usepackage{lmodern}              % Clean, professional font
\usepackage{microtype}            % ESSENTIAL: character protrusion + font expansion

% --- Math (include only if document has math) ---
\usepackage{mathtools}            % Superset of amsmath with fixes
\usepackage{amssymb}

% --- Layout ---
\usepackage{geometry}

% --- Tables & Figures ---
\usepackage{booktabs}             % Professional table rules
\usepackage{graphicx}
\usepackage{caption}
\usepackage{subcaption}

% --- Lists ---
\usepackage{enumitem}

% --- Language & Quotes ---
\usepackage[english]{babel}
\usepackage{csquotes}

% --- Colors ---
\usepackage{xcolor}

% --- Code Listings (include only if document has code) ---
\usepackage{listings}

% --- Units (include only if document has quantities) ---
\usepackage{siunitx}

% --- Links & References (load near-last) ---
\usepackage{hyperref}
\hypersetup{
  colorlinks=true,
  linkcolor=blue!70!black,
  citecolor=green!50!black,
  urlcolor=blue!70!black,
}
\usepackage{cleveref}             % MUST be after hyperref
```

### Package Notes

- `microtype` is non-negotiable for professional output - always include it
- `mathtools` loads `amsmath` automatically; never load both
- `cleveref` must load after `hyperref`
- `hyperref` should load near-last
- For bibliography: use `biblatex` with `biber` backend, not legacy `bibtex`/`natbib`

## Typography Rules

### Fonts

- Default: `lmodern` (Latin Modern) - clean, professional, widely available
- Alternative serif: `libertinus`, `newtxtext`/`newtxmath` (Times-like)
- For system fonts (Unicode): switch to LuaLaTeX with `fontspec`

### Spacing and Punctuation

- En-dash for ranges: `2020--2025` renders as 2020–2025
- Em-dash for breaks: `word---word` renders as word—word
- Non-breaking space (`~`) before `\cite`, `\cref`, and between numbers and units
- Thin space before differentials: `\int f(x) \, dx`
- Use `\enquote{}` from `csquotes` for quotation marks, never manual quote characters
- Use `\emph{}` instead of `\textit{}` - semantic emphasis that adapts to context
- Use `\dots` for ellipses, never three periods

### Sentence Spacing

- After abbreviations (not ending a sentence): `e.g.\ this`, `i.e.\ that`
- After a capital letter ending a sentence: `NASA\@. The next sentence`

## Formatting Standards

### Tables

- Always use `booktabs`: `\toprule`, `\midrule`, `\bottomrule`
- Never use vertical rules (`|`) or `\hline`
- Place table captions **above** the table
- Place figure captions **below** the figure

### Figures

- Use `\centering` inside floats, not `\begin{center}`
- Default float placement: `[htbp]`
- Reference all figures in text before they appear

### Cross-References

- Use `\cref{}` from `cleveref` - automatically produces "Figure 1", "Table 2", etc.
- Label prefixes: `fig:`, `tab:`, `sec:`, `eq:`, `lst:`
- Place `\label` immediately after `\caption` or `\section`

### Lists

- Use `enumitem` for customization
- Avoid nesting deeper than 3 levels - restructure content instead

## Common Pitfalls to Avoid

- Never use `$$...$$` for display math - use `\[...\]` or `equation` environment
- Never use `\\` for paragraph breaks - use a blank line
- Never use `\begin{center}` inside floats - use `\centering`
- Never place `\label` before `\caption` - produces wrong reference numbers
- Never hardcode reference numbers - always use `\cref{}`
- Never use bare function names in math - use `\sin`, `\log`, or `\DeclareMathOperator`
- Escape special characters: `#`, `%`, `$`, `&`, `_`, `{`, `}`

## Markdown-to-LaTeX Conversion

When converting from markdown, apply these mappings:

| Markdown | LaTeX |
|---|---|
| `# Heading` | `\section{Heading}` |
| `## Heading` | `\subsection{Heading}` |
| `### Heading` | `\subsubsection{Heading}` |
| `**bold**` | `\textbf{bold}` |
| `*italic*` | `\emph{italic}` |
| `` `code` `` | `\texttt{code}` |
| `> blockquote` | `\begin{quote}...\end{quote}` |
| `- item` | `\begin{itemize}\item ...\end{itemize}` |
| `1. item` | `\begin{enumerate}\item ...\end{enumerate}` |
| `[text](url)` | `\href{url}{text}` |
| `![alt](path)` | `\begin{figure}...\includegraphics{path}...\end{figure}` |
| `` ```lang ``` `` | `\begin{lstlisting}[language=lang]...\end{lstlisting}` |
| `---` (horizontal rule) | `\bigskip\noindent\rule{\textwidth}{0.4pt}\bigskip` |
| Tables | `\begin{tabular}` with `booktabs` rules |
| `$math$` | `$math$` (same) |
| `$$math$$` | `\[math\]` |

### Conversion Guidelines

- Infer document title from the first `#` heading or filename
- Infer `\author` and `\date` if present in the source, otherwise omit
- Preserve the semantic structure - do not flatten or over-nest headings
- Convert markdown tables to `booktabs`-styled `tabular` environments
- Escape all LaTeX special characters in text content
- For documents with chapters, promote heading levels (`#` → `\chapter`, `##` → `\section`)

## Compiling to PDF

When the user requests PDF output, detect the platform and compile.

### Engine Selection

| Engine | Command | Use When |
|---|---|---|
| pdfLaTeX | `latexmk -pdf` | Default. ASCII/Latin content, standard fonts, fastest compilation |
| XeLaTeX | `latexmk -xelatex` | System fonts via `fontspec`, native Unicode |
| LuaLaTeX | `latexmk -lualatex` | System fonts + Lua scripting, no memory limits |

### Using latexmk (Preferred)

`latexmk` automatically runs the correct number of passes for cross-references, bibliographies, and indices. Always prefer it over manual multi-pass compilation.

```bash
latexmk -pdf document.tex       # Compile to PDF (pdflatex)
latexmk -xelatex document.tex   # Compile with XeTeX
latexmk -lualatex document.tex  # Compile with LuaTeX
latexmk -c document.tex         # Clean auxiliary files (keep PDF)
```

### Manual Compilation (When latexmk Is Unavailable)

```bash
# Simple document (no bibliography)
pdflatex document.tex && pdflatex document.tex

# Document with biblatex/biber bibliography
pdflatex document.tex && biber document && pdflatex document.tex && pdflatex document.tex
```

### Platform-Specific Installation

If LaTeX tools are not installed or compilation fails due to missing packages, read the appropriate platform guide for installation instructions:

- **Ubuntu/Debian**: Read `references/pdf-compilation-ubuntu.md`
- **macOS**: Read `references/pdf-compilation-macos.md`

Detect the platform with `uname -s` (`Linux` or `Darwin`).

Package installation requires system permissions — present the installation command to the user and let them confirm before running `sudo` or `brew` commands.
