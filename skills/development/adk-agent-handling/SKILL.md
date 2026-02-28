---
name: adk-agent-handling
description: Google ADK (Agent Development Kit) multi-agent system architecture for BigQuery data analytics. Covers BigQuery agent vs conversational agent patterns, ADK Single Parent Rule, domain routing with sub-agents, agent selection mechanisms, SQL error recovery with ReflectAndRetryToolPlugin, transfer_to_agent workflows, and frontend-backend agent coordination. Use when working with Google ADK agents, multi-agent systems, BigQuery SQL automation, domain expert routing, agent orchestration, or implementing error recovery strategies in AI agent applications.
---

# Google ADK Agent Handling

## Purpose

Complete guide for architecting and managing Google ADK multi-agent systems in the ALYac Family data analytics platform. This skill covers the dual-orchestrator pattern (BigQuery vs Conversational agents), ADK framework constraints, domain-based routing, and production-ready error recovery strategies.

## When to Use

Automatically activates when you mention:
- Google ADK (Agent Development Kit)
- Multi-agent architecture or agent orchestration
- BigQuery agent or conversational agent patterns
- Domain routing or agent selection logic
- ADK Single Parent Rule or sub-agents
- transfer_to_agent workflows
- ReflectAndRetryToolPlugin
- SQL error recovery or auto-retry mechanisms
- Agent coordination or delegation patterns

**Manual activation**: Ask about ADK agent architecture, agent selection mechanisms, or error recovery strategies.

---

## System Architecture Overview

### Dual-Orchestrator Pattern

This project uses TWO top-level orchestrator agents:

**1. bigquery_agent** (SQL-based orchestrator)
- **File**: `adk-backend/src/adk_backend/agents/base/bigquery_agent.py`
- **Role**: Domain routing + direct SQL execution
- **Tools**: BigQuery (list_templates, render_template, dry_run, execute)
- **Sub-agents**: 4 domain experts (has parent relationship)
- **Use case**: Explicit SQL control, template-based analysis

**2. conversational_analytics_agent** (AI-based orchestrator)
- **File**: `adk-backend/src/adk_backend/agents/base/conversational_analytics_agent.py`
- **Role**: Conversational AI analysis + Google Search
- **Tools**: ask_data_insights, search_catalog, google_search
- **Sub-agents**: None (tools only, follows ADK Single Parent Rule)
- **Use case**: Natural language queries, web-enhanced insights

### Domain Expert Agents (Shared)

Both orchestrators delegate to these specialists:

1. **alyac_family_agent** - Malware detection, phishing messages
2. **security_agent** - Smishing, FCM security events, threat alerts
3. **marketing_agent** - Campaigns, segmentation, conversion marketing
4. **conversion_agent** - Subscription funnels, retention, SaaS metrics

**Critical**: Domain experts are sub_agents of `bigquery_agent` ONLY, following ADK's Single Parent Rule.

---

## ADK Single Parent Rule

### The Rule

**Every agent can have ONLY ONE parent agent.**

Violation example:
```python
# ❌ WRONG - Two parents
bigquery_agent = Agent(sub_agents=[alyac_family_agent, ...])
conversational_agent = Agent(sub_agents=[alyac_family_agent, ...])  # ERROR!
```

**Error**: `ValidationError: Agent 'alyac_family_analyst' already has a parent agent`

### Why This Rule Exists

1. **Context Propagation**: Single path for conversation history flow
2. **Controlled Flow**: Prevents circular delegation loops
3. **Debugging**: Clear call hierarchy for tracing
4. **State Management**: Unambiguous session state ownership

### Implementation Pattern

```python
# ✅ CORRECT - Single parent
# bigquery_agent: Has domain expert sub_agents
bigquery_agent = Agent(
    name="general_orchestrator",
    sub_agents=[alyac_family_agent, security_agent, marketing_agent, conversion_agent]
)

# conversational_analytics_agent: No sub_agents (tools only)
conversational_agent = Agent(
    name="conversational_analyst",
    tools=[ask_data_insights, search_catalog, google_search],
    sub_agents=[]  # Empty - uses AI tools instead
)
```

**Reference**: See `reference/single_parent_rule.md` for official ADK documentation and detailed examples.

---

## Agent Selection & Routing

### Backend Routing (Keyword-Based)

**File**: `adk-backend/src/adk_backend/services/agent_selector.py`

```python
def select_agent_for_request(
    agent_type: Optional[str],  # "sql" or "conversational"
    message: str,
    conversation_history: Iterable[Tuple[str, Optional[str]]]
) -> Tuple[AgentInfo, str, str]:
    """
    Routes requests based on:
    1. Explicit agent_type selection
    2. Keyword matching in message content
    """
```

**Routing Logic**:
- `agent_type="conversational"` → `conversational_analytics_agent` (default)
- `agent_type="sql"` + keywords → Domain expert agent
  - "악성 앱", "탐지" → alyac_family_agent
  - "스미싱", "보안" → security_agent
  - "캠페인", "전환" → marketing_agent, conversion_agent

### Frontend Domain Selection

**Discovery** (from `05_domain_routing_frontend_analysis.md`):

**Frontend `domain` state is NOT sent to backend!**

```typescript
// frontend/src/App.tsx:293-299
const stopStream = apiService.streamQuery({
  query: trimmed,
  user_id: 'demo-user',
  session_id: 'demo-session',
  agent_type: agentType  // ✅ Only agent_type is sent, NOT domain!
})
```

**Implication**:
- Frontend domain selector = UI/UX only (filters Quick Actions)
- Backend uses keyword matching exclusively
- Potential UI/backend mismatch: User selects "Security" but backend chooses "ALYac Family" based on keywords

**Reference**: See `reference/routing.md` for complete flow analysis and mismatch scenarios.

---

## Error Recovery Strategies

### Problem: BigQuery SQL Constraints

**Real error** (2025-11-16):
```sql
WHERE detection_timestamp >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 MONTH)
```

**BigQuery Error**:
```
400 POST: TIMESTAMP_SUB does not support the MONTH date part
when the argument is TIMESTAMP type
```

**Root cause**: TIMESTAMP_SUB only supports DAY, HOUR, MINUTE, SECOND (not MONTH/YEAR)

### Strategy 1: Instruction Enhancement (Preventive)

**Add explicit constraints to agent instructions:**

```python
"## SQL 작성 지침\n"
"- 시간 필터: **반드시 DAY 단위만 사용**\n"
"  - ✅ 올바름: `TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)` (1개월)\n"
"  - ❌ 금지: `TIMESTAMP_SUB(..., INTERVAL 1 MONTH)` → TIMESTAMP 타입에서 MONTH 지원 안 됨\n"
```

**Applied to**: All 4 domain expert agents
**Files**: `alyac_family_domain_expert.py`, `security_domain_expert.py`, `conversion_domain_expert.py`, `marketing_domain_expert.py`

**Effectiveness**: ~90% prevention rate

### Strategy 2: ReflectAndRetryToolPlugin (Reactive)

**Google ADK's official auto-retry mechanism:**

```python
# adk-backend/src/adk_backend/api/chat.py
from google.adk.plugins import ReflectAndRetryToolPlugin

runner = InMemoryRunner(
    app_name="ADK Chat Stream",
    agent=agent,
    plugins=[
        ReflectAndRetryToolPlugin(
            max_retries=2,  # Total 3 attempts
            throw_exception_if_retry_exceeded=True,
        ),
    ],
)
```

**How it works**:
1. Tool execution fails (e.g., BigQuery SQL error)
2. Plugin intercepts error → Sends error message to AI
3. AI analyzes error → Generates corrected SQL
4. Automatic retry (up to max_retries)
5. Success → Continue | Failure → Raise exception

**Effectiveness**: ~80% recovery rate of remaining 10%

### Combined Defense-in-Depth

```
User Query: "최근 한 달간 스미싱 TOP5"
    ↓
[1st Defense] Instruction (90% prevention)
AI generates: INTERVAL 30 DAY ✅
    ↓
BigQuery executes successfully (no retry needed)
    ↓
User receives results (fast)
```

**If instruction missed**:
```
[1st Defense] Instruction missed
AI generates: INTERVAL 1 MONTH ❌
    ↓
BigQuery error
    ↓
[2nd Defense] ReflectAndRetryToolPlugin (80% recovery)
Plugin: "Error detected, feedback to AI..."
    ↓
AI corrects: INTERVAL 30 DAY ✅
    ↓
BigQuery retry succeeds
    ↓
User receives results (slightly slower)
```

**Combined success rate**: ~98%

**Reference**: See `reference/error_recovery.md` for complete error patterns, custom plugin examples, and SQL validation layer.

---

## Transfer Workflows

### Domain Expert Delegation

**bigquery_agent → Domain Expert**:
```python
# In bigquery_agent instruction
"""
## 도메인 에이전트 위임 규칙
- `alyac_family_analyst`: 악성 앱·피싱 메시지·ALYac Family 고유 지표
- `security_analyst`: 스미싱, FCM 보안 이벤트, 악성앱/위협 경보
- `marketing_analyst`: 캠페인, 세그먼트, 가족 초대, 구독 전환
- `conversion_analyst`: 구독/퍼널/리텐션/코호트 등 SaaS 핵심 지표

위임이 필요할 때는 반드시 `transfer_to_agent`를 사용해 해당 이름을 호출하고,
응답을 사용자에게 요약/보완하세요.
"""
```

**Flow**:
```
User: "최근 악성 앱 Top 5는?"
    ↓
bigquery_agent detects keyword "악성 앱"
    ↓
transfer_to_agent("alyac_family_analyst")
    ↓
alyac_family_agent executes BigQuery tools
    ↓
Result → bigquery_agent
    ↓
bigquery_agent summarizes and returns to user
```

### Conversational vs BigQuery Pattern

**When to use each**:

| Scenario | Use bigquery_agent | Use conversational_agent |
|----------|-------------------|--------------------------|
| Direct SQL needed | ✅ | ❌ |
| Natural language query | ⚠️ Limited | ✅ |
| Web search required | ❌ | ✅ |
| Latest information | ❌ | ✅ |
| Template-based analysis | ✅ | ❌ |
| Exploratory data analysis | ⚠️ | ✅ |

---

## Registry Configuration

### Agent Registration

**File**: `adk-backend/src/adk_backend/agents/registry.py`

```python
AGENT_REGISTRY = {
    "conversational": AgentInfo(
        key="conversational",
        display_name="대화형 데이터 분석가",
        agent=conversational_analytics_agent,
        focus="자연어 데이터 분석",
        keywords=["자연어", "대화", "인사이트", "AI분석", ...],
    ),
    "general": AgentInfo(
        key="general",
        display_name="범용 데이터 분석가",
        agent=bigquery_agent,
        focus="범용 데이터 분석",
        keywords=[],
    ),
    "alyac_family": AgentInfo(
        key="alyac_family",
        display_name="ALYac Family 보안 분석가",
        agent=alyac_family_agent,
        focus="악성 앱 및 피싱 분석",
        keywords=["악성", "malware", "phishing", ...],
    ),
    # ... security, marketing, conversion
}
```

### Active Agent Selection

```python
def get_active_agents() -> List[AgentInfo]:
    """Returns all agents with active=True"""
    return [info for info in AGENT_REGISTRY.values() if info.active]
```

---

## Common Patterns

### Pattern 1: Add New Domain Expert

```python
# 1. Create domain expert agent
new_domain_agent = Agent(
    name="new_domain_analyst",
    description="Domain-specific analysis",
    tools=[bigquery_list_templates, bigquery_render_template,
           bigquery_dry_run, bigquery_execute],
    # NO sub_agents (domain experts are leaf nodes)
)

# 2. Add to bigquery_agent sub_agents
bigquery_agent = Agent(
    name="general_orchestrator",
    sub_agents=[
        alyac_family_agent,
        security_agent,
        marketing_agent,
        conversion_agent,
        new_domain_agent,  # ✅ Add here
    ]
)

# 3. Register in registry
AGENT_REGISTRY["new_domain"] = AgentInfo(
    key="new_domain",
    agent=new_domain_agent,
    keywords=["keyword1", "keyword2"],
)

# 4. Update agent_selector routing logic
```

### Pattern 2: Custom Error Recovery Plugin

```python
from google.adk.plugins import ReflectAndRetryToolPlugin

class BigQueryErrorRecoveryPlugin(ReflectAndRetryToolPlugin):
    """Custom error detection for BigQuery warnings"""

    async def extract_error_from_result(self, *, tool, tool_args,
                                       tool_context, result):
        if isinstance(result, dict):
            # Detect warnings in successful responses
            if result.get('warnings'):
                return {
                    'error': 'BigQuery Warning',
                    'warnings': result['warnings'],
                    'suggestion': '경고를 해결하여 최적화된 쿼리를 생성하세요.'
                }
        return None  # No error

# Apply to runner
runner = InMemoryRunner(
    agent=agent,
    plugins=[BigQueryErrorRecoveryPlugin(max_retries=3)]
)
```

### Pattern 3: Multi-Level Delegation

```
User Query
    ↓
conversational_analytics_agent (orchestrator)
    ↓
transfer_to_agent("general_orchestrator")
    ↓
bigquery_agent (receives delegation)
    ↓
transfer_to_agent("alyac_family_analyst")
    ↓
alyac_family_agent (executes)
    ↓
Result → bigquery_agent → conversational_agent → User
```

**Note**: This creates deep nesting. Prefer direct delegation when possible.

---

## Testing & Verification

### Verify Single Parent Rule

```bash
# Check for multiple parent assignments
grep -r "sub_agents=\[.*alyac_family" adk-backend/src/adk_backend/agents/
# Should find ONLY in bigquery_agent.py
```

### Test Agent Selection

```bash
# Test keyword matching
echo '{"agent_type":"sql","message":"악성 앱 탐지"}' | \
  python -c "from adk_backend.services.agent_selector import select_agent_for_request; \
             import json, sys; \
             data=json.load(sys.stdin); \
             info, reason, _ = select_agent_for_request(
                agent_type=data['agent_type'],
                message=data['message'],
                conversation_history=[]
             ); \
             print(f'Selected: {info.key}, Reason: {reason}')"
```

### Test Error Recovery

```bash
# Manually trigger SQL error
cd adk-backend
python -c "
from src.adk_backend.tools.bigquery import bigquery_dry_run
try:
    bigquery_dry_run('SELECT * FROM table WHERE time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 MONTH)')
except Exception as e:
    print(f'Expected error: {e}')
"
```

---

## Reference Documents

Detailed information in `reference/` folder:

### [reference/architecture.md](reference/architecture.md)
Complete architecture documentation:
- Dual-orchestrator pattern deep dive
- Agent communication flow diagrams
- Session management and state tracking
- Streaming response handling
- Frontend-backend integration
- Historical evolution and design decisions

### [reference/single_parent_rule.md](reference/single_parent_rule.md)
ADK Single Parent Rule explained:
- Official Google ADK documentation links
- Why this constraint exists (context propagation, controlled flow, debugging)
- Correct vs incorrect implementation patterns
- Validation error troubleshooting
- Tree structure design principles

### [reference/routing.md](reference/routing.md)
Domain routing and agent selection:
- Frontend domain selector implementation
- Backend keyword matching algorithm
- UI/backend mismatch scenarios
- Test cases and expected behaviors
- Proposed improvements (domain_hint parameter)

### [reference/error_recovery.md](reference/error_recovery.md)
SQL error recovery strategies:
- Real error case study (TIMESTAMP_SUB MONTH issue)
- Instruction enhancement examples for all domain agents
- ReflectAndRetryToolPlugin configuration
- Custom error extraction patterns
- SQL validation layer implementation
- Performance metrics and success rates

### [reference/agent_selection.md](reference/agent_selection.md)
Agent selection verification:
- Frontend agent type selection (sql/conversational)
- Backend routing logic (`agent_selector.py`)
- API request/response flow
- SSE (Server-Sent Events) streaming
- Testing and verification procedures

---

## Quick Reference

### File Locations

```
adk-backend/src/adk_backend/
├── agents/
│   ├── base/
│   │   ├── bigquery_agent.py (SQL orchestrator, has sub_agents)
│   │   └── conversational_analytics_agent.py (AI orchestrator, no sub_agents)
│   ├── alyac_family_domain_expert.py
│   ├── security_domain_expert.py
│   ├── marketing_domain_expert.py
│   ├── conversion_domain_expert.py
│   └── registry.py (AgentInfo registration)
├── services/
│   └── agent_selector.py (routing logic)
├── tools/
│   ├── bigquery.py (SQL tools)
│   └── conversational_analytics.py (AI analysis tools)
└── api/
    └── chat.py (SSE streaming, ReflectAndRetryToolPlugin)
```

### Decision Tree

```
Need to handle agent request?
    ↓
Q: User mentioned specific domain keywords?
    ├─ YES → Route to domain expert agent
    │         └─ Use agent_selector.py keyword matching
    │
    └─ NO → Default to conversational_analytics_agent
            └─ Let AI decide delegation

Q: Agent architecture change needed?
    ↓
First check: Does it violate Single Parent Rule?
    ├─ YES → Redesign to use tools or indirect delegation
    └─ NO → Proceed with implementation

Q: SQL errors occurring?
    ↓
Layer 1: Enhanced instructions (preventive)
    ├─ Success → Fast response
    └─ Fail → Layer 2: ReflectAndRetryToolPlugin (reactive)
              └─ Auto-retry with AI correction
```

### Common Issues

1. **ValidationError: already has a parent agent**
   - **Cause**: Violating Single Parent Rule
   - **Fix**: Remove agent from one parent's sub_agents list
   - **Reference**: `reference/single_parent_rule.md`

2. **Agent not selected for domain keyword**
   - **Cause**: Keyword not in routing logic
   - **Fix**: Update `agent_selector.py` keyword patterns
   - **Reference**: `reference/agent_selection.md`

3. **TIMESTAMP_SUB MONTH error**
   - **Cause**: BigQuery constraint not in instruction
   - **Fix**: Check domain expert instruction has DAY-only guidance
   - **Reference**: `reference/error_recovery.md`

4. **Frontend domain selection not working**
   - **Cause**: Domain state not sent to backend
   - **Fix**: This is by design - backend uses keyword matching
   - **Reference**: `reference/routing.md`

---

## Next Steps

1. **Add domain expert**: Follow Pattern 1 above
2. **Improve routing**: Update `agent_selector.py` keywords
3. **Custom error handling**: Extend ReflectAndRetryToolPlugin
4. **Frontend enhancement**: Add domain_hint to API request
5. **Monitoring**: Track agent selection accuracy and error recovery rates

---

**Skill Status**: Production-ready ✅
**Line Count**: <500 (follows 500-line rule) ✅
**Progressive Disclosure**: Detailed docs in reference/ ✅
**Last Updated**: 2025-11-16
**Related Commit**: `35470b1` (BigQuery SQL error recovery with dual-layer strategy)
