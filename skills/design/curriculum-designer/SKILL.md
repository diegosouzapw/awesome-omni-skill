---
name: curriculum-designer
description: Helps teachers and instructional designers create a structured curriculum or teaching plan from a list of topics or a specification document. Use when the user wants to design course materials, lesson plans, or a syllabus.
---

# Curriculum Designer

An expert instructional designer that helps you create customized, modern curricula with rich content suggestions and hands-on implementation support.

**Platform Compatibility:** Works with both **Gemini CLI** and **Claude Code**.

---

## Phase 0: Detect Mode (New vs. Existing Materials)

Before starting, determine if the user is creating from scratch or enhancing existing materials.

**For Claude Code**, use `AskUserQuestion`:
```
Question: "What would you like to do?"
Options:
- Create new curriculum from scratch
- Enhance existing course materials
- Update outdated content with latest practices
- Add new delivery format to existing curriculum
```

---

## Mode A: Enhancing Existing Materials

When users have existing course materials, the skill becomes an **upgrade assistant**.

### A.1 Analyze Existing Materials

1. **Request the materials:**
   - Ask user to point to files: syllabus, slides, notes, etc.
   - Use `Read` to analyze each document

2. **Generate an Analysis Report:**

```markdown
## Course Analysis: [Course Name]

### Current Structure
- **Modules:** [count] modules identified
- **Format:** [lecture/lab/mixed]
- **Depth:** [introductory/intermediate/advanced]

### Content Inventory
| Module | Topics Covered | Last Updated | Status |
|--------|---------------|--------------|--------|
| 1. [Name] | [topics] | [date if known] | ✅ Current / ⚠️ Needs update / ❌ Outdated |

### Gaps Identified
- ❌ Missing: [topic that should be covered]
- ⚠️ Outdated: [topic using old tools/methods]
- 💡 Opportunity: [trending topic to add]

### Strengths
- ✅ [What's working well]
- ✅ [Strong coverage area]
```

### A.2 Research Updates

**For Claude Code**, use `WebSearch` to find:
- What has changed in this field since the materials were created
- New tools, frameworks, or methodologies now standard
- Updated best practices or deprecated approaches
- Fresh case studies and examples

Present a **"What's New"** summary:

```markdown
## What's New in [Topic] (2024-2025)

### Breaking Changes
- ⚠️ [Old approach] is now deprecated; use [new approach]
- ⚠️ [Tool] has been replaced by [new tool]

### New Additions to Consider
- 🆕 [New concept] - now considered essential
- 🆕 [New tool] - industry adoption at [X]%
- 🆕 [New methodology] - improves [outcome]

### Updated Examples
- Replace [old case study] with [new relevant example]
- Add [recent industry incident] as teachable moment

### Resources Refresh
- 📚 New recommended textbook: [title]
- 🎥 Quality video series: [name]
- 🔧 Updated tool documentation: [link context]
```

### A.3 Propose Enhancements

**For Claude Code**, use `AskUserQuestion` with `multiSelect: true`:
```
Question: "Which enhancements would you like to apply?"
Options:
- Update outdated content with current practices
- Add new modules for emerging topics
- Refresh examples and case studies
- Add new delivery format (slides, web app, etc.)
- Enhance assessments and exercises
- Add interactive/hands-on components
```

### A.4 Targeted Updates

For each selected enhancement:
1. Show the specific changes proposed (before/after)
2. Get approval before modifying
3. Preserve the user's existing structure and voice
4. Track changes in a `changelog.md`

```markdown
## Changelog: [Course Name]

### [Date] - Enhancement Session

#### Content Updates
- **Module 3:** Updated "REST APIs" section to include GraphQL comparison
- **Module 5:** Replaced jQuery examples with modern vanilla JS

#### New Additions
- **Module 7 (NEW):** "Containerization with Docker" - 3 lessons
- **Lab 4.2 (NEW):** Hands-on Kubernetes deployment exercise

#### Removed/Deprecated
- Removed references to deprecated library X
- Archived old Flash-based interactive demos

#### Delivery Additions
- Generated slide deck for Module 1-3
- Created Jupyter notebook version of Labs
```

---

## Mode B: Creating New Curriculum

*(This is the original flow, now labeled as Mode B)*

## Phase 1: Course Vision & Setup

### 1.1 Understand the Goal

**For Claude Code**, use `AskUserQuestion` with multiple questions:

```
Questions:
1. "What is the main topic or subject area?" (free text via Other)
2. "Who is your target audience?"
   - Beginners (no prior knowledge)
   - Intermediate (some background)
   - Advanced (experienced practitioners)
   - Mixed levels
3. "What course format works best?"
   - University Semester (15 weeks)
   - Intensive Bootcamp (4-8 weeks)
   - Workshop Series (multiple sessions)
   - Self-Paced Online Course
4. "What's your primary teaching philosophy?"
   - Project-Based (learn by building)
   - Theory-First (concepts then application)
   - Problem-Based (real-world challenges)
   - Hands-On Labs (guided exercises)
```

### 1.2 Research & Suggest Modern Content

**For Claude Code**, use `WebSearch` to find:
- Latest trends and developments in the field
- Popular tools, frameworks, and technologies
- Industry certifications or standards
- Highly-rated courses, books, or resources
- Recent case studies or real-world applications

Present a **"Topic Landscape"** summary:

```markdown
## Topic Landscape: [Topic Name]

### Current Trends
- [Trend 1 with brief explanation]
- [Trend 2 with brief explanation]

### Essential Tools & Technologies
- [Tool 1] - [why it matters]
- [Tool 2] - [why it matters]

### Recommended Resources
- [Book/Course 1]
- [Book/Course 2]

### Industry Context
- [How this topic is used in practice]
```

### 1.3 Establish Output Files

- `curriculum.md` - The main teaching plan
- `delivery-plan.md` - Technical implementation roadmap
- `assessments.md` - All assessments and rubrics
- `resources.md` - Curated list of resources

---

## Phase 2: Curriculum Design (Per Module)

### 2.1 Smart Module Analysis

Present a **complete Module Proposal**:

```markdown
## Module: [Topic Name]

**Prerequisites** (based on your audience level):
- ✅ [Concept 1] - Include as lesson
- 🔄 [Concept 2] - Quick recap
- ⏭️ [Concept 3] - Skip (already known)

**Core Concepts** (recommended coverage):
- ⭐ [Concept A] - EMPHASIZE
- 📘 [Concept B] - Cover normally
- 💡 [Concept C] - Optional deep-dive

**Suggested Lessons:**
1. Lesson Title - description
2. Lesson Title - description

**Suggested Activities:**
- 🛠️ [Hands-on Lab idea]
- 🎮 [Interactive exercise idea]
- 👥 [Group activity idea]

**Modern Additions** (from research):
- Consider adding [new tool/technique]
- Include [case study] as example
```

**For Claude Code**, use `AskUserQuestion`:
- Options: **Approve as-is** / **Modify prerequisites** / **Modify concepts** / **Modify lessons** / **Start over**

### 2.2 Generate Module Content

```markdown
# Module X: [Title]

## Learning Objectives
By the end of this module, students will be able to:
- [Objective 1 - using Bloom's taxonomy verbs]
- [Objective 2]

## Lesson 1: [Title]

### Overview
[2-3 paragraph introduction]

### Key Concepts
1. **[Concept Name]**
   - Definition: ...
   - Why it matters: ...
   - Example: ...

### Hands-On Exercise
**Exercise 1.1: [Title]**
- Objective: ...
- Steps: ...
- Expected outcome: ...

### Discussion Questions
1. [Question for class discussion]
```

---

## Phase 3: Assessment Design

Assessments are critical for measuring learning outcomes. Design them intentionally.

### 3.1 Assessment Strategy

**For Claude Code**, use `AskUserQuestion`:
```
Question: "What assessment approaches do you want to use?"
Options (multiSelect: true):
- Quizzes (knowledge checks)
- Coding/Practical Labs (hands-on skills)
- Projects (applied learning)
- Written Assignments (analysis/reflection)
- Peer Reviews (collaborative assessment)
- Oral Presentations (communication skills)
- Portfolio (cumulative demonstration)
```

### 3.2 Assessment Templates

Generate assessments based on selection. Write all to `assessments.md`.

#### Quiz Template

```markdown
# Quiz: [Module Name]

**Duration:** [X] minutes
**Total Points:** [X]
**Passing Score:** [X]%

## Section A: Multiple Choice (X points each)

### Question 1
[Question text]

- A) [Option]
- B) [Option]
- C) [Option] ✓
- D) [Option]

**Explanation:** [Why C is correct]

---

### Question 2
[Continue pattern...]

## Section B: Short Answer (X points each)

### Question 5
[Question requiring brief written response]

**Expected Answer:** [Key points to cover]
**Rubric:**
- Full credit: [criteria]
- Partial credit: [criteria]
- No credit: [criteria]

## Section C: Code Analysis (X points)

### Question 8
Given the following code:
```[language]
[code snippet]
```

a) What is the output? (2 pts)
b) Identify the bug and fix it. (3 pts)
c) What is the time complexity? (2 pts)

**Answer Key:**
a) [answer]
b) [answer with fixed code]
c) [answer with explanation]
```

#### Lab/Practical Assessment Template

```markdown
# Lab Assessment: [Title]

**Module:** [X]
**Duration:** [X] hours
**Points:** [X]

## Objective
[What students will build/accomplish]

## Prerequisites
- [Required knowledge]
- [Required tools/setup]

## Starter Code/Resources
- [Link or embed starter files]

## Requirements

### Part 1: [Component] (X points)
- [ ] Requirement 1.1
- [ ] Requirement 1.2
- [ ] Requirement 1.3

### Part 2: [Component] (X points)
- [ ] Requirement 2.1
- [ ] Requirement 2.2

### Bonus Challenge (+X points)
- [ ] [Extra credit task]

## Submission
- Submit via [method]
- Include [required files]
- Due: [deadline]

## Grading Rubric

| Criterion | Excellent (100%) | Good (80%) | Satisfactory (60%) | Needs Work (40%) |
|-----------|-----------------|------------|-------------------|------------------|
| **Functionality** | All features work correctly | Minor issues | Major feature missing | Does not run |
| **Code Quality** | Clean, well-documented | Mostly clean | Some issues | Hard to read |
| **Requirements** | All met | Most met | Some met | Few met |

## Sample Solution
[Provide or link to reference implementation]
```

#### Project Assessment Template

```markdown
# Project: [Title]

**Modules Covered:** [X, Y, Z]
**Duration:** [X] weeks
**Total Points:** [X]
**Team Size:** [Individual / X members]

## Project Overview
[Description of what students will build]

## Learning Outcomes Assessed
- [LO1]: [How this project assesses it]
- [LO2]: [How this project assesses it]

## Milestones

### Milestone 1: [Name] (Week X) - X points
**Deliverables:**
- [ ] [Deliverable 1]
- [ ] [Deliverable 2]

**Check-in Questions:**
1. [Question to verify understanding]

### Milestone 2: [Name] (Week X) - X points
[Continue pattern...]

### Final Submission (Week X) - X points
**Deliverables:**
- [ ] Working application/solution
- [ ] Documentation (README, comments)
- [ ] Reflection/writeup (500 words)
- [ ] Demo video (3-5 minutes)

## Grading Rubric

### Technical Implementation (40%)
| Score | Criteria |
|-------|----------|
| A | Exceeds requirements, elegant solution, handles edge cases |
| B | Meets all requirements, works correctly |
| C | Meets most requirements, minor bugs |
| D | Partially complete, significant issues |
| F | Does not meet minimum requirements |

### Code Quality (20%)
| Score | Criteria |
|-------|----------|
| A | Exceptional organization, thorough documentation |
| B | Well-organized, good documentation |
| C | Adequate organization, some documentation |
| D | Disorganized, minimal documentation |
| F | No discernible structure |

### Innovation & Creativity (20%)
[Similar rubric...]

### Presentation & Communication (20%)
[Similar rubric...]

## Academic Integrity
[Collaboration policy, citation requirements]
```

#### Peer Review Template

```markdown
# Peer Review: [Assignment Name]

**Reviewer:** _______________
**Author:** _______________
**Date:** _______________

## Review Checklist

### Functionality
- [ ] Code runs without errors
- [ ] All requirements are met
- [ ] Edge cases are handled

### Code Quality
- [ ] Code is readable and well-formatted
- [ ] Variables/functions have clear names
- [ ] Comments explain complex logic

### Design
- [ ] Solution is appropriate for the problem
- [ ] No unnecessary complexity
- [ ] Follows best practices discussed in class

## Detailed Feedback

### Strengths (What works well)
1.
2.
3.

### Areas for Improvement
1. [Issue]: [Specific suggestion]
2. [Issue]: [Specific suggestion]

### Questions for the Author
1.

## Rating
Overall Quality: ☐ Excellent  ☐ Good  ☐ Satisfactory  ☐ Needs Work

## Reviewer Reflection
What did you learn from reviewing this work?
```

### 3.3 Assessment Calendar

Generate a timeline integrating assessments:

```markdown
## Assessment Calendar

| Week | Module | Assessment | Type | Weight |
|------|--------|------------|------|--------|
| 2 | 1 | Quiz 1 | Knowledge check | 5% |
| 3 | 1-2 | Lab 1 | Practical | 10% |
| 4 | 2 | Quiz 2 | Knowledge check | 5% |
| 6 | 1-3 | Midterm Project | Applied | 20% |
| ... | ... | ... | ... | ... |
| 15 | All | Final Project | Capstone | 30% |

**Grade Breakdown:**
- Quizzes: 20%
- Labs: 20%
- Projects: 50%
- Participation: 10%
```

---

## Phase 4: Delivery & Implementation

### 4.1 Choose Delivery Format

**For Claude Code**, use `AskUserQuestion` with `multiSelect: true`:
```
Question: "Which delivery formats do you need?"
Options:
- Interactive Web App
- Slide Decks
- Jupyter Notebooks
- LMS Package (Canvas/Moodle)
- Video Course Scripts
- Printed Worksheets/Handouts
- GitHub Repository (code + docs)
```

### 4.2 Delivery Templates

Based on selection, generate complete templates:

---

#### Template: Slide Deck (Reveal.js / Markdown)

```markdown
---
title: "[Module Title]"
theme: default
transition: slide
---

# [Module Title]

## [Course Name]

Instructor: [Name]

---

## Learning Objectives

By the end of this session, you will be able to:

1. [Objective 1]
2. [Objective 2]
3. [Objective 3]

---

## Agenda

1. [Topic 1] (15 min)
2. [Topic 2] (20 min)
3. Hands-on Exercise (25 min)
4. Wrap-up & Q&A (10 min)

---

## [Topic 1]: [Title]

### Key Concept

[Concise explanation - max 3 bullet points]

- Point 1
- Point 2
- Point 3

---

## [Topic 1]: Visual

![Diagram description](path/to/diagram.png)

[Speaker notes: Explain the diagram...]

---

## [Topic 1]: Example

```[language]
// Code example with comments
[code]
```

**What this demonstrates:** [explanation]

---

## 🛠️ Hands-on Exercise

**Task:** [What students will do]

**Time:** 15 minutes

**Instructions:**
1. [Step 1]
2. [Step 2]
3. [Step 3]

---

## Exercise: Solution

```[language]
[solution code]
```

---

## Key Takeaways

✅ [Takeaway 1]

✅ [Takeaway 2]

✅ [Takeaway 3]

---

## Next Session Preview

**[Next Module Title]**

We'll cover:
- [Preview topic 1]
- [Preview topic 2]

**Preparation:** [Any pre-work]

---

## Questions?

[Contact info / Office hours / Discussion forum]
```

---

#### Template: Jupyter Notebook

```python
# Cell 1: Markdown - Title
"""
# Module X: [Title]

**Course:** [Course Name]
**Duration:** [X] hours
**Prerequisites:** [List]

---

## Learning Objectives

By completing this notebook, you will:
1. [Objective 1]
2. [Objective 2]

## Setup

Run the cell below to import required libraries.
"""

# Cell 2: Code - Setup
"""
# Required imports
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

# Configuration
%matplotlib inline
plt.style.use('seaborn')

print("✅ Setup complete!")
"""

# Cell 3: Markdown - Section 1
"""
---
## 1. [Topic Title]

[Explanation of the concept - 2-3 paragraphs]

### Key Points:
- Point 1
- Point 2
"""

# Cell 4: Code - Demonstration
"""
# Demonstration: [What this shows]
[code with inline comments]
"""

# Cell 5: Markdown - Exercise
"""
---
### 🎯 Exercise 1.1: [Title]

**Task:** [Description]

**Instructions:**
1. [Step 1]
2. [Step 2]

**Hint:** [Optional hint]
"""

# Cell 6: Code - Exercise (student completes)
"""
# YOUR CODE HERE
# [Starter code or scaffolding]


# Test your solution
assert [test condition], "Error message"
print("✅ Exercise 1.1 passed!")
"""

# Cell 7: Markdown - Solution (hidden by default)
"""
<details>
<summary>Click to reveal solution</summary>

```python
[solution code]
```

**Explanation:** [Why this works]

</details>
"""

# Continue pattern for remaining sections...

# Final Cell: Markdown - Summary
"""
---
## Summary

In this notebook, you learned:

| Concept | Key Takeaway |
|---------|--------------|
| [Concept 1] | [Takeaway] |
| [Concept 2] | [Takeaway] |

## Next Steps

- [ ] Complete the [assignment/quiz]
- [ ] Read [resource]
- [ ] Try the bonus challenges below

### Bonus Challenges

1. [Advanced challenge 1]
2. [Advanced challenge 2]
"""
```

---

#### Template: LMS Package Structure

```
course-export/
├── imsmanifest.xml          # SCORM/LTI manifest
├── modules/
│   ├── module-01/
│   │   ├── overview.html    # Module landing page
│   │   ├── lesson-01.html   # Lesson content
│   │   ├── lesson-02.html
│   │   ├── quiz.xml         # QTI format quiz
│   │   └── assets/
│   │       ├── diagram-01.png
│   │       └── video-01.mp4
│   └── module-02/
│       └── ...
├── assignments/
│   ├── lab-01.html
│   └── project-01.html
├── resources/
│   ├── syllabus.html
│   ├── calendar.html
│   └── reading-list.html
└── gradebook/
    └── rubrics.xml
```

**Canvas-specific structure:**
```
canvas-export/
├── course_settings/
│   └── canvas_export.txt
├── [module-uuid]/
│   ├── [content-uuid].html
│   └── [quiz-uuid].xml
└── web_resources/
    └── [assets]
```

---

#### Template: Video Course Script

```markdown
# Video Script: [Lesson Title]

**Module:** [X]
**Video:** [Y] of [Total]
**Target Duration:** [X:XX]

---

## Pre-Production

**Equipment needed:**
- [ ] Screen recording software
- [ ] Microphone
- [ ] [Demo environment/files]

**Assets to prepare:**
- [ ] Slide deck (slides X-Y)
- [ ] Code samples in [IDE]
- [ ] [Diagram/visual]

---

## Script

### [0:00-0:30] Hook & Intro

**VISUAL:** [Title card / Instructor on camera]

**SCRIPT:**
"Welcome back to [Course Name]. In this video, we're going to [main topic]. By the end, you'll be able to [key outcome]."

"If you haven't already, make sure you've [prerequisite]. Let's dive in."

---

### [0:30-3:00] Concept Explanation

**VISUAL:** [Slides / Diagram animation]

**SCRIPT:**
"[Concept] is [definition]. Think of it like [analogy].

Here's why this matters: [relevance to real world].

Let me show you how this works..."

**B-ROLL:** [Suggested supplementary footage]

---

### [3:00-7:00] Demonstration

**VISUAL:** [Screen recording - IDE/terminal]

**SCRIPT:**
"Let's see this in action. I'm going to [describe what you're doing as you do it].

First, [step 1]... Notice how [observation].

Now [step 2]... This is where [insight]."

**CALLOUTS:** [Arrow pointing to X at 4:32]

---

### [7:00-8:30] Practice Prompt

**VISUAL:** [Exercise slide / Split screen with instructions]

**SCRIPT:**
"Now it's your turn. Pause the video and try [exercise].

Here's what you should do:
1. [Step 1]
2. [Step 2]
3. [Step 3]

I'll wait... [pause 3 seconds]

Ready? Let's see the solution."

---

### [8:30-9:30] Solution & Explanation

**VISUAL:** [Screen recording of solution]

**SCRIPT:**
"Here's how I approached it: [walk through solution].

A common mistake here is [pitfall]. If you did [alternative], that also works because [explanation]."

---

### [9:30-10:00] Wrap-up & Next Steps

**VISUAL:** [Summary slide / Instructor on camera]

**SCRIPT:**
"To recap: [key takeaway 1], [key takeaway 2], and [key takeaway 3].

In the next video, we'll cover [preview]. See you there!"

**END CARD:** [Subscribe / Next video / Resources link]

---

## Post-Production Notes

**Editing cues:**
- [Timestamp]: Add zoom effect on [element]
- [Timestamp]: Insert [graphic/animation]
- [Timestamp]: Add caption for [term]

**Chapters:**
- 0:00 Intro
- 0:30 What is [Concept]?
- 3:00 Demo
- 7:00 Your Turn
- 8:30 Solution
- 9:30 Wrap-up
```

---

#### Template: GitHub Repository Structure

```
course-repo/
├── README.md                 # Course overview, setup instructions
├── SYLLABUS.md              # Full syllabus
├── CONTRIBUTING.md          # For TAs/contributors
│
├── modules/
│   ├── 01-[module-name]/
│   │   ├── README.md        # Module overview
│   │   ├── slides/          # Slide sources
│   │   ├── notes/           # Lecture notes
│   │   ├── code/            # Demo code
│   │   └── exercises/       # Practice problems
│   │       ├── exercise-01/
│   │       │   ├── README.md
│   │       │   ├── starter/
│   │       │   └── solution/
│   │       └── exercise-02/
│   └── 02-[module-name]/
│       └── ...
│
├── labs/
│   ├── lab-01/
│   │   ├── README.md        # Instructions
│   │   ├── starter/         # Starter code
│   │   ├── tests/           # Autograder tests
│   │   └── solution/        # (private branch)
│   └── lab-02/
│
├── projects/
│   ├── project-01/
│   │   ├── README.md
│   │   ├── rubric.md
│   │   └── examples/
│   └── final-project/
│
├── resources/
│   ├── cheatsheets/
│   ├── references/
│   └── tools-setup.md
│
└── .github/
    ├── workflows/           # CI for autograding
    └── ISSUE_TEMPLATE/      # For student questions
```

**README.md template:**

```markdown
# [Course Name]

[One-line description]

## Quick Start

1. Clone this repository
2. [Setup instructions]
3. Start with [first module link]

## Course Structure

| Module | Topic | Duration |
|--------|-------|----------|
| 1 | [Topic] | [X] hours |
| 2 | [Topic] | [X] hours |
| ... | ... | ... |

## Prerequisites

- [Prerequisite 1]
- [Prerequisite 2]

## Getting Help

- [Discussion forum link]
- [Office hours info]
- Open an issue in this repo

## License

[License info]
```

---

### 4.3 Implementation Assistance

**For Claude Code**, use `AskUserQuestion`:
```
Question: "Would you like me to help implement these?"
Options:
- Yes, create the full project structure
- Yes, generate content for one module first
- Yes, set up the delivery platform config
- No, the templates are enough
```

If implementation requested:
1. Create actual files in the project
2. Generate working code, not placeholders
3. Include clear comments and documentation
4. Provide setup/deployment instructions

---

## Pedagogical Reference

| Pattern | When to Use |
|---------|-------------|
| **Problem-First** | New complex topics - show the "why" before the "how" |
| **Theory-to-Practice** | Formal topics - definitions then applications |
| **Scaffolding** | Always - simple to complex, layer by layer |
| **Recap & Refresh** | Start of each module - connect to prior knowledge |

See `references/pedagogy.md` for detailed examples.

---

## Quick Reference: Claude Code Tools

| Step | Tool | Usage |
|------|------|-------|
| Detect mode | `AskUserQuestion` | New vs. existing materials |
| Read materials | `Read` | Analyze existing course files |
| Research updates | `WebSearch` | Find latest trends, tools |
| Gather requirements | `AskUserQuestion` | Audience, format, preferences |
| Write curriculum | `Write` / `Edit` | Generate all output files |
| Implement delivery | `Write` / `Edit` | Create project structure |
| Explore codebase | `Glob` / `Grep` | Find patterns to match |
