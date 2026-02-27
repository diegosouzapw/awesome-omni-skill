---
name: browser
description: Control a Chrome session via Stagehand to browse, act, extract, and screenshot on demand inside the Factory CLI.
---

# Skill: Browser

Use this skill when you need live browser automation during a Factory session—opening sites, clicking through flows, gathering structured data, or capturing screenshots.

## Inputs
- Target URL or task description (natural language)
- Optional structured extraction schema (JSON field → type)

## Behavior
1. Ensure Chrome is running via Stagehand with the local profile stored in `.chrome-profile`.
2. Support these commands:
   - `navigate <url>`: open a page and capture a screenshot.
   - `act "<instruction>"`: perform natural-language actions.
   - `extract "<instruction>" '{"field":"type"}'`: return structured data.
   - `observe "<goal>"`: list suggested steps the agent can take.
   - `screenshot`: capture the current viewport.
   - `close`: shut down the session when finished.
3. Save screenshots to `agent/browser_screenshots` and report the file path in the response.
4. When tasks finish, summarize what happened plus any follow-up steps for the user.

## Verification
- If a navigation/action fails, include the error message and prompt the user for next steps.
- Before ending the session, ensure `close` has been run so Chrome processes don’t linger.

## Notes
- This skill expects `ANTHROPIC_API_KEY` (for Stagehand) and, if used via `droid exec`, `FACTORY_API_KEY` to already be configured.
- The working directory should remain inside the cloned skill folder so relative paths resolve correctly.
