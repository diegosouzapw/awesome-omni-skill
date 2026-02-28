---
name: ae-sdd-init
description: Initialize a new SDD change set after user-approved naming
---

# SDD Init

Create a new change set with the SDD CLI after collaboratively deriving a strong name and getting explicit user approval.

## Required Skills

- `spec-driven-development` (load first; use CLI-driven initialization workflow)

## Inputs

- **Intent Context**: What the user wants to propose or change.
- **Proposed Change Set Name**: Derived from user intent and naming conventions.

## Instructions

1. **Load SDD Context**: Load `spec-driven-development` first.

2. **Derive Candidate Name**:
   - Ask the user what they want to propose (or use the current request context if already provided).
   - Propose a concise kebab-case name that reflects the intended change.
   - Keep names action-oriented and specific (for example: `add-passwordless-login`, `improve-checkout-retries`).

3. **Require Explicit Approval**:
   - Ask the user to approve the proposed change set name.
   - If the user declines, iterate with a revised name.
   - Do not initialize anything until the user explicitly approves a final name.

4. **Initialize via CLI**:
   - After approval, run:

   ```bash
   ae sdd init <approved-name>
   ```

   - Use the SDD CLI only. Do not hand-roll `changes/<name>/` scaffolding.

5. **Confirm Result**:
   - Report the approved name and that initialization completed.
   - Suggest the next command based on status output (typically `ae sdd status <approved-name>`).

## Success Criteria

- `spec-driven-development` skill is loaded before init flow.
- A change set name is collaboratively derived from user intent.
- The user explicitly approves the final name.
- Initialization is performed using `ae sdd init <approved-name>`.
- The user receives a clear next command.

## Usage Example

```text
User intent: "I want to propose retry behavior for failed webhooks"
Agent: "Proposed change set name: improve-webhook-retry-behavior. Does that name work for you?"
User: "Yes"
Agent runs: ae sdd init improve-webhook-retry-behavior
Agent: "Initialized improve-webhook-retry-behavior. Next: ae sdd status improve-webhook-retry-behavior"
```
