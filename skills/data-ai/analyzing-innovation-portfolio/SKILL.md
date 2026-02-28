---
name: analyzing-innovation-portfolio
description: Analyze the CustomGPT.ai Labs Innovation workbook and cost tracking data to surface portfolio-level insights, trends, and recommendations for where to focus Innovation efforts.
---

# Analyzing the Innovation Portfolio

You turn the Innovation Projects workbook and cost tracking sheet into a
**portfolio review** with actionable recommendations.

## When to Use

Use this skill when the user:

- Is preparing a monthly or quarterly Innovation review.
- Wants to know which projects to double down on, pivot, or stop.
- Needs to justify Innovation investment to leadership.

## Inputs

Expect:

- The latest Innovation Projects workbook (Excel or CSV exports) containing
  active projects, completed projects, and the idea backlog.
- The cost tracking sheet with roles, hours, and rates.
- Any additional notes on business impact that might not yet be in the sheet.

## Analysis Tasks

1. **Portfolio Snapshot**
   - Count projects by status (Not started, In progress, Completed, Blocked).
   - Group by owner, type, or theme where possible.

2. **Outcomes & Impact**
   - Identify projects with clear, positive impact.
   - Flag projects with weak, unclear, or missing outcomes.

3. **Cost & Resourcing**
   - Roughly allocate costs by project or category, using reasonable
     assumptions when needed.
   - Highlight overloaded or under‑used contributors.

4. **Risks & Bottlenecks**
   - Call out late, blocked, or repeatedly slipping projects.
   - Note any single‑points‑of‑failure (e.g., only one dev can maintain X).

5. **Recommendations**
   - Suggest what to double down on, pivot, pause, or kill.
   - Suggest the most promising backlog ideas in light of current learnings.

## Output Format

Produce a Markdown report with sections:

- **Portfolio Snapshot**
- **Outcomes & Impact**
- **Cost & Resourcing**
- **Risks & Bottlenecks**
- **Recommendations**

Use small tables where appropriate and clearly mark any assumptions.

## Guidelines

- Be transparent about data gaps or uncertainties.
- Focus on **choices** Felipe can make, not just descriptive stats.
- Tie recommendations back to **business outcomes**, not just activity level.
