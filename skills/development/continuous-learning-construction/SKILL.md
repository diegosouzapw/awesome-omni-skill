---
name: "continuous-learning-construction"
description: "Automatically extract patterns, best practices, and reusable knowledge from construction automation sessions to improve future performance."
homepage: "https://datadrivenconstruction.io"
metadata: {"openclaw": {"emoji": "🛡️", "os": ["win32"], "homepage": "https://datadrivenconstruction.io", "requires": {"bins": ["python3"]}}}
---
# Continuous Learning for Construction Automation

This skill enables automatic extraction of valuable patterns, solutions, and best practices from construction automation sessions to build institutional knowledge.

## When to Use

Activate this skill:
- At the end of complex estimation sessions
- After solving non-trivial data processing problems
- When discovering new integration patterns
- After completing successful document processing
- When developing new automation workflows

## Pattern Extraction Framework

### 1. Session Analysis

```python
class ConstructionSessionAnalyzer:
    """Extract learnings from automation sessions"""

    # Categories of learnable patterns
    PATTERN_CATEGORIES = [
        'data_processing',      # Data transformation patterns
        'estimation',           # Cost estimation techniques
        'scheduling',           # Schedule optimization patterns
        'integration',          # API/system integration patterns
        'document_processing',  # Document handling patterns
        'quality_assurance',    # Validation and QA patterns
        'error_handling',       # Error resolution patterns
        'optimization'          # Performance optimization patterns
    ]

    def analyze_session(self, session_log: list) -> dict:
        """Extract patterns from session history"""

        patterns = {
            'successful_solutions': [],
            'error_resolutions': [],
            'optimization_discoveries': [],
            'integration_patterns': [],
            'reusable_code': [],
            'decision_rationales': []
        }

        for entry in session_log:
            if self._is_solution(entry):
                patterns['successful_solutions'].append(
                    self._extract_solution_pattern(entry)
                )

            if self._is_error_resolution(entry):
                patterns['error_resolutions'].append(
                    self._extract_error_pattern(entry)
                )

            if self._is_optimization(entry):
                patterns['optimization_discoveries'].append(
                    self._extract_optimization(entry)
                )

        return patterns
```

### 2. Knowledge Categories for Construction

#### 2.1 Cost Estimation Patterns

```yaml
# Example learned pattern
pattern:
  name: "electrical_cost_adjustment_pattern"
  category: "estimation"
  context: "When estimating electrical work for high-rise buildings"
  problem: "Standard rates don't account for vertical transportation costs"
  solution: |
    Apply height factor multiplier:
    - Floors 1-5: 1.0x base rate
    - Floors 6-15: 1.15x base rate
    - Floors 16-30: 1.25x base rate
    - Floors 30+: 1.35x base rate
  confidence: 0.85
  source_sessions: ["session_2026_01_15", "session_2026_01_20"]
  validations: 3
```

#### 2.2 BIM Data Processing Patterns

```yaml
pattern:
  name: "revit_level_extraction"
  category: "data_processing"
  context: "Extracting elements by level from Revit exports"
  problem: "Elements sometimes missing level association"
  solution: |
    1. First check 'Level' parameter
    2. If missing, check 'Reference Level' parameter
    3. If still missing, derive from bounding box Z coordinate
    4. Map Z ranges to known level elevations
  code_snippet: |
    def get_element_level(element: dict, levels: list) -> str:
        # Direct level parameter
        if level := element.get('Level'):
            return level

        # Reference level fallback
        if ref_level := element.get('Reference Level'):
            return ref_level

        # Derive from geometry
        z_coord = element['BoundingBox']['Min']['Z']
        return find_nearest_level(z_coord, levels)
  confidence: 0.92
```

#### 2.3 Integration Patterns

```yaml
pattern:
  name: "procore_rate_limit_handling"
  category: "integration"
  context: "Syncing data with Procore API"
  problem: "API returns 429 Too Many Requests during bulk operations"
  solution: |
    Implement exponential backoff with jitter:
    1. Initial delay: 1 second
    2. Multiply by 2 on each retry
    3. Add random jitter (0-500ms)
    4. Max retries: 5
    5. Max delay: 32 seconds
  code_snippet: |
    async def procore_request_with_retry(url, data):
        delay = 1
        for attempt in range(5):
            try:
                response = await procore_api.post(url, data)
                return response
            except RateLimitError:
                jitter = random.uniform(0, 0.5)
                await asyncio.sleep(delay + jitter)
                delay *= 2
        raise MaxRetriesExceeded()
  confidence: 0.95
```

#### 2.4 Error Resolution Patterns

```yaml
pattern:
  name: "cwicr_no_match_resolution"
  category: "error_handling"
  context: "CWICR semantic search returns no relevant matches"
  problem: "Query too specific or uses non-standard terminology"
  solution: |
    Resolution steps:
    1. Simplify query to core concepts
    2. Remove brand names and specifications
    3. Try alternative terminology (US vs UK terms)
    4. Expand search to parent category
    5. If still no match, flag for manual mapping
  examples:
    - original: "Kohler K-4519 wall-mounted water closet"
      simplified: "wall mounted toilet"
    - original: "Lutron Caseta wireless dimmer switch"
      simplified: "dimmer switch"
  confidence: 0.88
```

### 3. Learning Pipeline

```python
class ConstructionLearningPipeline:
    """Continuous learning pipeline for construction automation"""

    def __init__(self, knowledge_base_path: str):
        self.kb_path = knowledge_base_path
        self.patterns = self._load_patterns()

    def learn_from_session(self, session: dict) -> list:
        """Extract and store learnings from session"""

        # Analyze session
        analyzer = ConstructionSessionAnalyzer()
        new_patterns = analyzer.analyze_session(session['log'])

        # Validate patterns
        validated = []
        for pattern in new_patterns['successful_solutions']:
            if self._validate_pattern(pattern):
                # Check if similar pattern exists
                existing = self._find_similar_pattern(pattern)
                if existing:
                    # Reinforce existing pattern
                    self._reinforce_pattern(existing, pattern)
                else:
                    # Add new pattern
                    self._add_pattern(pattern)
                validated.append(pattern)

        # Persist to knowledge base
        self._save_patterns()

        return validated

    def apply_learnings(self, context: dict) -> list:
        """Retrieve relevant patterns for current context"""

        relevant_patterns = []

        for pattern in self.patterns:
            similarity = self._calculate_similarity(pattern['context'], context)
            if similarity > 0.7:
                relevant_patterns.append({
                    'pattern': pattern,
                    'relevance': similarity
                })

        return sorted(relevant_patterns, key=lambda x: x['relevance'], reverse=True)

    def _validate_pattern(self, pattern: dict) -> bool:
        """Validate pattern before adding to knowledge base"""

        # Check minimum confidence
        if pattern.get('confidence', 0) < 0.6:
            return False

        # Check for code quality (if code snippet)
        if code := pattern.get('code_snippet'):
            if not self._is_valid_code(code):
                return False

        # Check for completeness
        required_fields = ['name', 'category', 'context', 'solution']
        if not all(f in pattern for f in required_fields):
            return False

        return True
```

### 4. Knowledge Base Structure

```
knowledge_base/
├── patterns/
│   ├── estimation/
│   │   ├── height_factors.yaml
│   │   ├── material_adjustments.yaml
│   │   └── labor_productivity.yaml
│   ├── data_processing/
│   │   ├── revit_extraction.yaml
│   │   ├── ifc_parsing.yaml
│   │   └── excel_transformations.yaml
│   ├── integration/
│   │   ├── procore_patterns.yaml
│   │   ├── plangrid_patterns.yaml
│   │   └── webhook_handlers.yaml
│   └── error_handling/
│       ├── cwicr_resolutions.yaml
│       ├── api_errors.yaml
│       └── data_validation.yaml
├── code_snippets/
│   ├── python/
│   ├── javascript/
│   └── sql/
├── decision_trees/
│   ├── estimate_type_selection.yaml
│   ├── schedule_method_selection.yaml
│   └── integration_approach.yaml
└── metrics/
    ├── pattern_usage.json
    └── success_rates.json
```

### 5. Session End Learning Prompt

At the end of each construction automation session:

```markdown
## Session Learning Review

### What Worked Well
- [Successful approaches discovered]
- [Efficient patterns used]
- [Integrations that worked smoothly]

### Challenges Overcome
- [Errors encountered and how resolved]
- [Workarounds developed]
- [Edge cases handled]

### New Patterns Discovered
- [Novel approaches to problems]
- [Optimization techniques found]
- [Reusable code created]

### Knowledge to Preserve
- [Key learnings to remember]
- [Context-specific solutions]
- [Client/project-specific adaptations]

### Recommendations for Future
- [Improvements to suggest]
- [Patterns to apply elsewhere]
- [Automation opportunities identified]
```

### 6. Pattern Application

When starting new construction tasks:

```python
def suggest_approaches(task_context: dict) -> list:
    """Suggest learned approaches for new tasks"""

    pipeline = ConstructionLearningPipeline('knowledge_base/')
    relevant = pipeline.apply_learnings(task_context)

    suggestions = []
    for item in relevant[:5]:  # Top 5 suggestions
        pattern = item['pattern']
        suggestions.append({
            'name': pattern['name'],
            'relevance': f"{item['relevance']*100:.0f}%",
            'summary': pattern['solution'][:200],
            'confidence': pattern['confidence'],
            'previous_uses': pattern.get('usage_count', 0)
        })

    return suggestions
```

## Integration with Other Skills

This skill works with:
- **verification-loop-construction**: Learn from verification failures
- **security-review-construction**: Capture security patterns
- **estimation skills**: Build estimation knowledge base
- **integration skills**: Capture API patterns

## Usage Commands

```bash
# Extract learnings from current session
/learn

# View patterns for current context
/suggest-patterns

# Add manual pattern
/add-pattern --category estimation --name "my_pattern"

# Export knowledge base
/export-kb --format yaml
```

---

**Every session is an opportunity to learn. Capture knowledge to compound expertise over time.**
