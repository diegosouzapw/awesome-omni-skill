---
name: ai-agent-development
description: 'Build production-ready AI agents with Microsoft Foundry and Agent Framework. Use when creating AI agents, selecting LLM models, implementing agent orchestration, adding tracing/observability, or evaluating agent quality. Covers agent architecture, model selection, multi-agent workflows, and production deployment.'
---

# AI Agent Development

> **Purpose**: Build production-ready AI agents with Microsoft Foundry and Agent Framework.  
> **Scope**: Agent architecture, model selection, orchestration, observability, evaluation.

---

## When to Use This Skill

- Building AI agents with Microsoft Foundry or Agent Framework
- Selecting LLM models for agent scenarios
- Implementing multi-agent orchestration workflows
- Adding tracing and observability to AI agents
- Evaluating agent quality and response accuracy

## Prerequisites

- Python 3.11+ or .NET 8+
- agent-framework-azure-ai package
- Microsoft Foundry workspace with deployed model

## Quick Start

### Installation

**Python** (Recommended):
```bash
pip install agent-framework-azure-ai --pre  # --pre required during preview
```

**.NET**:
```bash
dotnet add package Microsoft.Agents.AI.AzureAI --prerelease
dotnet add package Microsoft.Agents.AI.Workflows --prerelease
```

### Model Selection

**Top Production Models** (Microsoft Foundry):

| Model | Best For | Context | Cost/1M |
|-------|----------|---------|---------|
| **gpt-5.2** | Enterprise agents, structured outputs | 200K/100K | TBD |
| **gpt-5.1-codex-max** | Agentic coding workflows | 272K/128K | $3.44 |
| **claude-opus-4-5** | Complex agents, coding, computer use | 200K/64K | $10 |
| **gpt-5.1** | Multi-step reasoning | 200K/100K | $3.44 |
| **o3** | Advanced reasoning | 200K/100K | $3.5 |

**Deploy Model**: `Ctrl+Shift+P` → `AI Toolkit: Deploy Model`

---

## Agent Patterns

### Single Agent

```python
from pathlib import Path
from agent_framework.openai import OpenAIChatClient

# Load prompt from file — NEVER embed prompts as inline strings
prompt = Path("prompts/assistant.md").read_text(encoding="utf-8")

client = OpenAIChatClient(
    model="gpt-5.1",
    api_key=os.getenv("FOUNDRY_API_KEY"),
    endpoint=os.getenv("FOUNDRY_ENDPOINT")
)

agent = {
    "name": "Assistant",
    "instructions": prompt,  # Loaded from prompts/assistant.md
    "tools": []  # Add tools as needed
}

response = await client.chat(
    messages=[{"role": "user", "content": "Hello"}],
    agent=agent
)
```

### Multi-Agent Orchestration

```python
from pathlib import Path
from agent_framework.workflows import SequentialWorkflow

# Each agent loads its prompt from a dedicated file
researcher = {
    "name": "Researcher",
    "instructions": Path("prompts/researcher.md").read_text(encoding="utf-8")
}
writer = {
    "name": "Writer",
    "instructions": Path("prompts/writer.md").read_text(encoding="utf-8")
}

workflow = SequentialWorkflow(
    agents=[researcher, writer],
    handoff_strategy="on_completion"
)

result = await workflow.run(query="Write about AI agents")
```

**Advanced Patterns**: Search [github.com/microsoft/agent-framework](https://github.com/microsoft/agent-framework) for:
- Group Chat, Concurrent, Conditional, Loop
- Human-in-the-Loop, Reflection, Fan-out/Fan-in
- MCP, Multimodal, Custom Executors

---

## Best Practices

### Prompt & Template File Management

> **RULE**: NEVER embed prompts or output templates as inline strings in code. Always store them as separate files.

**Why**: Prompts are content, not code. Separating them enables:
- Version control diffs that show exactly what changed in a prompt
- Non-developer editing (PMs, prompt engineers) without touching code
- A/B testing different prompts without code changes
- Reuse across agents, languages, and test harnesses
- Clear separation of concerns (logic vs. content)

**Directory Convention**:
```
project/
  prompts/                    # All system/agent prompts
    assistant.md              # One file per agent role
    researcher.md
    writer.md
    reviewer.md
  templates/                  # Output templates used by agents
    report-template.md        # Structured output templates
    email-template.md
    summary-template.md
  config/
    models.yaml               # Model configuration
```

**Loading Pattern**:
```python
from pathlib import Path

# Load prompt
prompt = Path("prompts/assistant.md").read_text(encoding="utf-8")

# Load output template and inject into prompt
template = Path("templates/report-template.md").read_text(encoding="utf-8")
prompt_with_template = f"{prompt}\n\n## Output Format\n{template}"
```

**Rules**:
- MUST store all system prompts in `prompts/` directory as `.md` or `.txt` files
- MUST store output format templates in `templates/` directory
- MUST NOT embed prompt text longer than one sentence directly in code
- SHOULD use Markdown format for prompts (readable, supports structure)
- SHOULD name files after the agent role: `prompts/{agent-name}.md`
- SHOULD include a brief comment header in each prompt file (purpose, version, model target)
- MAY use template variables (`{variable}`) for dynamic content injected at runtime

### Development

✅ **DO**:
- Plan agent architecture before coding (Research → Design → Implement)
- Use Microsoft Foundry models for production
- Implement tracing from day one
- Test with evaluation datasets before deployment
- Use structured outputs for reliable agent responses
- Implement error handling and retry logic
- Version your agents and track changes
- **Store all prompts as separate files in `prompts/` directory**
- **Store output templates as separate files in `templates/` directory**

❌ **DON'T**:
- Hardcode API keys or endpoints
- Embed prompts or output templates as multi-line strings in code
- Skip tracing setup (critical for debugging)
- Deploy without evaluation
- Use GitHub models in production (free tier has limits)
- Ignore token limits and context windows
- Mix agent logic with business logic

### Security

- Store credentials in environment variables or Azure Key Vault
- Validate all tool inputs and outputs
- Implement rate limiting for agent APIs
- Log agent actions for audit trails
- Use role-based access control (RBAC) for Foundry resources
- Review OWASP Top 10 for AI: [owasp.org/AI-Security-and-Privacy-Guide](https://owasp.org/www-project-ai-security-and-privacy-guide/)

### Performance

- Cache model responses when appropriate
- Use batch processing for multiple requests
- Monitor token usage and costs
- Implement timeout handling
- Use async/await for I/O operations
- Consider model size vs. latency tradeoffs

### Monitoring

- Track key metrics: latency, success rate, token usage, cost
- Set up alerts for failures and anomalies
- Use structured logging with context
- Integrate with Azure Monitor / Application Insights
- Review traces regularly for optimization opportunities

---

## Production Checklist

**Development**
- [ ] Agent architecture documented
- [ ] Model selected and deployed
- [ ] Tools/plugins implemented and tested
- [ ] Error handling with retries
- [ ] Structured outputs configured
- [ ] No hardcoded secrets
- [ ] All prompts stored as separate files in `prompts/` (not inline in code)
- [ ] All output templates stored in `templates/` (not inline in code)

**Model Change Management (MANDATORY)**
- [ ] Model version pinned explicitly (e.g., `gpt-5.1-2026-01-15`)
- [ ] Model version configurable via environment variable
- [ ] Evaluation baseline saved for current model
- [ ] A/B evaluation run before any model switch
- [ ] Structured output schema verified after model change
- [ ] Tool/function-calling accuracy verified after model change
- [ ] Model change documented in changelog with eval results
- [ ] Weekly evaluation monitoring configured for drift detection
- [ ] Alert threshold set for score drops > 10% from baseline

**Model Change Test Automation (MANDATORY)**
- [ ] Agent designed as model-agnostic (model injected via config)
- [ ] `config/models.yaml` defines model test matrix with thresholds
- [ ] Tested against ≥2 models (primary + fallback from different provider)
- [ ] Multi-model comparison pipeline in CI/CD (weekly + on model config change)
- [ ] Deployment gated on threshold checks (CI fails on regression)
- [ ] Validated fallback model designated and documented
- [ ] Comparison report generated per run (JSON + human-readable)
- [ ] Cost and latency evaluators included alongside quality metrics

**Observability**
- [ ] OpenTelemetry tracing enabled
- [ ] Trace viewer tested
- [ ] Structured logging implemented
- [ ] Metrics collection configured

**Evaluation**
- [ ] Evaluation dataset created
- [ ] Evaluators defined (built-in + custom)
- [ ] Evaluation runs passing
- [ ] Results meet quality thresholds
- [ ] Multi-model comparison run (2+ models tested)
- [ ] Fallback model validated and documented
- [ ] Model comparison baseline saved

**Security & Compliance**
- [ ] Credentials in Key Vault/env vars
- [ ] Input validation implemented
- [ ] RBAC configured
- [ ] Audit logging enabled
- [ ] OWASP AI Top 10 reviewed

**Operations**
- [ ] Health checks implemented
- [ ] Rate limiting configured
- [ ] Monitoring alerts set up
- [ ] Deployment strategy defined
- [ ] Rollback plan documented
- [ ] Cost monitoring enabled

---

## Resources

**Official Documentation**:
- Agent Framework: [github.com/microsoft/agent-framework](https://github.com/microsoft/agent-framework)
- Microsoft Foundry: [ai.azure.com](https://ai.azure.com)
- Azure AI Projects SDK: [learn.microsoft.com/python/api/overview/azure/ai-projects](https://learn.microsoft.com/python/api/overview/azure/ai-projects)
- OpenTelemetry: [opentelemetry.io](https://opentelemetry.io)

**AI Toolkit**:
- Model Catalog: `Ctrl+Shift+P` → `AI Toolkit: Model Catalog`
- Trace Viewer: `Ctrl+Shift+P` → `AI Toolkit: Open Trace Viewer`
- Playground: `Ctrl+Shift+P` → `AI Toolkit: Model Playground`

**Security**:
- OWASP AI Security: [owasp.org/AI-Security-and-Privacy-Guide](https://owasp.org/www-project-ai-security-and-privacy-guide/)
- Azure Security Best Practices: [learn.microsoft.com/azure/security](https://learn.microsoft.com/azure/security)

---

**Related**: [AGENTS.md](../../../../AGENTS.md) for agent behavior guidelines • [Skills.md](../../../../Skills.md) for general production practices

**Last Updated**: January 17, 2026


## Scripts

| Script | Purpose | Usage |
|--------|---------|-------|
| [`scaffold-agent.py`](scripts/scaffold-agent.py) | Scaffold AI agent project (Python/.NET) with tracing & eval | `python scripts/scaffold-agent.py --name my-agent [--pattern multi-agent] [--with-eval]` |
| [`validate-agent-checklist.ps1`](scripts/validate-agent-checklist.ps1) | Validate agent project against production checklist | `./scripts/validate-agent-checklist.ps1 [-Path ./my-agent] [-Strict]` |
| [`check-model-drift.ps1`](scripts/check-model-drift.ps1) | Validate model pinning, data drift signals, and judge LLM readiness | `./scripts/check-model-drift.ps1 [-Path ./my-agent] [-Strict]` |
| [`run-model-comparison.py`](scripts/run-model-comparison.py) | Run eval suite against multiple models and generate comparison report | `python scripts/run-model-comparison.py --config config/models.yaml --dataset evaluation/core.jsonl` |

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Model not found | Verify model deployment in Foundry portal and check endpoint URL |
| Tracing not appearing | Ensure AIInferenceInstrumentor().instrument() called before agent creation |
| Agent loops indefinitely | Set max_turns limit and add termination conditions |

## References

- [Tracing And Evaluation](references/tracing-and-evaluation.md)
- [Multi Model Patterns](references/multi-model-patterns.md)
- [Model Drift And Judge Patterns](references/model-drift-judge-patterns.md)
- [Model Change Test Automation](references/model-change-test-automation.md)