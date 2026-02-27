---
name: PolicyPulse Development
description: AI-powered synthetic population simulator with clean architecture, Python type safety, and professional dashboard design patterns
---

# PolicyPulse Development

This skill guides AI agents in developing PolicyPulse, a professional analytical tool for stress-testing policies against synthetic populations using hybrid AI (LLM + Neural Network).

## When to use

Activate this skill when:
- Building or modifying PolicyPulse application code
- Creating modular Python components following clean architecture
- Designing dashboard UI components for Streamlit
- Implementing type-safe Python code with proper annotations
- Writing tests for PolicyPulse functionality

## Core Principles

### 1. Document-Driven Development
- **Source of Truth**: PRD.md, DESIGN_DOC.md, and TECH_STACK.md are authoritative
- **No Feature Invention**: Only implement features explicitly defined in documents
- **Incremental Progress**: Small, safe steps that keep the project runnable
- **No Summary Documents**: Don't create documentation unless explicitly requested

### 2. Clean Architecture (Modular Monolith)
Follow the defined module structure:
```
app.py              # Entry point, session management
ui_sections.py      # UI component rendering  
simulation.py       # Simulation orchestration
llm_client.py       # Gemini API wrapper
nn_model.py         # Neural network training/inference
population.py       # Synthetic citizen generation
stats.py            # Aggregation and statistics
data_models.py      # Dataclass definitions
utils.py            # Helper functions
config.py           # Environment loading
ml_data.py          # Training dataset management
```

**Module Dependency Rules**:
- `data_models.py` has no dependencies (stdlib only)
- `config.py` is imported by app.py only
- UI layers depend on business logic, not vice versa
- External dependencies (genai, sklearn) isolated in specific modules

### 3. Python Type Safety
- **Always use type hints**: Function signatures, variables, return types
- **Leverage Python 3.10+ features**: 
  - Dataclasses for data models
  - Union types with `|` syntax (e.g., `str | None`)
  - Type aliases for complex types
- **Example Pattern**:
```python
from dataclasses import dataclass
from typing import Literal

@dataclass
class Citizen:
    id: int
    age: int
    income_level: Literal["Low", "Middle", "High"]
    happiness: float
    
def calculate_average(citizens: list[Citizen]) -> float:
    return sum(c.happiness for c in citizens) / len(citizens)
```

### 4. Professional Dashboard Design
- **Data Density without Overwhelm**: Show comprehensive info in digestible chunks
- **Progressive Disclosure**: Start simple, reveal complexity on demand
- **Sidebar + Main Content**: Sidebar for config (300-320px fixed), main area for results
- **Muted Professional Palette**: 
  - Primary: Deep Purple (#667EEA)
  - Semantic: Teal (positive), Coral (negative)
  - Neutrals: Charcoal text, Pearl backgrounds
- **Tabbed Analysis**: Overview, Demographics, Individuals, Experts, AI Insights, Compare
- **Metric Cards**: Large bold values, small change indicators with colored arrows
- **Responsible AI**: Prominent disclaimer that this is synthetic simulation, not prediction

### 5. Streamlit-Specific Patterns
- **Session State**: Use `st.session_state` for simulation results, populations, models
- **File Persistence**: Save models to `models/`, training data to `data/` as CSV/Joblib
- **Component Organization**: Separate UI rendering from business logic
- **Layout Structure**:
```python
# Sidebar config
with st.sidebar:
    # Population, steps, mode, policy config
    pass

# Main content with tabs
tab1, tab2, tab3 = st.tabs(["Overview", "Demographics", "Individuals"])
with tab1:
    # Metrics cards, time-series charts
    pass
```

## Implementation Guidelines

### Data Models First
1. Define dataclasses in `data_models.py` with full type annotations
2. Include docstrings explaining purpose and constraints
3. Use `frozen=True` for immutable data

### Business Logic Isolation
1. Keep simulation logic in `simulation.py`, `population.py`, `nn_model.py`
2. No Streamlit imports in business logic modules
3. Return structured data (dataclasses, DataFrames), not UI components

### UI Components
1. UI rendering functions in `ui_sections.py`
2. Accept typed data structures as parameters
3. Use Plotly for charts (`st.plotly_chart(fig, use_container_width=True)`)
4. Follow design system: spacing (8px base), typography (Inter font), colors

### Error Handling
1. Graceful degradation: LLM → NN → Rule-based fallback
2. User-friendly error messages in UI
3. Log technical details for debugging
4. Never crash the app; show error state and recovery options

### Testing Patterns
1. Unit tests for business logic (population generation, stats calculations)
2. Integration tests for simulation workflow
3. Mock external dependencies (Gemini API)
4. Validate type contracts with pytest

## File Organization

```
policypulse/
├── .streamlit/
│   └── config.toml          # Streamlit configuration
├── data/
│   └── llm_training_samples.csv  # Persistent training data
├── models/
│   ├── citizen_reaction_model.joblib  # Trained NN
│   └── feature_scaler.joblib          # Feature normalization
├── src/                     # Source code modules
│   ├── app.py
│   ├── ui_sections.py
│   ├── simulation.py
│   ├── llm_client.py
│   ├── nn_model.py
│   ├── population.py
│   ├── stats.py
│   ├── data_models.py
│   ├── utils.py
│   ├── config.py
│   └── ml_data.py
├── tests/                   # Test files
├── requirements.txt         # Python dependencies
├── .env.example            # Template for API keys
├── .gitignore
└── README.md
```

## Technology Stack Constraints

- **Python**: 3.10+ only (for dataclasses, type hints, pattern matching)
- **Web Framework**: Streamlit (no React, Flask, or alternatives)
- **LLM**: Google Gemini (free tier: 15 req/min, 200/day)
- **ML**: Scikit-learn MLPRegressor (no PyTorch, TensorFlow)
- **Data**: Pandas + NumPy (no Spark, Dask)
- **Charts**: Plotly (no Matplotlib, Altair)
- **Persistence**: File-based CSV/Joblib (no databases)

## Anti-Patterns to Avoid

❌ **Don't**:
- Invent features not in PRD
- Create React/JavaScript frontends
- Use databases (PostgreSQL, MongoDB, etc.)
- Add user authentication/accounts
- Create pixel-perfect custom CSS (accept Streamlit's design)
- Parallelize with threading (Python GIL limitations)
- Make predictive accuracy claims in UI

✅ **Do**:
- Follow the three source documents strictly
- Use Streamlit's built-in components
- Implement file-based persistence
- Maintain session-based architecture
- Show prominent "synthetic simulation" disclaimers
- Implement graceful LLM fallbacks

## Quick Reference

### Adding a New Feature
1. Verify it's in PRD.md
2. Check DESIGN_DOC.md for UI specs
3. Review TECH_STACK.md for implementation patterns
4. Create/update data models if needed
5. Implement business logic (pure Python)
6. Add UI rendering (Streamlit components)
7. Write tests
8. Keep project runnable at each step

### Code Review Checklist
- [ ] Type hints on all functions and class attributes
- [ ] Dataclasses used for structured data
- [ ] No Streamlit imports in business logic modules
- [ ] Error handling with graceful degradation
- [ ] Follows design system (colors, spacing, typography)
- [ ] Matches PRD feature specifications exactly
- [ ] No invented functionality outside documents
