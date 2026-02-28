---
name: analysis-spec-builder
description: Build and iteratively refine physics analysis specifications using analysis-specification-template.md. Use when the user asks to create or update an analysis spec, requests plots/histograms for a dataset, or describes a quick analysis task that should be formalized into a specification document.
---

# Analysis Spec Builder

## Workflow

1. Read the template from `./assets/analysis-specification-template.md` (relative to the this file, in the `analysis-spec-building` sub directory).
2. Draft a filled-in specification by replacing all `{{...}}` placeholders with concrete content derived from the user's request.
3. Ask focused follow-up questions for any missing critical details (datasets, selections, histogram definitions, workflow steps, tooling). Keep questions short and grouped by section.
4. Iterate revisions until the user confirms the specification is correct.
5. Write the final content to `specification.md` unless the user provides a different path.

## Drafting Rules

- Replace every `{{...}}` placeholder with real content; do not leave placeholders in the final spec.
- Keep the tone concise and technical; preserve the template section order.
- If something is unknown, propose a reasonable default and mark it as a question in the next response (do not leave placeholders).
- Preserve any explicit dataset identifiers or analysis names verbatim. Assume the user will use PHYSLITE unless told otherwise, or you know the data isn't available there.
- Use ASCII-only text unless the template already uses non-ASCII (e.g., LaTeX labels inside backticks).
- This is a specification, not a step-by-step plan for implementation of the analysis or plot.
- Details, exactly what version of objects or containers, are not included here. When detailed plans are developed from here, they will be determined. On the other hand, if the user specifies them, then they should be explicitly mentioned.
- The tools section: by default use `servicex` with `func_adl` to fetch data from the source datasets, and `awkward`, `hist`, and `vector` for manipulation and histogram fitting. Histograms should be stored as PNG by default. For statistical analysis (if needed) use cabinetry and pyhf.

## Follow-up Questions

- Ask only for details needed to proceed (e.g., missing datasets, histogram axes, selections).
- Prefer yes/no or short-answer questions.
- When the user provides new details, update only the affected sections and summarize what changed.

## Output

- Provide the full draft spec in the response until the user says it is final.
- On confirmation, write the specification to the requested path (default: `specification.md`) and confirm where it was written.
