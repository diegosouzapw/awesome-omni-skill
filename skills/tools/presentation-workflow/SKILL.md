---
name: presentation-workflow
description: A skill for creating, updating, and managing slide presentations. Use for tasks involving slide decks, speaker notes, and integrating presentation materials with Git.
---

# Presentation Workflow

This skill provides a structured workflow for creating and managing slide presentations, from initial concept to final delivery and version control.

## Core Principles

- **Governance First:** All presentation artifacts are version-controlled and auditable.
- **Proof Over Promise:** Slides are built on verifiable data and evidence.
- **Quiet Execution:** The process is automated and efficient, speaking through the quality of the output.

## Workflow Phases

This skill follows a five-phase workflow:

1.  **Initialization:** Create the slide deck structure.
2.  **Content Generation:** Create the individual slides.
3.  **Speaker Notes:** Generate the presentation script.
4.  **Final Presentation:** Assemble and present the final deck.
5.  **Version Control:** Commit all artifacts to Git.

### Phase 1: Initialization

Use `slide_initialize` to create the project structure and outline. If the user provides a detailed outline, use it. Otherwise, create a standard structure:

- Title Slide
- Problem Statement
- Proposed Solution
- Architecture/Methodology
- Key Features/Differentiators
- Market/Use Cases
- Financials/Projections (if applicable)
- Next Steps/Call to Action

### Phase 2: Content Generation

Use `slide_edit` to create each slide. Adhere to the following principles:

- **Visual Consistency:** Maintain a consistent design language (colors, fonts, layout).
- **Data-Driven:** Use charts and tables to present data. Never fabricate information.
- **Stark & Minimal:** Focus on clarity and impact. Avoid decorative elements.

### Phase 3: Speaker Notes

Use `slide_edit_notes` to generate the presentation script. The notes should be:

- **Concise:** Under 200 words per slide.
- **Authoritative:** Clear, direct, and confident.
- **Structured:** Use talking points for natural delivery.

### Phase 4: Final Presentation

Use `slide_present` to assemble the final slide deck. This tool generates the final output and provides the presentation link.

### Phase 5: Version Control

Use the `shell` tool to commit all presentation artifacts to a Git repository. This includes:

- The presentation project directory
- The final presentation notes (`slide_notes.md`)
- Any supporting assets (images, data files)

Create a new, separate repository for each presentation project to maintain modularity and a clean history.

## Bundled Resources

- **`templates/`**: Contains a `slide_template.html` for consistent slide design.
- **`scripts/`**: Includes a `git_push.sh` script to automate the version control process.
