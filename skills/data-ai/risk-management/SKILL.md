---
name: risk-management
cluster: financial
description: "Manages financial risks through quantitative analysis, modeling, and mitigation strategies."
tags: ["finance","risk-management","quantitative-analysis"]
dependencies: []
composes: []
similar_to: []
called_by: []
authorization_required: false
scope: general
model_hint: claude-sonnet
embedding_hint: "financial risk management analysis mitigation strategies quantitative"
---

# risk-management

## Purpose
This skill enables quantitative analysis, modeling, and mitigation of financial risks. It processes data to calculate metrics like Value at Risk (VaR), stress testing, and suggests strategies to reduce exposure, such as hedging or diversification.

## When to Use
Use this skill for scenarios involving financial uncertainty, like portfolio risk assessment, credit risk evaluation, or market volatility analysis. Apply it when you need data-driven insights to comply with regulations (e.g., Basel III) or optimize investment decisions.

## Key Capabilities
- Perform VaR calculations using historical or Monte Carlo simulations.
- Build risk models for market, credit, or operational risks with inputs like asset prices or default probabilities.
- Generate mitigation strategies, such as recommending stop-loss levels or portfolio rebalancing based on risk thresholds.
- Integrate with data sources for real-time analysis, supporting formats like CSV, JSON, or API feeds.
- Output results in structured formats, including reports or JSON for further processing.

## Usage Patterns
Always initialize with authentication via `$OPENCLAW_API_KEY`. For CLI, pipe data inputs directly; for API, use asynchronous calls for large datasets. Start by loading configuration files (e.g., YAML for model parameters). Common pattern: Analyze risk -> Review outputs -> Apply mitigation. For code integration, import the SDK and wrap calls in try-except blocks. Example 1: Analyze a stock portfolio's market risk by providing historical prices. Example 2: Evaluate credit risk for a loan portfolio and generate mitigation recommendations.

## Common Commands/API
Use the OpenClaw CLI for quick tasks or the REST API for programmatic access. Authentication requires setting `$OPENCLAW_API_KEY` in your environment.

- CLI Command: `openclaw risk analyze --type market --model var --input portfolio.csv --confidence 95`  
  This calculates 95% VaR for market risk; output is a JSON file with metrics.

- API Endpoint: POST https://api.openclaw.ai/v1/risk/analyze  
  Body: `{"type": "credit", "data": {"loans": [{"amount": 100000, "rating": "A"}]}, "model": "default-prob"}`  
  Response: JSON object with risk score and strategies, e.g., `{"var": 5000, "mitigation": ["increase collateral"]}`.

- Code Snippet (Python):  
  ```python
  import openclaw
  openclaw.set_key(os.environ['OPENCLAW_API_KEY'])
  result = openclaw.risk.analyze(type='operational', data={'events': [100, 200]}, model='monte-carlo')
  print(result['mitigation'])
  ```

- Config Format: YAML for custom models, e.g.,  
  ```
  model:
    type: var
    parameters:
      window: 252  # trading days
      confidence: 0.95
  ```

## Integration Notes
Integrate by setting `$OPENCLAW_API_KEY` and using the SDK in your application. For web apps, handle webhooks for asynchronous results (e.g., POST to your endpoint on completion). Connect to data providers like Bloomberg via custom adapters; specify in config: `{"data_source": "bloomberg", "api_endpoint": "https://api.bloomberg.com/data"}`. Ensure compatibility with other OpenClaw skills by chaining outputs, e.g., pipe risk analysis results into a financial-analysis skill.

## Error Handling
Always validate inputs before commands (e.g., check for required fields like `--input`). For API calls, catch HTTP errors: if status >= 400, retry up to 3 times with exponential backoff. Common errors: 401 (unauthorized – check `$OPENCLAW_API_KEY`), 400 (bad request – verify JSON schema), or 500 (server error – log and notify). In code, use:  
```python
try:
    result = openclaw.risk.analyze(...)
except openclaw.APIError as e:
    if e.status == 401:
        print("Reauthenticate with $OPENCLAW_API_KEY")
    else:
        raise
```
Log all errors with timestamps and include debug flags, e.g., `openclaw risk analyze --debug`.

## Graph Relationships
- Related to: financial-analysis (shares finance tag for combined data processing), portfolio-management (uses risk outputs for optimization).
- Connected via: quantitative-analysis (common modeling techniques), mitigation-strategies (links to compliance tools).
- Dependencies: Requires financial cluster skills for data input; provides outputs for decision-making skills.
