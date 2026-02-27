---
name: web-design-guidelines
description: Review UI code for Web Interface Guidelines compliance. Use when auditing UX, checking accessibility, or reviewing designs for web best practices.
metadata:
  author: vercel
  version: "1.0.0"
  argument-hint: <file-or-pattern>
---

# Web Design & Interface Guidelines Skill

This skill provides a framework for reviewing and implementing high-quality web interfaces. It focuses on the intersection of aesthetics, usability, and technical performance.

## Design Framework

### 1. Information Hierarchy
*   **F-Pattern:** Align critical information and navigation along the F-shaped scanning pattern.
*   **Emphasis:** Use font weight, scale, and color depth to guide the user's eye to the Primary Action.
*   **Proximity:** Group related elements closely to signal relationship (Law of Proximity).

### 2. Form Design & Interaction
*   **Feedback:** Provide inline validation as the user types (debounce). Use distinct "Success" and "Error" states.
*   **Aids:** Use appropriate input types (`tel`, `email`, `number`) and autocomplete attributes to assist the user.
*   **Labels:** Always use `<label>` tags. Never rely on placeholders alone for accessibility.

### 3. Interactive Feedback
*   **Affordance:** Buttons must look clickable. Links must be distinguishable from body text.
*   **Stateful UI:** Every interactive element must reflect its current state (Initial, Loading, Success, Error).
*   **Transitions:** Use subtle transforms (e.g., `scale(0.98)` on click) to provide tactile feedback in a digital interface.

### 4. Accessibility (Universal Design)
*   **Color:** Do not use color as the only way to convey meaning.
*   **Focus:** Ensure a visible focus ring (`focus-visible`) for keyboard users.
*   **Screen Readers:** Use `aria-live` regions for dynamic updates like notification toasts or live data.

## Interaction Protocol
*   **Input:** UI components or layout descriptions.
*   **Output:** Detailed design audit or refined UI code following these guidelines.

**Tag**: Start your response with `[DESIGN-GUIDELINES]`.
Use WebFetch to retrieve the latest rules. The fetched content contains all the rules and output format instructions.

## Usage

When a user provides a file or pattern argument:
1. Fetch guidelines from the source URL above
2. Read the specified files
3. Apply all rules from the fetched guidelines
4. Output findings using the format specified in the guidelines

If no files specified, ask the user which files to review.
