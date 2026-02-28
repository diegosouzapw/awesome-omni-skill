---
name: Chemistry Skill
description: Provides a deep understanding of chemistry through a technical-philosophical approach, for research and education tasks. In research tasks, you will use technical and logical understanding of the subtopics of the different theoretical models of chemistry and particles, as well as taking examples from the different historical chains of thought that have guided scientific advancement in chemistry throughout human history, compiled in this skill. For education, you will lead and guide the user to understand and predict the behavior of subatomic particles and how they interact to form molecules and compounds, as well as explain the chemical processes that occur in nature and industry, taking as an example the information compiled in this skill.
licence: MIT
---

# Chemistry Skill
investigation and education in chemistry

## 1.0 Index and structure of resources
A continuation of the index and structure of the educational and research resources available, the subtopic, a brief description to understand the content and the link to the document is shown.

### 1.1 Theoretical and technical knowledge of chemistry

**Reflection** on the “indivisibility” of the atom, **basic chemical reactions**, **the nucleus**, its particles, and the relationship between the number of protons (atomic number) in defining the identity of the element and why the element depends on the number of protons (atomic number) from a logical perspective, **Electron** from a logical and philosophical perspective, and finally, *“the entire chain of thought”* that humanity had, from philosophers, scientists, and researchers throughout history that led to the discovery of the atom as we know it today. [Atom Level 1](subtopics/atom-level-1.md)

## 2.0 Chemical Formula Script
The project includes a Python script to execute chemical and physical formulas related to the skill.

### 2.1 Formula List
| Formula Name | Description | Arguments |
| :--- | :--- | :--- |
| `brownian_motion` | Calculates Einstein's Brownian Motion (Diffusion * Time) | `t` (s), `T` (K), `eta` (Pa·s), `r` (m) |

### 2.2 Agent Instructions for Script Execution
To execute a formula, use the `run_command` tool with the following syntax:
`python scripts/formulas.py [formula_name] [args...]`

**Example:**
`python scripts/formulas.py brownian_motion 10 298 0.001 0.000001`
