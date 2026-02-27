---
name: agentv-prompt-optimizer
description: Iteratively optimize prompt files against AgentV evaluation datasets by analyzing failures and refining instructions.
---

# AgentV Prompt Optimizer

## Input Variables
- `eval-path`: Path or glob pattern to the AgentV evaluation file(s) to optimize against
- `optimization-log-path` (optional): Path where optimization progress should be logged

## Workflow

1.  **Initialize**
    - Verify `<eval-path>` (file or glob) targets the correct system.
    - **Identify Prompt Files**:
        - Infer prompt files from the eval file content (look for `file:` references in `input` that match these patterns).
        - Recursively check referenced prompt files for *other* prompt references (dependencies).
        - If multiple prompts are found, consider ALL of them as candidates for optimization.
    - **Identify Optimization Log**:
        - If `<optimization-log-path>` is provided, use it.
        - If not, create a new one in the parent directory of the eval files: `optimization-[timestamp].md`.
    - Read content of the identified prompt file.

2.  **Optimization Loop** (Max 10 iterations)
    - **Execute (The Generator)**: Run `agentv eval <eval-path>`.
        - *Targeted Run*: If iterating on specific stubborn failures, use `--test-id <test_id>` to run only the relevant tests.
    - **Analyze (The Reflector)**:
        - Locate the results file path from the console output (e.g., `.agentv/results/eval_...jsonl`).
        - **Orchestrate Subagent**: Use `runSubagent` to analyze the results.
            - **Task**: Read the results file, calculate pass rate, and perform root cause analysis.
            - **Output**: Return a structured analysis including:
                - **Score**: Current pass rate.
                - **Root Cause**: Why failures occurred (e.g., "Ambiguous definition", "Hallucination").
                - **Insight**: Key learning or pattern identified from the failures.
                - **Strategy**: High-level plan to fix the prompt (e.g., "Clarify section X", "Add negative constraint").
    - **Decide**:
        - If **100% pass**: STOP and report success.
        - If **Score decreased**: Revert last change, try different approach.
        - If **No improvement** (2x): STOP and report stagnation.
    - **Refine (The Curator)**:
        - **Orchestrate Subagent**: Use `runSubagent` to apply the fix.
            - **Task**: Read the relevant prompt file(s), apply the **Strategy** from the Reflector, and generate the log entry.
            - **Output**: The **Log Entry** describing the specific operation performed.
                  ```markdown
                  ### Iteration [N]
                  - **Operation**: [ADD / UPDATE / DELETE]
                  - **Target**: [Section Name]
                  - **Change**: [Specific text added/modified]
                  - **Trigger**: [Specific failing test case or error pattern]
                  - **Rationale**: [From Reflector: Root Cause]
                  - **Score**: [From Reflector: Current Pass Rate]
                  - **Insight**: [From Reflector: Key Learning]
                  ```
        - **Strategy**: Treat the prompt as a structured set of rules. Execute atomic operations:
            - **ADD**: Insert a new rule if a constraint was missed.
            - **UPDATE**: Refine an existing rule to be clearer or more general.
                - *Clarify*: Make ambiguous instructions specific.
                - *Generalize*: Refactor specific fixes into high-level principles (First Principles).
            - **DELETE**: Remove obsolete, redundant, or harmful rules.
                - *Prune*: If a general rule covers specific cases, delete the specific ones.
            - **Negative Constraint**: If hallucinating, explicitly state what NOT to do. Prefer generalized prohibitions over specific forbidden tokens where possible.
            - **Safety Check**: Ensure new rules don't contradict existing ones (unless intended).
        - **Constraint**: Avoid rewriting large sections. Make surgical, additive changes to preserve existing behavior.
    - **Log Result**:
        - Append the **Log Entry** returned by the Curator to the optimization log file.

3.  **Completion**
    - Report final score.
    - Summarize key changes made to the prompt.
    - **Finalize Optimization Log**: Add a summary header to the optimization log file indicating the session completion and final score.

## Guidelines
- **Generalization First**: Prefer broad, principle-based guidelines over specific examples or "hotfixes". Only use specific rules if generalized instructions fail to achieve the desired score.
- **Simplicity ("Less is More")**: Avoid overfitting to the test set. If a specific rule doesn't significantly improve the score compared to a general one, choose the general one.
- **Structure**: Maintain existing Markdown headers/sections.
- **Progressive Disclosure**: If the prompt grows too large (>200 lines), consider moving specialized logic into a separate file or skill.
- **Quality Criteria**: Ensure the prompt defines a clear persona, specific task, and measurable success criteria.
