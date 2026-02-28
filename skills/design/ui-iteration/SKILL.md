---
name: ui-iteration
description: Iterate on UI visuals by editing code, hot reloading, capturing screenshots, and refining in loops.
metadata:
  short-description: Visual UI iteration loop with screenshots and mock states.
---

Use this skill when the user wants iterative UI refinements with live preview and screenshots.

Core loop (repeat until objectives are met):

1) Inspect the current UI state and related code.
2) Make a small, focused UI change.
3) Hot reload.
4) Capture a screenshot with a timestamped filename.
5) Review the screenshot, explain what changed, and decide the next tweak.

Requirements:

- Follow the user-provided commands for hot reload and screenshots when given.
- If the user asks to use specific tools/commands, use them exactly as written.
- Save screenshots with timestamped names (e.g., `TEST-YYYYMMDD-HHMMSS.png`) so progress is traceable.
- Keep each iteration small and intentional; avoid large refactors during visual tuning.
- If you temporarily mock states (enabled/disabled, demo text, forced visibility), revert the mocks before the final response.
- Call out any errors from hot reload or rendering and fix them before continuing.
- If a screenshot command requires elevated permissions, request the needed approval or ask the user to run it.

Mocking guidance:

- It is acceptable to add temporary flags or demo strings to preview UI states.
- Keep mock logic clearly isolated and easy to remove.
- Always remove mock flags/values in the final deliverable.

Screenshot notes:

- Use the device ID and output path specified by the user.
- Include the full output path in your response each iteration.
- If the wrong app is in front, ask the user to bring the target UI forward, then re-capture.

Tooling and command usage (examples):

- Timestamp for filenames (shell):
  - `date +%Y%m%d-%H%M%S`
- Flutter hot reload:
  - `scripts/flutter-runctl reload -d <DEVICE_ID> -t r`
  - Flutter hot reload command must be run outside of sandbox environment.
- Screenshot capture:
  - `scripts/device-screenshot take_screenshot <DEVICE_TARGET> /full/path/TEST-<TIMESTAMP>.png`
  - Screenshot capture command must be run outside of sandbox environment.

If the user supplies different tooling or a different platform, replace the above with their commands.

Finish criteria:

- The UI meets the user's aesthetic and usability goals.
- Mock code and preview-only data are removed.
- Provide the final screenshot path and a concise summary of changes.
