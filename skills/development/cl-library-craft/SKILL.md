---
name: cl-library-craft
description: Analyze and generate idiomatic Common Lisp libraries following patterns from Edi Weitz, Marijn Haverbeke, and Eitaro Fukamachi
---

# CL Library Craft

A comprehensive skill for analyzing and creating idiomatic Common Lisp libraries.

## Purpose

This skill enables AI assistants to:
1. **Analyze** existing CL libraries to understand patterns and conventions
2. **Generate** new CL libraries following established best practices
3. **Reference** design patterns from exemplary CL authors

## Sub-Skills

This skill is organized into sub-skills for better context management. **Read only the sub-skills relevant to your task.**

### For Analyzing Libraries

```
analyze/SKILL.md
```

Use when: "Analyze this CL library", "What patterns does X use?", "Compare these libraries"

### For Writing Libraries

```
write/SKILL.md
```

Use when: "Create a CL library for...", "Generate a Lisp package", "Scaffold a CL project"

### For Reference Only

```
guide/          # Design pattern guides
templates/      # Code templates
analyses/       # Library analyses
```

## Quick Routing

| User Request | Sub-Skill to Read |
|--------------|-------------------|
| "Analyze the library at..." | `analyze/SKILL.md` |
| "What patterns does X use?" | `analyze/SKILL.md` + `analyses/` |
| "Create a CL library for..." | `write/SKILL.md` |
| "Write a Common Lisp package" | `write/SKILL.md` |
| "How should I structure..." | `guide/` files |
| "Show me the template for..." | `templates/` files |

## Directory Structure

```
cl-library-craft/
├── SKILL.md                    # This file - routing
├── analyze/
│   └── SKILL.md                # Library analysis sub-skill
├── write/
│   └── SKILL.md                # Library generation sub-skill
├── guide/
│   ├── 00-philosophy.md
│   ├── 01-project-structure.md
│   ├── 02-asdf-systems.md
│   ├── 03-package-design.md
│   ├── 04-api-design.md
│   ├── 05-error-handling.md
│   ├── 06-documentation.md
│   ├── 07-portability.md
│   ├── 08-testing.md
│   ├── 09-pattern-matrix.md
│   ├── 10-author-styles.md
│   └── 11-sbcl-optimizations.md
├── templates/
│   ├── edi-weitz.md            # Traditional flat style
│   ├── fukamachi.md            # Modern src/ style
│   ├── haverbeke.md            # Multi-system layered
│   ├── minimal.md              # Smallest possible
│   └── snippets.md             # Common code patterns
└── analyses/
    ├── edi-weitz/
    │   ├── hunchentoot.md
    │   ├── cl-ppcre.md
    │   ├── drakma.md
    │   └── cl-who.md
    ├── marijn-haverbeke/
    │   └── postmodern.md
    ├── fukamachi/
    │   └── libraries.md
    └── portability/
        └── libraries.md
```

## Usage Examples

### Example 1: Analyze a Library

User: "Analyze the patterns in Hunchentoot"

1. Read `analyze/SKILL.md`
2. Read `analyses/edi-weitz/hunchentoot.md`
3. Provide analysis

### Example 2: Create a New Library

User: "Create a CL library for parsing TOML files"

1. Read `write/SKILL.md`
2. Determine style (edi-weitz for parsers)
3. Read `templates/edi-weitz.md`
4. Generate files

### Example 3: Understand a Pattern

User: "How should I handle errors in CL?"

1. Read `guide/05-error-handling.md`
2. Optionally check `analyses/` for examples
3. Provide guidance

## Key Principles

These guides are built from analysis of exemplary CL libraries by:
- **Edi Weitz** - Hunchentoot, CL-PPCRE, Drakma, CL-WHO
- **Marijn Haverbeke** - Postmodern, S-SQL
- **Eitaro Fukamachi** - Mito, SXQL, Woo
- **Portability authors** - bordeaux-threads, closer-mop, ironclad

The patterns reflect real-world, battle-tested conventions from the CL community.
