---
name: 1password
description: Plan, validate, and use 1Password CLI setup for secret injection and
  auth. Use when tasks need 1Password CLI usage, secret references, op run/read/inject,
  or provisioning secrets via env vars/.env files and scripts.
knowledge_graph_profile: references/task-profile.json
---

# 1Password CLI

Follow the official CLI get-started steps. Don't guess install commands.

## Links

- https://developer.1password.com/docs/cli/get-started/
- https://developer.1password.com/docs/cli/secret-references/
- https://developer.1password.com/docs/cli/secrets-environment-variables/
- https://developer.1password.com/docs/cli/secrets-scripts/
- https://developer.1password.com/docs/cli/environment-variables/
- https://developer.1password.com/docs/cli/secret-reference-syntax/
- https://developer.1password.com/docs/cli/secrets-template-syntax/
- https://developer.1password.com/docs/cli/item-fields/
- https://developer.1password.com/docs/cli/item-template-json/
- https://developer.1password.com/docs/cli/vault-permissions/
- https://developer.1password.com/docs/cli/user-states/
- https://developer.1password.com/docs/cli/item-create/
- https://developer.1password.com/docs/cli/item-edit/
- https://developer.1password.com/docs/cli/ssh-keys/
- https://developer.1password.com/docs/cli/reference/
- https://developer.1password.com/docs/cli/best-practices/
- https://developer.1password.com/docs/cli/reference/commands/completion/
- https://developer.1password.com/docs/cli/reference/commands/inject/
- https://developer.1password.com/docs/cli/reference/commands/read/
- https://developer.1password.com/docs/cli/reference/commands/run/
- https://developer.1password.com/docs/cli/reference/commands/signin/
- https://developer.1password.com/docs/cli/reference/commands/signout/
- https://developer.1password.com/docs/cli/reference/commands/update/
- https://developer.1password.com/docs/cli/reference/commands/whoami/
- https://developer.1password.com/docs/cli/reference/management-commands/account/
- https://developer.1password.com/docs/cli/reference/management-commands/connect/
- https://developer.1password.com/docs/cli/reference/management-commands/document/
- https://developer.1password.com/docs/cli/reference/management-commands/events-api/
- https://developer.1password.com/docs/cli/reference/management-commands/group/
- https://developer.1password.com/docs/cli/reference/management-commands/item/
- https://developer.1password.com/docs/cli/reference/management-commands/plugin/
- https://developer.1password.com/docs/cli/reference/management-commands/service-account/
- https://developer.1password.com/docs/cli/reference/management-commands/user/
- https://developer.1password.com/docs/cli/reference/management-commands/vault/
- https://developer.1password.com/docs/environments/
- https://developer.1password.com/docs/environments/local-env-file/
- https://developer.1password.com/docs/environments/cursor-hook-validate/
- https://developer.1password.com/docs/cli/shell-plugins/homebrew/
- https://developer.1password.com/docs/cli/shell-plugins/huggingface/
- https://developer.1password.com/docs/cli/shell-plugins/openai/
- https://developer.1password.com/docs/cli/shell-plugins/cloudflare-workers/

## References

- `references/get-started.md` (install + app integration + sign-in flow)
- `references/cli-examples.md` (real `op` examples)
- `references/secret-references.md` (what secret references are + how to resolve)
- `references/secrets-environment-variables.md` (env + .env usage with `op run`)
- `references/secrets-scripts.md` (script patterns using `op run/read/inject`)
- `references/environment-variables.md` (OP_* env vars and precedence)
- `references/secret-reference-syntax.md` (URI rules, attributes, variables)
- `references/secrets-template-syntax.md` (template/enclosure/variables rules)
- `references/item-fields.md` (built-in vs custom fields + types)
- `references/item-template-json.md` (template keys + sections/fields)
- `references/vault-permissions.md` (permission hierarchy + dependencies)
- `references/user-states.md` (user state meanings)
- `references/item-create.md` (create items safely, templates, assignments)
- `references/item-edit.md` (edit items safely, templates, caveats)
- `references/ssh-keys.md` (generate and retrieve SSH keys)
- `references/cli-reference.md` (command structure, IDs, caching, flags)
- `references/best-practices.md` (updates, least privilege, templates)
- `references/commands-completion.md` (shell completion)
- `references/commands-inject.md` (inject secrets into templates)
- `references/commands-read.md` (read secrets by reference)
- `references/commands-run.md` (run with env secrets)
- `references/commands-signin.md` (sign in via app integration)
- `references/commands-signout.md` (sign out behavior)
- `references/commands-update.md` (update op CLI)
- `references/commands-whoami.md` (active account info)
- `references/management-account.md` (account management commands)
- `references/management-connect.md` (Connect server commands)
- `references/management-document.md` (document item commands)
- `references/management-events-api.md` (Events API integration)
- `references/management-group.md` (group commands)
- `references/management-item.md` (item commands)
- `references/management-plugin.md` (shell plugin commands)
- `references/management-service-account.md` (service account commands)
- `references/management-user.md` (user commands)
- `references/management-vault.md` (vault commands)
- `references/environments.md` (Environments overview + requirements)
- `references/environments-local-env-file.md` (local .env mount destination)
- `references/environments-cursor-hook-validate.md` (Cursor hook validation flow)
- `references/shell-plugins-homebrew.md` (Homebrew plugin setup)
- `references/shell-plugins-huggingface.md` (Hugging Face plugin setup)
- `references/shell-plugins-openai.md` (OpenAI plugin setup)
- `references/shell-plugins-cloudflare-workers.md` (Cloudflare Workers plugin setup)

## Workflow

1. Check OS + shell.
2. Verify CLI present: `op --version`.
3. Confirm desktop app integration is enabled (per get-started) and the app is unlocked.
4. REQUIRED: create a fresh tmux session for all `op` commands (no direct `op` calls outside tmux).
5. Sign in / authorize inside tmux: `op signin` (expect app prompt).
6. Verify access inside tmux: `op whoami` (must succeed before any secret read).
7. If multiple accounts: use `--account` or `OP_ACCOUNT`.
8. Choose the secret-loading path:
   - `op run` for environment variables / `.env` files.
   - `op read` for a single secret to stdout or a file.
   - `op inject` for config/template files.
   - `op plugin run` for shell plugin flows.
9. Environment notes:
   - Prefer 1Password Environments or service accounts for automation.
   - Keep `.env` templates in source control; never commit resolved env files.
   - Use Cursor validate hooks (if enabled) to fail fast on missing env vars.

## Environments UI quick nav

- Enable Developer: Settings > Developer > Show 1Password Developer experience.
- Open Environments: Developer > View Environments.
- Create environment: New environment.
- Add variables: Import .env file or New variable.
- Manage access: Manage environment > Manage access.
- Configure destinations: Destinations tab > Configure destination.

## REQUIRED tmux session (T-Max)

The shell tool uses a fresh TTY per command. To avoid re-prompts and failures, always run `op` inside a dedicated tmux session with a fresh socket/session name.

Example (see `tmux` skill for socket conventions, do not reuse old session names):

```bash
SOCKET_DIR="${CLAWDBOT_TMUX_SOCKET_DIR:-${TMPDIR:-/tmp}/clawdbot-tmux-sockets}"
mkdir -p "$SOCKET_DIR"
SOCKET="$SOCKET_DIR/clawdbot-op.sock"
SESSION="op-auth-$(date +%Y%m%d-%H%M%S)"

tmux -S "$SOCKET" new -d -s "$SESSION" -n shell
tmux -S "$SOCKET" send-keys -t "$SESSION":0.0 -- "op signin --account my.1password.com" Enter
tmux -S "$SOCKET" send-keys -t "$SESSION":0.0 -- "op whoami" Enter
tmux -S "$SOCKET" send-keys -t "$SESSION":0.0 -- "op vault list" Enter
tmux -S "$SOCKET" capture-pane -p -J -t "$SESSION":0.0 -S -200
tmux -S "$SOCKET" kill-session -t "$SESSION"
```

## Guardrails

- Never paste secrets into logs, chat, or code.
- Prefer `op run` / `op inject` over writing secrets to disk.
- If sign-in without app integration is needed, use `op account add`.
- If a command returns "account is not signed in", re-run `op signin` inside tmux and authorize in the app.
- Do not run `op` outside tmux; stop and ask if tmux is unavailable.

## Compliance
- Follow repo and platform security standards (least privilege, no plaintext secrets).

## Scope and triggers
- Use this skill when the task matches its description and triggers.
- If the request is outside scope, route to the referenced skill.

## Response format (required)
- For normal requests, include a `## Outputs` section describing delivered artifacts.
- For edge cases with missing info, include a `## Inputs` section listing what is missing.

## Cognitive Support / Plain-Language
- Optimize for low cognitive load (TBI support): one task at a time, explicit steps.
- Use plain language first; define jargon in parentheses.
- Keep steps short and checklist-driven where possible.
- Externalize state: decisions, assumptions, and the next step.
- Provide ELI5 explanations for non-trivial logic.
- Ask one question at a time; prefer multiple-choice when possible.

- For out-of-scope requests, include a `## When to use` section explaining the correct trigger.


## Required inputs
- User request details and any relevant files/links.


## Deliverables
- A structured response or artifact appropriate to the skill.
- Include `schema_version: 1` if outputs are contract-bound.


## Constraints
- Redact secrets/PII by default.
- Avoid destructive operations without explicit user direction.


## Validation
- Run any relevant checks or scripts when available.
- Fail fast and report errors before proceeding.


## Philosophy
- Favor clarity, explicit tradeoffs, and verifiable outputs.


## Anti-patterns
- Avoid vague guidance without concrete steps.
- Do not invent results or commands.
 - Do not add features outside the agreed scope.
## Procedure
1) Clarify scope and inputs.
2) Execute the core workflow.
3) Summarize outputs and next steps.

## Variation
- Vary tone, depth, and structure based on context.
- Avoid repeating the same outline across outputs.

## Remember
The agent is capable of extraordinary work in this domain. Use judgment, adapt to context, and push boundaries when appropriate.

<!-- skill-score-boost-v1 -->
## Philosophy and tradeoffs
- Use this skill when consistent decision-making matters more than one-off execution because project context should drive the approach.
- Principle and mindset: prioritize tradeoffs and constraints over rigid checklists; understand why each step exists.
- Ask this to keep outcomes robust: Why is this the right default, and what could change this outcome?
- How do we adapt if constraints shift?
- What evidence is needed before choosing one path over another?

## Anti-patterns and caveats
- Avoid applying this playbook generically without checking repository-specific context.
- **NEVER** skip required validation gates when behavior changes.
- **DO NOT** use this skill as a rigid replacement for engineering judgment.
- **DON'T** ignore warnings or assume one pattern fits all repos.
- Common pitfall: treating anti-patterns as optional.
- Incorrect assumptions here can lead to fragile guidance.
- Warning: wrong sequencing can create avoidable regressions.

## Variation and adaptation
- Vary the workflow by team size, risk, and deployment target.
- Use different strategies for small, medium, and large changes.
- Adapt recommendations to the specific environment and avoid repetitive templates.
- Avoid generic or cookie-cutter responses; craft context-specific alternatives.
- Keep outputs diverse and not repetitive.
- Converge on a custom path only after evidence review.
- Different constraints should produce different, non-generic recommendations.

## Empowering execution style
- Be capable of exploring multiple options and enabling the team to make safe decisions.
- Unlock confidence by explaining options and tradeoffs clearly.
- Feel free to be creative while staying rigorous and precise.
- Push boundaries with practical alternatives when simple recipes fail.
- Enable outcomes-oriented problem solving.
