---
name: academic-review
description: Interactive review sessions with academic PDFs (lectures, research papers, book chapters). Extract concepts, run Q&A sessions, generate quizzes with scoring. Preserves mathematical formulas in LaTeX format. Privacy-preserving local processing - PDFs never uploaded. Use when studying academic materials, reviewing research, or preparing for exams.
allowed-tools: Read, Glob, Bash(python:*)
---

# Academic Review Skill

## Overview

This skill enables interactive review sessions with **academic PDFs** while **preserving your privacy**. All PDF processing happens locally on your machine using the Marker library - PDFs are never sent to Anthropic servers. Only extracted text (with LaTeX formulas) is used in our conversation.

**Supported Document Types:**
- **Lecture Slides**: Review presentations, generate quizzes, Q&A on concepts
- **Research Papers**: Analyze methodology, results, and discussion sections
- **Book Chapters**: Study concepts, work through examples and exercises

**Key Features:**
- **Privacy-preserving**: PDFs processed locally, never uploaded
- **Math-aware**: Formulas preserved in LaTeX format (e.g., `$E = mc^2$`, `$$\int_a^b f(x)dx$$`)
- **Cached extraction**: First extraction is slow, subsequent access is instant
- **Two review modes**: Q&A (free-form questions) and Quiz (auto-generated questions with scoring)
- **Visual fallback**: Can extract specific pages as images for complex diagrams

## Document Type Detection

When starting a review session, **identify the document type** from context:

**Lectures** - Indicators:
- File names with "lecture", "slides", "presentation"
- Bullet-point heavy content
- Sequential slide numbers
- Course/semester codes (e.g., "CS229_Lecture05.pdf")

**Research Papers** - Indicators:
- File names with "paper", author names, conference/journal codes
- Standard sections: Abstract, Introduction, Methods, Results, Discussion, References
- Citations and bibliography
- Two-column format common

**Book Chapters** - Indicators:
- File names with "chapter", book titles
- Sections and subsections with numbered headings
- End-of-chapter exercises or problems
- Dense paragraph-based text

**Default approach**: If unclear, start with Q&A mode and adapt based on the content structure.

## Quick Start

### Starting a Review Session

When the user requests a review session:

1. **Find PDFs** using Glob: `**/*.pdf` or more specific patterns
2. **Identify document type** (lecture/paper/chapter) from filename and request
3. **Extract content** using the extraction script
4. **Ask mode preference**: "Would you like Q&A mode or Quiz mode?"
5. **Begin the selected mode** with document-type-appropriate approach

### Example Flow

```
User: "Review the SLAM paper by Smith et al."

Your actions:
1. Use Glob to find PDFs matching "smith" or "slam"
2. Identify as research paper
3. Run: python scripts/extract_pdf.py <pdf_path>
4. Read the cached markdown file
5. Ask: "Q&A mode or Quiz mode?"
6. Begin selected mode (adapt to paper structure)
```

## Review Modes

### Q&A Mode (Free-Form Questions)

**Purpose**: Answer user's specific questions about content with detailed explanations.

**How to conduct Q&A mode:**

1. **Load and parse content**:
   ```bash
   # Extract PDF (or use cached version)
   python scripts/extract_pdf.py /path/to/document.pdf
   ```
   Then read the output markdown file using the Read tool.

2. **Present overview** (adapt to document type):

   **For Lectures**:
   - List main topics covered
   - Highlight key formulas (show in LaTeX)
   - Mention important definitions or concepts

   Example:
   ```
   📚 Lecture Overview: Epipolar Geometry

   This lecture covers 45 slides on:
   - Epipolar constraint: $x'^T F x = 0$
   - Fundamental matrix $F$ (3x3, rank 2)
   - Essential matrix $E = K'^T F K$
   - Applications: stereo vision, 3D reconstruction

   Ask me anything about these topics, or say "quiz" to switch to quiz mode.
   ```

   **For Research Papers**:
   - Summarize the research question/contribution
   - Key methodology and approach
   - Main results and conclusions
   - Important formulas or algorithms

   Example:
   ```
   📄 Paper Overview: "ORB-SLAM2: Real-Time SLAM for Monocular, Stereo and RGB-D Cameras"

   **Research Question**: How to build a complete SLAM system that works across multiple camera types?

   **Key Contributions**:
   - Unified SLAM system for monocular, stereo, and RGB-D cameras
   - Place recognition and loop closing
   - Real-time performance on standard CPUs

   **Methods**: ORB features, bag-of-words place recognition, pose graph optimization

   **Results**: Evaluated on KITTI and TUM datasets, outperforms previous methods

   Ask me about methodology, results, or implementation details.
   ```

   **For Book Chapters**:
   - Main concepts introduced
   - Theorems or key results
   - Important formulas
   - Example problems covered

   Example:
   ```
   📖 Chapter Overview: "Matrix Decompositions" (Chapter 7)

   **Topics Covered**:
   - Singular Value Decomposition (SVD): $A = U\Sigma V^T$
   - Eigenvalue decomposition: $A = Q\Lambda Q^T$
   - QR decomposition and applications
   - Least squares via matrix decompositions

   **Key Theorems**: Spectral theorem, SVD existence

   **Exercises**: 15 problems on computing decompositions and applications

   Ask me about concepts, work through examples, or get help with exercises.
   ```

3. **Answer questions**:
   - Reference specific page/section numbers
   - Show formulas in LaTeX format
   - Explain concepts with examples
   - Connect related topics
   - If user asks about a diagram, offer to extract that page as an image

4. **Track progress**:
   - Note which topics user asks about
   - Identify apparent knowledge gaps
   - Suggest related concepts proactively

**Document-specific guidance:**

**Lectures**: Focus on concept understanding, derivations, applications
**Papers**: Focus on methodology critique, results interpretation, reproducibility
**Chapters**: Focus on theorem understanding, example walkthrough, exercise solving

### Quiz Mode (Auto-Generated Questions)

**Purpose**: Test user's knowledge with auto-generated questions, provide scoring and feedback.

**How to conduct Quiz mode:**

1. **Load and analyze content**:
   ```bash
   # Extract PDF (or use cached version)
   python scripts/extract_pdf.py /path/to/document.pdf
   ```
   Read the markdown and analyze:
   - Key concepts and definitions
   - Important formulas (in LaTeX)
   - Learning objectives
   - Example problems

2. **Generate questions** (default: 10, but ask user for preference):

   **For Lectures**:
   - **Multiple choice**: Test understanding of concepts
   - **True/False**: Quick concept checks
   - **Short answer**: Define terms or explain relationships
   - **Formula problems**: Apply equations to scenarios

   **For Research Papers**:
   - **Multiple choice**: Methodology choices, experimental design
   - **True/False**: Claims about results or methods
   - **Short answer**: Explain key contributions, limitations
   - **Analysis questions**: Critique methods or interpret results

   **For Book Chapters**:
   - **Multiple choice**: Theorem conditions, concept understanding
   - **True/False**: Mathematical statements
   - **Short answer**: Prove simple results, explain concepts
   - **Problems**: Similar to end-of-chapter exercises

   Mix question types and topics proportionally. Order by difficulty (easier first).

3. **Present questions one at a time**:
   ```
   Quiz Mode - 10 Questions
   Score: 0/0

   Question 1 of 10 [Multiple Choice]
   What is the rank of the Fundamental matrix $F$?

   a) 1
   b) 2
   c) 3
   d) 4

   Your answer:
   ```

4. **Evaluate and provide feedback**:
   ```
   ✓ Correct! [+1 point]

   The Fundamental matrix $F$ has rank 2, which means det($F$) = 0. This constraint
   arises from the fact that $F$ maps points to epipolar lines, and the mapping has
   a one-dimensional null space.

   Score: 1/1 (100%)

   Question 2 of 10...
   ```

   For incorrect answers:
   ```
   ✗ Incorrect [+0 points]
   Your answer: a) 1
   Correct answer: b) 2

   The Fundamental matrix has rank 2, not 1. The rank-2 constraint (det($F$) = 0)
   is one of the key properties used in estimating $F$ from point correspondences.

   Score: 1/2 (50%)

   Question 3 of 10...
   ```

5. **End with summary**:
   ```
   📊 Quiz Complete!

   Final Score: 8/10 (80%) - B

   ✓ Topics Mastered:
   - Epipolar constraint
   - Essential matrix properties
   - Stereo reconstruction basics

   ⚠️ Topics to Review:
   - Fundamental matrix estimation (8-point algorithm)
   - RANSAC for outlier rejection

   Would you like to:
   1. Review the topics you missed in Q&A mode?
   2. Take another quiz on the same material?
   3. Move to a different document?
   ```

**Scoring Guidelines:**
- Multiple choice: 1 point for correct answer
- True/False: 1 point for correct answer
- Short answer: 1 point if answer captures key concept (be flexible)
- Formula problems: 1 point for correct answer, 0.5 for correct approach but calculation error

## Finding PDFs

**General patterns:**

```bash
# All PDFs in current directory and subdirectories
glob pattern: "**/*.pdf"

# Find specific document by name
glob pattern: "**/*smith*.pdf"

# Course-specific (if organized in directories)
glob pattern: "CS229/**/*.pdf"
```

**When user's request is ambiguous:**
1. Use Glob to find matching PDFs
2. Present options if multiple matches
3. Let user select which PDF to review

## Extraction and Caching

### First-Time Extraction

When extracting a PDF for the first time:

```bash
# Run extraction script (uses venv python)
source .venv/bin/activate && python scripts/extract_pdf.py /path/to/document.pdf
```

This will:
- Convert PDF to markdown using Marker
- Preserve formulas as LaTeX
- Cache result in `.cache/extracted/<hash>.md`
- Save metadata in `.cache/extracted/<hash>.json`
- Take 30-60 seconds depending on PDF size

The script prints the path to the cached markdown file - use Read to load it.

### Using Cached Extraction

If the PDF was previously extracted:
- Script immediately returns cached file path
- Read the markdown file using Read tool
- Instant access (no re-processing)

### Extracting Page Images (Fallback)

If user asks about a specific diagram or visual element:

```bash
# Extract page 23 as an image
source .venv/bin/activate && python scripts/extract_page_image.py /path/to/document.pdf 23
```

Then use Read to view the image and analyze it visually.

## Working with LaTeX Formulas

**Extracted formulas are in LaTeX format:**

Inline: `$E = mc^2$`
Display: `$$\int_a^b f(x)dx$$`

**When explaining formulas:**
- Show the LaTeX notation
- Explain each variable
- Provide context from the document
- Give examples if helpful

**Example:**
```
The epipolar constraint is expressed as:

$$x'^T F x = 0$$

Where:
- $x$ and $x'$ are corresponding points in homogeneous coordinates
- $F$ is the 3×3 Fundamental matrix
- The equation states that $x'$ lies on the epipolar line $Fx$ in the second image
```

## Tips for Best Results

**General Guidelines:**
1. **Always extract first**: Run the extraction script before answering questions
2. **Use caching**: Check if extraction already exists (script handles this)
3. **Reference pages**: Include page numbers when answering
4. **Show formulas**: Display LaTeX formulas when explaining math concepts
5. **Be interactive**: Ask follow-up questions, offer deeper explanations
6. **Adapt to document type**: Use appropriate review style (lectures vs papers vs chapters)

**For Math-Heavy Content:**
- Formulas are preserved in LaTeX - use them!
- Explain notation and variables clearly
- Show step-by-step derivations when helpful
- Offer to extract page images for complex diagrams

**For Research Papers:**
- Focus on understanding methodology and contributions
- Help interpret results and figures
- Discuss limitations and future work
- Compare with related work when relevant

**For Book Chapters:**
- Work through examples step-by-step
- Help with end-of-chapter exercises
- Connect concepts across chapters
- Prove theorems when requested

**For Multi-PDF Sessions:**
- Can review multiple documents in one session
- Cross-reference concepts between documents
- Build connections across topics

**Mode Switching:**
- User can switch from Q&A to Quiz (or vice versa) anytime
- Just ask and switch modes
- Keep the extracted content loaded

## Session Examples

### Lecture Review Session

```
User: "Quiz me on the SLAM lecture"

Your actions:
1. glob pattern: "**/*slam*.pdf"
2. Find matching PDF (e.g., "Lecture_12_SLAM.pdf")
3. Identify as lecture (filename, slide structure)
4. source .venv/bin/activate && python scripts/extract_pdf.py Lecture_12_SLAM.pdf
5. Read cached markdown
6. Generate 10 questions covering SLAM topics
7. Start quiz mode
```

### Research Paper Review Session

```
User: "Help me understand the ORB-SLAM2 paper"

Your actions:
1. glob pattern: "**/*orb*slam*.pdf"
2. Find matching PDF
3. Identify as research paper (structure, citations)
4. source .venv/bin/activate && python scripts/extract_pdf.py orb_slam2.pdf
5. Read cached markdown
6. Present paper overview (research question, methods, results)
7. Enter Q&A mode - focus on methodology and results interpretation
```

### Book Chapter Review Session

```
User: "Review chapter 7 on matrix decompositions"

Your actions:
1. glob pattern: "**/*chapter*7*.pdf" or "**/*matrix*.pdf"
2. Find matching PDF
3. Identify as book chapter (numbered sections, exercises)
4. source .venv/bin/activate && python scripts/extract_pdf.py chapter_07_decompositions.pdf
5. Read cached markdown
6. Present chapter overview (concepts, theorems, exercises)
7. Ask: "Q&A mode or Quiz mode?"
8. If Q&A: Help with concepts and exercises
9. If Quiz: Generate problems similar to exercises
```

### Switching Modes Mid-Session

```
[In Q&A mode]
User: "Actually, can you quiz me instead?"

Your response:
"Sure! I'll generate a quiz based on this content. How many questions would you like? (default: 10)"

User: "10 is fine"

Your response:
[Generate 10 questions and start quiz mode]
```

### Using Page Images

```
User: "I don't understand the diagram on page 15"

Your actions:
1. source .venv/bin/activate && python scripts/extract_page_image.py /path/to/document.pdf 15
2. Read the image file
3. Analyze the diagram visually
4. Explain what it shows, referencing specific elements

Your response:
"Let me extract that diagram for you..."
[After reading image]
"This diagram shows the epipolar geometry configuration. I can see two cameras (left and right) viewing a 3D point P. The key elements are:
- Point P in 3D space
- Its projections p and p' in the two images
- The baseline connecting camera centers C and C'
- The epipolar plane (gray triangle)
- Epipolar lines l and l' in each image

Would you like me to explain how these elements relate to the Fundamental matrix?"
```

## Error Handling

**If extraction fails:**
1. Check if PDF file exists
2. Ensure dependencies are installed (`pip list | grep marker`)
3. Check file permissions
4. Report error to user with helpful message

**If formula extraction is unclear:**
1. Show what was extracted
2. Offer to extract the page as an image
3. Analyze the formula visually from the image

**If no PDFs found:**
1. Double-check the glob pattern
2. Ask user for the PDF file path
3. Clarify which document they want to review

## Privacy Reminder

Always remember: **PDFs are processed locally**. Only extracted markdown text (with LaTeX formulas) is sent to Claude. The original PDFs never leave the user's machine. This ensures privacy for proprietary or sensitive academic materials.

---

For detailed documentation, see [reference.md](reference.md).
For usage examples, see [examples.md](examples.md).
