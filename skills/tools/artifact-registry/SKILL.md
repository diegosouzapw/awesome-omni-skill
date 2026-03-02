---
name: artifact-registry
description: 'Helps scaffold and register new artifacts in the strict-mode context engineering kit.'
metadata:
  id: ce.skill.artifact-registry
  tags: [routing, validation, tools]
  inputs:
    files: [ce.manifest.jsonc, .github/ce/vocab.md]
    concepts: [strict-mode]
    tools: [toolset:write]
  outputs:
    artifacts: []
    files: []
    actions: [create-artifact, register-manifest, run-task]
  dependsOn:
    artifacts: [ce.task.validate]
    files: [.vscode/tasks.json]
  related:
    artifacts: [ce.prompt.add-artifact]
    files: [.github/skills/artifact-registry/scripts/register-artifact.py]
---

# Artifact Registry Skill

This skill assists the agent in safely adding new artifacts (prompts, instructions, agents, skills,
toolsets or docs) to the strict-mode manifest. Use it whenever the user asks to create a new file
type or extend functionality.

## Steps

1. **Gather details**. Ask the user for:
   - The type of artifact they wish to create (doc, agent, prompt, instruction, skill, toolset or task).
   - A unique identifier slug (lower‑case words separated by hyphens).
   - A short description explaining what it does and when to use it.
   - Any tags (from the controlled vocabulary defined in `.github/ce/vocab.md`).
   - The path where the file should reside, relative to the repository root (e.g.
     `.github/prompts/my-feature.prompt.md`).

   If the artifact is a skill, also ask for the skill `name`.

2. **Scaffold the file**. Use the Python script `scripts/scaffold-artifact.py` to create the file
   with appropriate front matter. Pass the captured details as command‑line arguments. The
   script will create any missing directories, add required subfolders for skills and insert
   placeholders for the body.

3. **Register in the manifest**. Immediately run `scripts/register-artifact.py` on the newly
   created file. This script parses the YAML front matter, constructs a manifest entry with
   declared inputs/outputs and inserts it into `ce.manifest.jsonc`. It will fail if the `id` is
   not unique or if tags are invalid.

4. **Validate**. Invoke the `Context Kit: Validate` task to ensure the manifest and new artifact
   meet all structural and dependency requirements. Address any reported issues before proceeding.

5. **Summarise**. Inform the user that the artifact was created and registered successfully, and
   remind them to fill in the body of the file with appropriate instructions or documentation.

By following these steps, new artifacts enter the system cleanly without bypassing strict‑mode
constraints or introducing technical debt.
