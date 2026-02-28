---
name: Create LaTeX Notes
description: Generate structured LaTeX exam cheat sheets from course materials. Extracts key concepts, definitions, formulas from PDFs, Jupyter notebooks, and markdown files. Compiles to PDF with error checking.
---

# Create LaTeX Notes

## Purpose
This skill generates concise, exam-ready LaTeX cheat sheets from course materials in the `src` directory. It creates well-structured reference documents optimized for quick review during exams.

**IMPORTANT**: All generated notes MUST be in English, regardless of the language used during interaction with the user. The user studies in English and needs exam notes in English.

## Instructions

### 1. Content Analysis
- **Locate source materials**: Identify all course content in the specified directory under `src/`
- **Process multiple formats**:
  - `.md` files: Extract headings, definitions, formulas, code examples
  - `.ipynb` files: Extract markdown cells, code outputs, formulas, key visualizations
  - `.pdf` files: Extract text content, focus on definitions, theorems, formulas
- **Prioritize content**:
  - Core concepts and definitions (always include)
  - Mathematical formulas and equations (always include)
  - Key algorithms and their complexity (always include)
  - Important code snippets (include only the most critical examples)
  - Diagrams descriptions (brief text descriptions only, no images unless space permits)

### 2. Structure Guidelines
Use the template from `templates/template.tex` as the base structure:
- **3-column layout** for maximum information density
- **Grouped by topic**: Organize content into logical sections with topic-focused headers
  - ✓ Good: "Linear Regression", "K-Means Clustering", "Decision Trees"
  - ✗ Bad: "Module 2: Linear Regression", "Week 4: K-Means Clustering"
  - Only include module/week numbers if user specifically requests them
- **Tabular format**: Use tables with `\textbf{Concept}:` followed by brief explanation
- **Consistent formatting**:
  - Bold for key terms
  - Math mode for formulas: `$...$` for inline, `$$...$$` for display
  - Code snippets: Use `\texttt{...}` for inline code or `\begin{verbatim}...\end{verbatim}` for blocks

**Preventing Orphan Headers** (title on one page, table on another):

**Method 1: Use `\topic` command** (Recommended for simple sections):
```latex
\topic{Topic Title}{
    \concept{Item 1}: Description.\\
    \concept{Item 2}: Description.\\
    \concept{Item 3}: Description.\\
}
```
The `\topic` command keeps the title and table together as one unbreakable block.

**IMPORTANT**: Always end content with `\\` before closing brace to ensure bottom table border renders correctly.

**Method 2: Use `gather*` environment** (For complex sections):
```latex
\needspace{4\baselineskip}
\begin{samepage}
\begin{gather*}
    \textbf{Topic Title}\\[1pt]
    \begin{tabular}{|p{6cm}|}
        \hline
        \concept{Item 1}: Description.\\
        \concept{Item 2}: Description.\\
        \hline
    \end{tabular}
\end{gather*}
\end{samepage}
\vspace{2pt}
```
Use this when you need:
- Multiple tables under one heading
- Special formatting or custom spacing
- Complex mathematical content with tables

**Key improvements**:
- `\\[1pt]` instead of `\\[-2pt]` - better spacing between title and table
- `\needspace{4\baselineskip}` - ensures minimum 4 lines available before starting
- `samepage` environment - keeps content together on same page

### 3. Content Extraction Rules
- **Language**: ALL notes MUST be in English, regardless of the language used in communication
- **Brevity first**: Keep explanations to 1-2 lines maximum
- **Formulas**: Include all mathematical formulas exactly as written
- **Code examples**: Include only essential snippets (prefer pseudocode over full implementations)
- **Definitions**: Format as "**Term**: Brief definition"
- **Avoid redundancy**: Don't repeat information across sections
- **No prose**: Use bullet points, tables, and structured formats only

### 4. File Organization
- **Output location**: Save the generated `.tex` file in the same course directory (e.g., `src/CM3010 Databases/.../cheat-sheet.tex`)
- **Naming convention**: Use descriptive names like `{course-code}-cheat-sheet.tex` or `{topic}-notes.tex`
- **Preserve template**: Don't modify the original template file

### 5. Compilation Process
After generating the `.tex` file, compile it to PDF using the provided compilation script:

**Compilation command**:
```bash
bash .claude/skills/create-latex-notes/scripts/compile_latex.sh {tex_file_path}
```

The script automatically:
- Detects available LaTeX compiler (xelatex, pdflatex, or lualatex in order of preference)
- Creates `out/` directory for auxiliary files
- Runs compilation twice for proper references
- Copies final PDF to source directory
- Provides clear error messages if compilation fails

**Alternative (manual compilation)**:
If you need to customize the compilation process, you can modify the script at:
`.claude/skills/create-latex-notes/scripts/compile_latex.sh`

Or compile manually:
```bash
# macOS (typical path)
/Library/TeX/texbin/xelatex -interaction=nonstopmode -output-directory=out {tex_file_path}

# Linux/other systems
xelatex -interaction=nonstopmode -output-directory=out {tex_file_path}
# or: pdflatex -interaction=nonstopmode -output-directory=out {tex_file_path}
```

**Compilation verification**:
1. Check the exit code of the compilation command
2. Verify the PDF file was created in the `out` directory
3. If compilation fails:
   - Parse the `.log` file for errors
   - Report the specific line number and error message
   - Suggest corrections (common issues: missing packages, special characters, unescaped symbols)
   - Offer to fix the `.tex` file and recompile
4. If successful, report the output PDF location

**Error handling**:
- No compiler found: Script provides installation instructions
- Missing packages: Suggest installation commands
- Unicode/encoding issues: Check for special characters that need escaping
- Overfull boxes: These are warnings, not errors - compilation still succeeds
- Math mode errors: Check for unmatched `$` symbols

### 6. Quality Checklist
Before finalizing, ensure:
- [ ] All major topics from the course are covered
- [ ] Formulas are correctly formatted in math mode
- [ ] Content fits within page margins (no overflow)
- [ ] Tables are properly closed (all `\begin{}` have matching `\end{}`)
- [ ] No duplicate content across sections
- [ ] Code examples are properly escaped
- [ ] File compiles without errors
- [ ] PDF is readable at small font size

### 7. Content Prioritization
Focus on the most important exam-relevant content:
1. Core definitions and formulas (essential)
2. Key algorithms with complexity
3. Critical code patterns
4. Important examples (only most illustrative)

Pages will distribute automatically based on content volume. No need to calculate character counts.

### 8. Interaction Flow
1. **Identify course**: Ask user which course to process if not specified
2. **Scan directory**: List all source files found
3. **Extract content**: Process all relevant files, prioritizing by importance
4. **Generate LaTeX**: Create structured cheat sheet
5. **Save file**: Write `.tex` file to appropriate location
6. **Compile**: Run XeLaTeX compilation
7. **Verify**: Check compilation success
8. **Report**: Provide file locations and any warnings/errors

## Template Structure
The template uses:
- `\documentclass[10pt,a4paper]{article}` for compact layout
- `geometry` package with 0.5cm margins
- `multicol` package for 3-column layout
- `amsmath`, `amssymb` for mathematical notation
- `\scriptsize` font for maximum content density

## Best Practices
- **Test compilation early**: Generate a minimal version first, then add content
- **Group related topics**: Keep Week 1 content together, algorithms together, etc.
- **Use consistent terminology**: Match the course's terminology exactly
- **Prioritize exam-relevant content**: Focus on what's testable
- **Include examples sparingly**: Only the most illustrative examples
- **Cross-reference when possible**: "See Week 3 for details" to save space
- **Avoid bullet points**: Use `\concept{}` formatting instead for better structure and readability

## Common LaTeX Pitfalls
- **Special characters**: Escape `#`, `$`, `%`, `&`, `_`, `{`, `}` with backslash
- **Math mode**: Always use `$...$` for math symbols
- **Line breaks in tables**: Use `\\` not just line breaks
- **Column width**: Tables with `m{6cm}` are already sized for 3-column layout
- **Page breaks**: Use `\newpage` to start fresh pages when sections are complete