---
name: create-custom-prompt
description: Prompt for creating custom prompt files
---

# Skill: Custom Prompt File Creation Assistant

<role_gate>
<required_agent>Architect</required_agent>
<instruction>
Before proceeding with any instructions, you MUST strictly check that your `ACTIVE_AGENT_ID` matches the `required_agent` above.

Match Case:

- Proceed normally.

Mismatch Case:

- You MUST read the file `.github/agents/{required_agent}.agent.md`.
- You MUST ADOPT the persona defined in that file for the duration of this skill.
- Proceed with the skill acting as the {required_agent}.

</instruction>
</role_gate>

You are an expert in creating VS Code custom prompt files (`.prompt.md`).
You will interview the user to understand their requirements and propose an effective prompt file.

## 🔒 Terraformer Convention (Mandatory)

In this repository, prompt files in `.github/prompts/` MUST be **thin wrappers**.

- Prompt files should only:
  - declare metadata (typically `description` + `agent`)
  - state the supporting role (for role-specific prompts)
  - delegate the actual procedure to a **Skill** under `.github/skills/…`
- The detailed, repeatable procedure MUST live in `SKILL.md`.

If the user requests a new prompt that would otherwise require detailed instructions, you MUST:

1. ensure a corresponding Skill exists (create it if needed), then
2. generate a thin wrapper prompt that calls that Skill.

## 📋 Task Initialization

**IMMEDIATELY** use the `#todo` tool to register the following tasks to track your progress:

1.  **Fetch Documentation**: Retrieve official docs from `code.visualstudio.com`.
2.  **Requirement Hearing**: Interview user and infer settings (Agent, Tools, etc.).
3.  **Create Prompt File**: Generate the YAML frontmatter and body.
4.  **Proposal and Review**: Present the file and get approval.
5.  **Final Check**: Review the "Final Check" section.

## Step 1: Fetch Documentation (Mandatory)

**You must perform the following action first:**

1. Use the `fetch` tool to retrieve the official documentation for custom prompt files from the following URL:
   - URL: `https://code.visualstudio.com/docs/copilot/customization/prompt-files`

This document contains the format, variables, usage, and best practices for prompt files.
Read the documentation before creating the prompt file based on the user's requirements.

## Step 2: Requirement Hearing

**To minimize user burden, proceed with the following approach:**

1. **First Question (Mandatory)**: "What do you want to achieve with this prompt?"
   - Understand the goal from the user's answer.

2. **Auto-Inference**: Automatically determine the following items from the goal and present them as a proposal:
   - **Agent**:
     - Question answering, explanation, information provision → `ask`
     - Code generation, editing, complex tasks, file operations → `agent` (default)
     - **Important**: For Terraformed projects, it is recommended to use role-specific agents (e.g., `@Developer`, `@Architect`) instead of the generic `agent` whenever possible.
   - **Tools**:
     - Basically unspecified (all tools enabled).
     - Explicitly specify only if restrictions are necessary for security reasons, etc.
   - **Output Format**: Assume a format naturally derived from the goal.
   - **Skill Delegation (Mandatory)**:
     - Propose a corresponding Skill name and directory under `.github/skills/…`.
     - Default to creating/updating that Skill to contain the full procedure.
     - The new prompt file must be a thin wrapper that calls the Skill.
   - **Scope**:
     - Default: **Workspace prompt** in `.github/prompts/` (do not offer User Profile unless explicitly requested)

3. **Questions only if confirmation is needed**:
   - Ask questions with specific options only if there are important details that cannot be inferred.
   - Example: "Do you need input variables such as component names? (Recommended: `${input:componentName:Component Name}`)"

4. **Use of Default Values**:
   - Use default values based on best practices for items not explicitly specified.
   - Do not ask the user, but include an explanation in the proposal.

## Step 3: Create Prompt File

**Create a prompt file that follows the thin-wrapper convention:**

1. **YAML Frontmatter**:
   - `description`: Generate a concise description from the user's goal.
   - `agent`: Set the inferred agent (explain the reason).
   - `tools`: Basically omitted (all tools enabled). Explicitly specify only if restrictions are necessary.
   - `model`: Omit unless there are special requirements (use default model).

   Repo convention:
   - Prefer to omit `name` and `argument-hint` unless there is a strong usability reason.

2. **Prompt Body**:

   The body MUST be minimal and delegate to a Skill.
   Use the same pattern as existing prompts in `.github/prompts/`.

   Required structure:
   - One line that establishes the supporting role (when applicable):
     - `You are Supporting the **@Architect**.` (or other appropriate role)
   - One line that delegates to the Skill:
     - `Use the \`<skill-directory-or-name>\` skill to <do the task>.`

   Do NOT embed detailed procedures, long checklists, or multi-step instructions in the prompt body.

3. **Explanation of Recommendations**:
   - Why that agent was chosen.
   - If tools were restricted, the reason why.
   - In what situations it is effective.

**Apply Official Documentation Best Practices (and the repo convention):**

- Clearly describe what you want to achieve and the expected output format.
- Provide examples of expected input and output.
- Avoid duplication and utilize Markdown links to custom instructions.
- Accurately use the variable syntax described in the official documentation.
- Write specific and actionable instructions.
- Strictly observe the YAML frontmatter format.

Additionally (repo-specific):

- Ensure the prompt delegates to a Skill under `.github/skills/…`.
- If the corresponding Skill does not exist yet, create it (or propose creating it) and put the procedure there.

## Step 4: Proposal and Review

**Complete with minimal confirmation:**

1. **Present the completed prompt file**:
   - State the auto-inferred settings and their reasons.
   - Display the complete file content in a code block.
   - Confirm with: "I will create it with this prompt. Please let me know if there are any parts that need correction."

2. **Fine-tune if necessary**:
   - Reflect immediately if there is specific feedback from the user.

3. **File Creation**:
   - Once approved, create the prompt file in `.github/prompts/`.
   - Ensure the delegated Skill exists in `.github/skills/<skill-directory>/SKILL.md`.
   - After creation, briefly guide how to use it (type `/prompt-name` in chat).

---

**Important:** When executing this prompt, be sure to fetch the official documentation first and create the prompt file based on the latest specifications.

## ✅ Final Check

**Before finishing, confirm:**

- [ ] All todo are marked as completed.
- [ ] Official documentation was fetched and used.
- [ ] The prompt file includes valid YAML frontmatter.
- [ ] The prompt file content is displayed in a code block.
- [ ] The file is created in the correct directory.
- [ ] The prompt body is a thin wrapper that delegates to a Skill.
- [ ] The delegated Skill exists and contains the detailed procedure.
