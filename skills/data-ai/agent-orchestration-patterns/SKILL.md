---
name: agent-orchestration-patterns
description: Automatically applies when designing multi-agent systems. Ensures proper tool schema design with Pydantic, agent state management, error handling for tool execution, and orchestration patterns.
category: ai-llm
---

# Agent Orchestration Patterns

When building multi-agent systems and tool-calling workflows, follow these patterns for reliable, maintainable orchestration.

**Trigger Keywords**: agent, multi-agent, tool calling, orchestration, subagent, tool schema, function calling, agent state, agent routing, agent graph, LangChain, LlamaIndex, Anthropic tools

**Agent Integration**: Used by `ml-system-architect`, `agent-orchestrator-engineer`, `llm-app-engineer`, `security-and-privacy-engineer-ml`

## ✅ Correct Pattern: Tool Schema with Pydantic

```python
from pydantic import BaseModel, Field
from typing import List, Literal, Optional
from enum import Enum


class SearchQuery(BaseModel):
    """Tool input for search."""
    query: str = Field(..., description="Search query string")
    max_results: int = Field(
        10,
        ge=1,
        le=100,
        description="Maximum number of results to return"
    )
    filter_domain: Optional[str] = Field(
        None,
        description="Optional domain to filter results (e.g., 'python.org')"
    )


class SearchResult(BaseModel):
    """Individual search result."""
    title: str
    url: str
    snippet: str
    relevance_score: float = Field(ge=0.0, le=1.0)


class SearchResponse(BaseModel):
    """Tool output for search."""
    results: List[SearchResult]
    total_found: int
    query_time_ms: float


async def search_tool(input: SearchQuery) -> SearchResponse:
    """
    Search the web and return relevant results.

    Args:
        input: Validated search parameters

    Returns:
        Search results with metadata

    Example:
        >>> result = await search_tool(SearchQuery(
        ...     query="Python async patterns",
        ...     max_results=5
        ... ))
        >>> print(result.results[0].title)
    """
    # Implementation
    results = await perform_search(
        query=input.query,
        limit=input.max_results,
        domain_filter=input.filter_domain
    )

    return SearchResponse(
        results=results,
        total_found=len(results),
        query_time_ms=123.45
    )


# Convert to Claude tool schema
def tool_to_anthropic_schema(func, input_model: type[BaseModel]) -> dict:
    """Convert Pydantic model to Anthropic tool schema."""
    return {
        "name": func.__name__.replace("_tool", ""),
        "description": func.__doc__.strip().split("\n")[0],
        "input_schema": input_model.model_json_schema()
    }


# Register tool
SEARCH_TOOL = tool_to_anthropic_schema(search_tool, SearchQuery)
```

## Agent State Management

```python
from typing import List, Dict, Any, Optional
from datetime import datetime
from pydantic import BaseModel, Field
import uuid


class Message(BaseModel):
    """A single message in conversation."""
    role: Literal["user", "assistant", "system"]
    content: str
    timestamp: datetime = Field(default_factory=datetime.utcnow)
    metadata: Dict[str, Any] = Field(default_factory=dict)


class ToolCall(BaseModel):
    """Record of a tool execution."""
    tool_name: str
    input: Dict[str, Any]
    output: Any
    duration_ms: float
    success: bool
    error: Optional[str] = None
    timestamp: datetime = Field(default_factory=datetime.utcnow)


class AgentState(BaseModel):
    """State for an agent conversation."""
    session_id: str = Field(default_factory=lambda: str(uuid.uuid4()))
    messages: List[Message] = Field(default_factory=list)
    tool_calls: List[ToolCall] = Field(default_factory=list)
    metadata: Dict[str, Any] = Field(default_factory=dict)
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)

    def add_message(self, role: str, content: str, **metadata):
        """Add message to conversation history."""
        self.messages.append(
            Message(role=role, content=content, metadata=metadata)
        )
        self.updated_at = datetime.utcnow()

    def add_tool_call(self, tool_call: ToolCall):
        """Record tool execution."""
        self.tool_calls.append(tool_call)
        self.updated_at = datetime.utcnow()

    def get_conversation_history(self) -> List[Dict[str, str]]:
        """Get messages in format for LLM API."""
        return [
            {"role": msg.role, "content": msg.content}
            for msg in self.messages
            if msg.role != "system"
        ]


class AgentStateManager:
    """Manage agent states with persistence."""

    def __init__(self):
        self._states: Dict[str, AgentState] = {}

    async def get_or_create(self, session_id: str | None = None) -> AgentState:
        """Get existing state or create new one."""
        if session_id and session_id in self._states:
            return self._states[session_id]

        state = AgentState(session_id=session_id or str(uuid.uuid4()))
        self._states[state.session_id] = state
        return state

    async def save(self, state: AgentState):
        """Persist agent state."""
        self._states[state.session_id] = state
        # Could also save to database/redis here

    async def load(self, session_id: str) -> Optional[AgentState]:
        """Load agent state from storage."""
        return self._states.get(session_id)
```

## Tool Execution with Error Handling

```python
from typing import Callable, Any, Type
import asyncio
import logging
from datetime import datetime

logger = logging.getLogger(__name__)


class ToolError(Exception):
    """Base tool execution error."""
    pass


class ToolTimeoutError(ToolError):
    """Tool execution timeout."""
    pass


class ToolValidationError(ToolError):
    """Tool input validation error."""
    pass


class ToolExecutor:
    """Execute tools with validation and error handling."""

    def __init__(self, timeout: float = 30.0):
        self.timeout = timeout
        self.tools: Dict[str, tuple[Callable, Type[BaseModel]]] = {}

    def register_tool(
        self,
        name: str,
        func: Callable,
        input_model: Type[BaseModel]
    ):
        """Register a tool with its input schema."""
        self.tools[name] = (func, input_model)

    async def execute(
        self,
        tool_name: str,
        tool_input: Dict[str, Any]
    ) -> ToolCall:
        """
        Execute tool with validation and error handling.

        Args:
            tool_name: Name of tool to execute
            tool_input: Raw input dict from LLM

        Returns:
            ToolCall record with result or error

        Raises:
            ToolError: If tool execution fails unrecoverably
        """
        if tool_name not in self.tools:
            error_msg = f"Unknown tool: {tool_name}"
            logger.error(error_msg)
            return ToolCall(
                tool_name=tool_name,
                input=tool_input,
                output=None,
                duration_ms=0.0,
                success=False,
                error=error_msg
            )

        func, input_model = self.tools[tool_name]
        start_time = datetime.utcnow()

        try:
            # Validate input
            try:
                validated_input = input_model(**tool_input)
            except Exception as e:
                raise ToolValidationError(
                    f"Invalid input for {tool_name}: {str(e)}"
                ) from e

            # Execute with timeout
            try:
                output = await asyncio.wait_for(
                    func(validated_input),
                    timeout=self.timeout
                )
            except asyncio.TimeoutError:
                raise ToolTimeoutError(
                    f"Tool {tool_name} exceeded timeout of {self.timeout}s"
                )

            duration_ms = (datetime.utcnow() - start_time).total_seconds() * 1000

            logger.info(
                f"Tool executed successfully",
                extra={
                    "tool_name": tool_name,
                    "duration_ms": duration_ms
                }
            )

            return ToolCall(
                tool_name=tool_name,
                input=tool_input,
                output=output,
                duration_ms=duration_ms,
                success=True
            )

        except ToolError as e:
            duration_ms = (datetime.utcnow() - start_time).total_seconds() * 1000

            logger.error(
                f"Tool execution failed",
                extra={
                    "tool_name": tool_name,
                    "error": str(e),
                    "duration_ms": duration_ms
                }
            )

            return ToolCall(
                tool_name=tool_name,
                input=tool_input,
                output=None,
                duration_ms=duration_ms,
                success=False,
                error=str(e)
            )
```

## Agent Orchestration Patterns

### Pattern 1: Sequential Agent Chain

```python
from typing import List


class SequentialOrchestrator:
    """Execute agents in sequence, passing output to next."""

    def __init__(self, agents: List[Callable]):
        self.agents = agents

    async def run(self, initial_input: str) -> str:
        """
        Run agents sequentially.

        Args:
            initial_input: Input for first agent

        Returns:
            Output from final agent
        """
        current_input = initial_input

        for i, agent in enumerate(self.agents):
            logger.info(f"Running agent {i + 1}/{len(self.agents)}")
            current_input = await agent(current_input)

        return current_input


# Example usage
async def research_agent(query: str) -> str:
    """Research a topic."""
    # Search and gather information
    return "research results..."


async def synthesis_agent(research: str) -> str:
    """Synthesize research into summary."""
    # Analyze and synthesize
    return "synthesized summary..."


async def writer_agent(summary: str) -> str:
    """Write final article."""
    # Generate polished content
    return "final article..."


# Chain agents
orchestrator = SequentialOrchestrator([
    research_agent,
    synthesis_agent,
    writer_agent
])

result = await orchestrator.run("Tell me about Python async patterns")
```

### Pattern 2: Parallel Agent Execution

```python
import asyncio


class ParallelOrchestrator:
    """Execute multiple agents concurrently."""

    def __init__(self, agents: List[Callable]):
        self.agents = agents

    async def run(self, input: str) -> List[Any]:
        """
        Run all agents in parallel with same input.

        Args:
            input: Input for all agents

        Returns:
            List of outputs from each agent
        """
        tasks = [agent(input) for agent in self.agents]
        results = await asyncio.gather(*tasks, return_exceptions=True)

        # Handle any failures
        for i, result in enumerate(results):
            if isinstance(result, Exception):
                logger.error(f"Agent {i} failed: {result}")

        return results


# Example: Multiple specialized agents
async def technical_reviewer(code: str) -> str:
    """Review code for technical issues."""
    return "technical review..."


async def security_reviewer(code: str) -> str:
    """Review code for security issues."""
    return "security review..."


async def performance_reviewer(code: str) -> str:
    """Review code for performance issues."""
    return "performance review..."


# Run reviewers in parallel
orchestrator = ParallelOrchestrator([
    technical_reviewer,
    security_reviewer,
    performance_reviewer
])

reviews = await orchestrator.run(code_to_review)
```

### Pattern 3: Router-Based Orchestration

```python
from enum import Enum


class AgentType(str, Enum):
    """Available agent types."""
    TECHNICAL = "technical"
    CREATIVE = "creative"
    ANALYTICAL = "analytical"


class RouterOrchestrator:
    """Route requests to appropriate specialized agent."""

    def __init__(self):
        self.agents: Dict[AgentType, Callable] = {}

    def register(self, agent_type: AgentType, agent: Callable):
        """Register an agent."""
        self.agents[agent_type] = agent

    async def classify_request(self, request: str) -> AgentType:
        """
        Classify request to determine which agent to use.

        Args:
            request: User request

        Returns:
            Agent type to handle request
        """
        # Use LLM to classify
        prompt = f"""Classify this request into one of:
        - technical: Code, debugging, technical implementation
        - creative: Writing, brainstorming, creative content
        - analytical: Data analysis, research, evaluation

        Request: {request}

        Return only the category name."""

        category = await llm_classify(prompt)
        return AgentType(category.lower().strip())

    async def route(self, request: str) -> str:
        """
        Route request to appropriate agent.

        Args:
            request: User request

        Returns:
            Response from selected agent
        """
        agent_type = await self.classify_request(request)
        agent = self.agents.get(agent_type)

        if not agent:
            raise ValueError(f"No agent registered for type: {agent_type}")

        logger.info(f"Routing to {agent_type} agent")
        return await agent(request)
```

### Pattern 4: Hierarchical Agent System

```python
class SupervisorAgent:
    """Supervisor that delegates to specialized sub-agents."""

    def __init__(self):
        self.sub_agents: Dict[str, Callable] = {}
        self.state_manager = AgentStateManager()

    async def delegate(
        self,
        task: str,
        state: AgentState
    ) -> str:
        """
        Decompose task and delegate to sub-agents.

        Args:
            task: High-level task description
            state: Current conversation state

        Returns:
            Final result after delegation
        """
        # Plan decomposition using LLM
        plan = await self.plan_task(task, state)

        results = []
        for subtask in plan.subtasks:
            # Find appropriate sub-agent
            agent = self.find_agent_for_task(subtask)

            # Execute subtask
            result = await agent(subtask.description, state)
            results.append(result)

            # Update state
            state.add_message("assistant", f"Subtask result: {result}")

        # Synthesize final result
        return await self.synthesize_results(task, results, state)

    async def plan_task(self, task: str, state: AgentState) -> TaskPlan:
        """Decompose task into subtasks."""
        # Use LLM to plan
        ...

    def find_agent_for_task(self, subtask: SubTask) -> Callable:
        """Select appropriate sub-agent for subtask."""
        # Match subtask to agent capabilities
        ...
```

## ❌ Anti-Patterns

```python
# ❌ No input validation
async def tool(input: dict) -> dict:  # Raw dict!
    return await do_something(input["query"])

# ✅ Better: Use Pydantic for validation
async def tool(input: SearchQuery) -> SearchResponse:
    return await do_something(input.query)


# ❌ No error handling in tool execution
async def execute_tool(name: str, input: dict):
    func = tools[name]
    return await func(input)  # Can fail!

# ✅ Better: Comprehensive error handling
async def execute_tool(name: str, input: dict) -> ToolCall:
    try:
        validated = InputModel(**input)
        result = await func(validated)
        return ToolCall(success=True, output=result)
    except ValidationError as e:
        return ToolCall(success=False, error=str(e))


# ❌ No timeout on tool execution
result = await long_running_tool(input)  # Could hang forever!

# ✅ Better: Add timeout
result = await asyncio.wait_for(
    long_running_tool(input),
    timeout=30.0
)


# ❌ Stateless conversations
async def handle_request(prompt: str) -> str:
    return await agent.run(prompt)  # No history!

# ✅ Better: Maintain conversation state
async def handle_request(prompt: str, session_id: str) -> str:
    state = await state_manager.load(session_id)
    state.add_message("user", prompt)
    response = await agent.run(state.get_conversation_history())
    state.add_message("assistant", response)
    await state_manager.save(state)
    return response


# ❌ No logging of tool calls
result = await tool(input)

# ✅ Better: Log all tool executions
logger.info("Executing tool", extra={
    "tool_name": tool.__name__,
    "input": input.model_dump()
})
result = await tool(input)
logger.info("Tool completed", extra={
    "duration_ms": duration,
    "success": True
})
```

## Best Practices Checklist

- ✅ Define tool schemas with Pydantic models
- ✅ Validate all tool inputs before execution
- ✅ Set timeouts on tool execution
- ✅ Handle tool errors gracefully (don't crash)
- ✅ Maintain conversation state across turns
- ✅ Log all tool executions with inputs and outputs
- ✅ Use typed responses from tools
- ✅ Implement retry logic for transient failures
- ✅ Redact sensitive data in tool logs
- ✅ Use async/await throughout agent code
- ✅ Structure agent output as Pydantic models
- ✅ Track agent performance metrics

## Auto-Apply

When building multi-agent systems:
1. Define tool schemas with Pydantic
2. Implement ToolExecutor for safe execution
3. Maintain AgentState for conversations
4. Add comprehensive error handling
5. Log all agent and tool interactions
6. Use appropriate orchestration pattern (sequential, parallel, router, hierarchical)
7. Set timeouts on all agent operations

## Related Skills

- `pydantic-models` - For tool schema definition
- `async-await-checker` - For async agent patterns
- `llm-app-architecture` - For LLM integration
- `structured-errors` - For error handling
- `observability-logging` - For agent logging
- `type-safety` - For type-safe tool definitions
