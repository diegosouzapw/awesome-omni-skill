---
name: agentv-eval-builder
description: Create and maintain AgentV YAML evaluation files for testing AI agent performance. Use this skill when creating new eval files, adding eval cases, or configuring custom evaluators (code validators or LLM judges) for agent testing workflows.
---

# AgentV Eval Builder

## Schema Reference
- Schema: `references/eval-schema.json` (JSON Schema for validation and tooling)
- Format: YAML or JSONL (see below)
- Examples: `references/example-evals.md`

## Feature Reference
- Rubrics: `references/rubric-evaluator.md` - Structured criteria-based evaluation
- Composite Evaluators: `references/composite-evaluator.md` - Combine multiple evaluators
- Tool Trajectory: `references/tool-trajectory-evaluator.md` - Validate agent tool usage
- Structured Data + Metrics: `references/structured-data-evaluators.md` - `field_accuracy`, `latency`, `cost`
- Custom Evaluators: `references/custom-evaluators.md` - Code and LLM judge templates
- Batch CLI: `references/batch-cli-evaluator.md` - Evaluate batch runner output (JSONL)
- Compare: `references/compare-command.md` - Compare evaluation results between runs

## Structure Requirements
- Root level: `description` (optional), `execution` (with `target`), `evalcases` (required)
- Eval case fields: `id` (required), `expected_outcome` (required), `input_messages` or `input` (required)
- Optional fields: `expected_messages` (or `expected_output`), `conversation_id`, `rubrics`, `execution`
- `expected_messages` is optional - omit for outcome-only evaluation where the LLM judge evaluates based on `expected_outcome` criteria alone
- Message fields: `role` (required), `content` (required)
- Message roles: `system`, `user`, `assistant`, `tool`

## Input/Output Shorthand (Aliases)

For simpler eval cases, use shorthand aliases instead of the verbose `input_messages` and `expected_messages`:

| Alias | Canonical | Description |
|-------|-----------|-------------|
| `input` | `input_messages` | String expands to single user message |
| `expected_output` | `expected_messages` | String/object expands to single assistant message |

**String shorthand:**
```yaml
evalcases:
  - id: simple-test
    expected_outcome: Correct answer
    input: "What is 2+2?"                    # Expands to [{role: user, content: "..."}]
    expected_output: "The answer is 4"       # Expands to [{role: assistant, content: "..."}]
```

**Object shorthand** (for structured output validation):
```yaml
evalcases:
  - id: structured-output
    expected_outcome: Risk assessment
    input: "Analyze this transaction"
    expected_output:                          # Expands to assistant message with object content
      riskLevel: High
      confidence: 0.95
```

**Array syntax** still works for multi-message conversations:
```yaml
input:
  - role: system
    content: "You are a calculator"
  - role: user
    content: "What is 2+2?"
```

**Precedence:** Canonical names (`input_messages`, `expected_messages`) take precedence when both are specified.
- Content types: `text` (inline), `file` (relative or absolute path)
- Attachments (type: `file`) should default to the `user` role
- File paths: Relative (from eval file dir) or absolute with "/" prefix (from repo root)

## JSONL Format

For large-scale evaluations, use JSONL (one eval case per line) instead of YAML:

**dataset.jsonl:**
```jsonl
{"id": "test-1", "expected_outcome": "Correct answer", "input_messages": [{"role": "user", "content": "What is 2+2?"}]}
{"id": "test-2", "expected_outcome": "Clear explanation", "input_messages": [{"role": "user", "content": [{"type": "text", "value": "Review this"}, {"type": "file", "value": "./code.py"}]}]}
```

**dataset.yaml (optional sidecar for defaults):**
```yaml
description: My dataset
dataset: my-tests
execution:
  target: azure_base
evaluator: llm_judge
```

Benefits: Git-friendly diffs, streaming-compatible, easy programmatic generation.
Per-case fields override sidecar defaults. See `examples/features/basic-jsonl/` for complete example.

## Custom Evaluators

Configure multiple evaluators per eval case via `execution.evaluators` array.

### Code Evaluators
Scripts that validate output programmatically:

```yaml
execution:
  evaluators:
    - name: json_format_validator
      type: code_judge
      script: uv run validate_output.py
      cwd: ../../evaluators/scripts
```

**Contract:**
- Input (stdin): JSON with `question`, `expected_outcome`, `reference_answer`, `candidate_answer`, `guideline_files`, `input_files`, `input_messages`, `expected_messages`, `output_messages`, `trace_summary`
- Output (stdout): JSON with `score` (0.0-1.0), `hits`, `misses`, `reasoning`

**Target Proxy:** Code evaluators can access an LLM through the target proxy for sophisticated evaluation logic (e.g., Contextual Precision, semantic similarity). Enable with `target: {}`:

```yaml
execution:
  evaluators:
    - name: contextual_precision
      type: code_judge
      script: bun run evaluate.ts
      target: {}           # Enable target proxy (max_calls: 50 default)
```

**RAG Evaluation Pattern:** For retrieval-based evals, pass retrieval context via `expected_messages.tool_calls`:

```yaml
expected_messages:
  - role: assistant
    tool_calls:
      - tool: vector_search
        output:
          results: ["doc1", "doc2", "doc3"]
```

**TypeScript evaluators:** Keep `.ts` source files and run them via Node-compatible loaders such as `npx --yes tsx` so global `agentv` installs stay portable. See `references/custom-evaluators.md` for complete templates, target proxy usage, and command examples.

**Template:** See `references/custom-evaluators.md` for Python and TypeScript templates

### LLM Judges
Language models evaluate response quality:

```yaml
execution:
  evaluators:
    - name: content_evaluator
      type: llm_judge
      prompt: /evaluators/prompts/correctness.md
      model: gpt-5-chat
```

### Tool Trajectory Evaluators
Validate agent tool usage patterns (requires `output_messages` with `tool_calls` from provider):

```yaml
execution:
  evaluators:
    - name: research_check
      type: tool_trajectory
      mode: any_order       # Options: any_order, in_order, exact
      minimums:             # For any_order mode
        knowledgeSearch: 2
      expected:             # For in_order/exact modes
        - tool: knowledgeSearch
        - tool: documentRetrieve
```

See `references/tool-trajectory-evaluator.md` for modes and configuration.

### Multiple Evaluators
Define multiple evaluators to run sequentially. The final score is a weighted average of all results.

```yaml
execution:
  evaluators:
    - name: format_check      # Runs first
      type: code_judge
      script: uv run validate_json.py
    - name: content_check     # Runs second
      type: llm_judge
```

### Rubric Evaluator
Inline rubrics for structured criteria-based evaluation:

```yaml
evalcases:
  - id: explanation-task
    expected_outcome: Clear explanation of quicksort
    input_messages:
      - role: user
        content: Explain quicksort
    rubrics:
      - Mentions divide-and-conquer approach
      - Explains the partition step
      - id: complexity
        description: States time complexity correctly
        weight: 2.0
        required: true
```

See `references/rubric-evaluator.md` for detailed rubric configuration.

### Composite Evaluator
Combine multiple evaluators with aggregation:

```yaml
execution:
  evaluators:
    - name: release_gate
      type: composite
      evaluators:
        - name: safety
          type: llm_judge
          prompt: ./prompts/safety.md
        - name: quality
          type: llm_judge
          prompt: ./prompts/quality.md
      aggregator:
        type: weighted_average
        weights:
          safety: 0.3
          quality: 0.7
```

See `references/composite-evaluator.md` for aggregation types and patterns.

### Batch CLI Evaluation
Evaluate external batch runners that process all evalcases in one invocation:

```yaml
description: Batch CLI evaluation
execution:
  target: batch_cli

evalcases:
  - id: case-001
    expected_outcome: Returns decision=CLEAR
    expected_messages:
      - role: assistant
        content:
          decision: CLEAR
    input_messages:
      - role: user
        content:
          row:
            id: case-001
            amount: 5000
    execution:
      evaluators:
        - name: decision-check
          type: code_judge
          script: bun run ./scripts/check-output.ts
          cwd: .
```

**Key pattern:**
- Batch runner reads eval YAML via `--eval` flag, outputs JSONL keyed by `id`
- Each evalcase has its own evaluator to validate its corresponding output
- Use structured `expected_messages.content` for expected output fields

See `references/batch-cli-evaluator.md` for full implementation guide.

## Example
```yaml
description: Example showing basic features and conversation threading
execution:
  target: default

evalcases:
  - id: code-review-basic
    expected_outcome: Assistant provides helpful code analysis
    
    input_messages:
      - role: system
        content: You are an expert code reviewer.
      - role: user
        content:
          - type: text
            value: |-
              Review this function:
              
              ```python
              def add(a, b):
                  return a + b
              ```
          - type: file
            value: /prompts/python.instructions.md
    
    expected_messages:
      - role: assistant
        content: |-
          The function is simple and correct. Suggestions:
          - Add type hints: `def add(a: int, b: int) -> int:`
          - Add docstring
          - Consider validation for edge cases
```
