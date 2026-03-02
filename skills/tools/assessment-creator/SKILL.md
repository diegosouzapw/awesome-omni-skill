---
name: assessment-creator
description: Creates comprehensive assessment tools, rubrics, and evaluation materials for student work. Use when the user asks to create rubrics, build assessments, evaluate student learning, or needs grading criteria. Generates clear, fair assessment tools with examples.
allowed-tools: Read, Write, Edit, Glob, Task
---

# Assessment Creator Skill

This skill creates comprehensive, fair assessment tools for evaluating student learning in AI/prompt engineering.

## When to Use

Use this skill when the user:
- Asks to create a rubric or assessment tool
- Wants to evaluate student work
- Needs grading criteria for assignments
- Requests self-assessment tools for students
- Wants to measure learning objectives
- Needs portfolio or project evaluation criteria

## What This Skill Does

1. **Gathers Requirements**:
   - What to assess (assignment, skill, project, etc.)
   - Learning objectives being measured
   - Target grade level(s)
   - Type of assessment (rubric, self-assessment, checklist, etc.)
   - Grading scale or performance levels needed

2. **Invokes the assessment-builder Subagent** to create:
   - Detailed assessment rubric with clear criteria
   - 4-level performance scale (Developing, Proficient, Advanced, Exemplary)
   - Examples at each performance level
   - Student self-assessment component
   - Reflection questions
   - Success examples and common pitfalls
   - Actionable feedback guidance

3. **Saves Assessment** to:
   - `assessments/assignment-name-rubric.md`
   - OR within class folder if class-specific

4. **Creates Additional Materials**:
   - Student-facing version (simplified language)
   - Grade tracking template if requested
   - Feedback sentence starters

## Assessment Philosophy

- **Focus on Application**: Can students USE skills, not just explain them?
- **Authentic Tasks**: Real homework and real-world problems
- **Growth Mindset**: Emphasize improvement over perfection
- **Clear Criteria**: Observable and measurable standards
- **Student-Friendly**: Students understand criteria without explanation

## Output Structure

Assessments include:
- **Learning Objectives**: What's being measured and why
- **Clear Criteria**: Specific skills or qualities
- **Performance Levels**: 4-level scale with descriptions
- **Examples**: Concrete work samples at each level
- **Success Indicators**: What good looks like
- **Common Pitfalls**: What to avoid and how to improve
- **Reflection Questions**: For student self-assessment

## Example Usage

User: "Create a rubric for evaluating students' prompt engineering skills"

This skill will generate a comprehensive, fair rubric with clear criteria and examples.
