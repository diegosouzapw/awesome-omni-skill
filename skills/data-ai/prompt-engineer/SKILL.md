---
name: prompt-engineer
description: Transform rough prompts/ideas into production-ready LLM prompts. Use when crafting, refining, or optimizing prompts for any AI model (Claude, GPT, Llama, etc.) with advanced techniques like CoT, constitutional AI, RAG optimization.
---

# Prompt Engineer

Expert prompt engineering skill that transforms rough ideas into well-structured, production-ready prompts optimized for LLMs.

## When to Activate

- User provides a rough prompt/idea and wants it refined
- User asks to create/design/optimize a prompt for any LLM
- User needs prompt architecture for agents, RAG, or multi-step workflows
- User asks about prompting techniques or best practices

## Workflow

### 1. Analyze Input

Identify from user's request:
- **Target model** (Claude, GPT, Llama, etc.) — default: Claude
- **Use case** (agent system prompt, task prompt, RAG, chat, etc.)
- **Domain** (technical, creative, business, etc.)
- **Constraints** (token limits, output format, safety requirements)

### 2. Apply Techniques

Select appropriate techniques from `references/techniques.md` based on use case:
- Complex reasoning → Chain-of-Thought, Tree-of-Thoughts
- Safety-critical → Constitutional AI patterns
- Data extraction → Structured output, JSON mode
- Multi-step tasks → Prompt chaining, agent patterns
- Knowledge-heavy → RAG optimization

### 3. Craft the Prompt

Follow model-specific guidelines from `references/model-optimization.md`:
- Structure with clear sections (role, context, instructions, output format)
- Include examples where beneficial (few-shot)
- Add constraints and guardrails
- Optimize for token efficiency

### 4. Deliver Output

**MANDATORY format** — always include ALL sections:

#### The Prompt
Display complete prompt in a single copyable code block.

#### Implementation Notes
- Techniques used and rationale
- Model-specific optimizations
- Parameter recommendations (temperature, max_tokens)
- Expected behavior and output format

#### Testing & Evaluation
- 3-5 test cases to validate
- Edge cases and failure modes
- Optimization suggestions

#### Usage Guidelines
- When/how to use effectively
- Customization options
- Integration considerations

## Key Principles

- **Always show the complete prompt** — never just describe it
- **Token efficiency** — concise but comprehensive
- **Production-ready** — reliable, safe, optimized
- **Model-aware** — tailor to target model's strengths
- Refer to `references/techniques.md` for advanced technique details
- Refer to `references/model-specific-optimization-guide.md` for model-specific guidance
- Refer to `references/production-patterns-and-enterprise-templates.md` for enterprise patterns
