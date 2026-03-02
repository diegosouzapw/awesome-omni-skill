---
name: argentic-framework-development
description: Expert knowledge for building AI agents with Argentic - a Python microframework for async MQTT-based agents with multi-LLM support, custom tools, and multi-agent orchestration
---

# Argentic Framework Development Skill

You are an expert in building AI agents using the Argentic framework. This skill provides comprehensive knowledge about Argentic's architecture, patterns, and best practices.

## Framework Overview

Argentic is a Python 3.11+ microframework for building local AI agents with async MQTT messaging.

**Core Architecture:**
- Fully async/await based
- MQTT-driven messaging between components
- Multi-LLM support (Google Gemini, Ollama, Llama.cpp)
- Dynamic tool registration via messaging protocol
- Single-agent and multi-agent (Supervisor) patterns

**Key Components:**
- `Agent` - Main AI agent with LLM integration
- `Messager` - MQTT messaging layer
- `ToolManager` - Tool registry and execution
- `Supervisor` - Multi-agent coordinator
- `BaseTool` - Base class for custom tools

## Installation

```bash
# From PyPI
pip install argentic

# From source
git clone https://github.com/angkira/argentic.git
cd argentic
pip install -e .
```

**Prerequisites:**
- Python 3.11+
- MQTT broker (mosquitto): `docker run -d -p 1883:1883 eclipse-mosquitto:2.0`
- API keys for LLM providers (e.g., `GOOGLE_GEMINI_API_KEY` in `.env`)

## Pattern 1: Single Agent Application

```python
import asyncio
from argentic import Agent, Messager, LLMFactory
from argentic.core.tools import ToolManager
import yaml

async def main():
    # Load configuration
    with open("config.yaml") as f:
        config = yaml.safe_load(f)
    
    # Initialize LLM
    llm = LLMFactory.create_from_config(config["llm"])
    
    # Setup MQTT messaging
    messager = Messager(
        broker_address=config["messaging"]["broker_address"],
        port=config["messaging"]["port"]
    )
    await messager.connect()
    
    # Initialize ToolManager
    tool_manager = ToolManager(messager)
    await tool_manager.async_init()
    
    # Create Agent
    agent = Agent(
        llm=llm,
        messager=messager,
        tool_manager=tool_manager,
        role="assistant",
        system_prompt="You are a helpful AI assistant.",
        enable_dialogue_logging=True  # For debugging
    )
    await agent.async_init()
    
    # Use the agent
    response = await agent.query("What is the capital of France?")
    print(response)
    
    # Cleanup
    await messager.disconnect()

if __name__ == "__main__":
    asyncio.run(main())
```

**Key Points:**
- Always `await messager.connect()` before using components
- Initialize `ToolManager` before creating agents
- Always `await agent.async_init()` after creation
- Clean up with `await messager.disconnect()` in finally block

## Pattern 2: Custom Tool Development

```python
import json
from typing import Any
from pydantic import BaseModel, Field
from argentic.core.tools.tool_base import BaseTool
from argentic.core.messager.messager import Messager

# Step 1: Define input schema with Pydantic
class MyToolInput(BaseModel):
    param1: str = Field(description="First parameter description")
    param2: int = Field(default=10, ge=1, le=100, description="Integer between 1-100")
    optional_param: str = Field(default="", description="Optional parameter")

# Step 2: Implement tool class
class MyCustomTool(BaseTool):
    def __init__(self, messager: Messager):
        super().__init__(
            name="my_custom_tool",
            manual="Tool description that LLM will see. Explain what it does and how to use it.",
            api=json.dumps(MyToolInput.model_json_schema()),
            argument_schema=MyToolInput,
            messager=messager,
        )
    
    async def _execute(self, **kwargs) -> Any:
        """
        Implement tool logic here.
        Arguments are validated by Pydantic before this is called.
        """
        param1 = kwargs["param1"]
        param2 = kwargs.get("param2", 10)
        optional = kwargs.get("optional_param", "")
        
        # Your implementation here
        try:
            result = f"Processed {param1} with {param2}"
            return result
        except Exception as e:
            raise RuntimeError(f"Tool execution failed: {e}")

# Step 3: Register tool
async def setup_tool(messager):
    tool = MyCustomTool(messager)
    await tool.register(
        registration_topic="agent/tools/register",
        status_topic="agent/status/info",
        call_topic_base="agent/tools/call",
        response_topic_base="agent/tools/response",
    )
    return tool

# Step 4: Use with agent
async def main():
    messager = Messager(broker_address="localhost", port=1883)
    await messager.connect()
    
    tool_manager = ToolManager(messager)
    await tool_manager.async_init()
    
    # Register tool BEFORE creating agent
    tool = await setup_tool(messager)
    await asyncio.sleep(1)  # Wait for registration to complete
    
    # Create agent (will auto-discover registered tools)
    agent = Agent(llm, messager, tool_manager, role="assistant")
    await agent.async_init()
    
    # Agent can now use the tool
    response = await agent.query("Use my_custom_tool with param1='test'")
    print(response)
    
    await messager.disconnect()
```

**Tool Development Best Practices:**
- Use Pydantic for automatic validation
- Provide clear `manual` description for LLM
- Include field descriptions in Pydantic schema
- Handle errors with specific exceptions
- Test tools independently before agent integration

## Pattern 3: Multi-Agent System

```python
from argentic.core.graph.supervisor import Supervisor
from argentic import Agent, Messager, LLMFactory
from argentic.core.tools import ToolManager
import asyncio

async def main():
    # Setup (same as single agent)
    llm = LLMFactory.create_from_config(config["llm"])
    messager = Messager(broker_address="localhost", port=1883)
    await messager.connect()
    
    # IMPORTANT: Use ONE shared ToolManager for all agents
    tool_manager = ToolManager(messager)
    await tool_manager.async_init()
    
    # Create specialized agents with separate MQTT topics
    researcher = Agent(
        llm=llm,
        messager=messager,
        tool_manager=tool_manager,  # Shared!
        role="researcher",
        description="Research and information gathering specialist",
        system_prompt="You are a researcher. Find and synthesize information.",
        expected_output_format="text",
        # Agent-specific topics to avoid conflicts
        register_topic="agent/researcher/tools/register",
        tool_call_topic_base="agent/researcher/tools/call",
        tool_response_topic_base="agent/researcher/tools/response",
        status_topic="agent/researcher/status/info",
        graph_id="multi_agent_system",
    )
    await researcher.async_init()
    
    coder = Agent(
        llm=llm,
        messager=messager,
        tool_manager=tool_manager,  # Same shared instance!
        role="coder",
        description="Code writing and debugging specialist",
        system_prompt="You are a coder. Write clean, efficient code.",
        expected_output_format="code",
        register_topic="agent/coder/tools/register",
        tool_call_topic_base="agent/coder/tools/call",
        tool_response_topic_base="agent/coder/tools/response",
        status_topic="agent/coder/status/info",
        graph_id="multi_agent_system",
    )
    await coder.async_init()
    
    # Create supervisor
    supervisor = Supervisor(
        llm=llm,
        messager=messager,
        role="supervisor",
        system_prompt="You are a supervisor. Analyze tasks and route them to appropriate agents.",
        enable_dialogue_logging=True,
    )
    supervisor.add_agent(researcher)
    supervisor.add_agent(coder)
    await supervisor.async_init()
    
    # Execute workflow with callback
    workflow_complete = asyncio.Event()
    result_data = {}
    
    def completion_callback(task_id, success, result="", error=""):
        result_data["success"] = success
        result_data["result"] = result
        result_data["error"] = error
        workflow_complete.set()
    
    task_id = await supervisor.start_task(
        "Research async patterns in Python and write example code",
        completion_callback
    )
    
    # Wait for completion
    await asyncio.wait_for(workflow_complete.wait(), timeout=180)
    
    if result_data["success"]:
        print(f"Success: {result_data['result']}")
    else:
        print(f"Failed: {result_data['error']}")
    
    await messager.disconnect()

if __name__ == "__main__":
    asyncio.run(main())
```

**Multi-Agent Critical Points:**
- Use ONE shared `ToolManager` instance for all agents
- Give each agent separate MQTT topic namespaces
- Set unique `role` for each agent
- Use clear `description` for supervisor routing
- Set same `graph_id` for all agents in the system
- Enable logging for debugging: `enable_dialogue_logging=True`

## Configuration

### config.yaml Structure

```yaml
llm:
  # Provider: google_gemini, ollama, llama_cpp_server, llama_cpp_cli
  provider: google_gemini
  
  # Google Gemini
  google_gemini_model_name: gemini-2.0-flash
  google_gemini_api_key: ${GOOGLE_GEMINI_API_KEY}  # From env
  google_gemini_parameters:
    temperature: 0.7
    top_p: 0.95
    top_k: 40
    max_output_tokens: 2048
  
  # Ollama
  ollama_model_name: llama3
  ollama_base_url: http://localhost:11434
  ollama_use_chat_model: true
  ollama_parameters:
    temperature: 0.7
    num_predict: 128
  
  # Llama.cpp Server
  llama_cpp_server_host: 127.0.0.1
  llama_cpp_server_port: 5000
  llama_cpp_server_auto_start: false
  llama_cpp_server_binary: ~/llama.cpp/build/bin/llama-server

messaging:
  protocol: mqtt
  broker_address: localhost
  port: 1883
  keepalive: 60
  client_id: ""  # Auto-generated if empty

topics:
  tools:
    register: "agent/tools/register"
    call: "agent/tools/call"
    response_base: "agent/tools/response"
    status: "agent/tools/status"
  commands:
    ask_question: "agent/command/ask_question"
  responses:
    answer: "agent/response/answer"
  log: "agent/log"
```

### .env File

```bash
# LLM API Keys
GOOGLE_GEMINI_API_KEY=your_api_key_here

# MQTT (optional)
MQTT_USERNAME=username
MQTT_PASSWORD=password

# Logging
LOG_LEVEL=INFO
CONFIG_PATH=config.yaml
```

## Core API Reference

### Agent

```python
Agent(
    llm: ModelProvider,                    # Required
    messager: Messager,                    # Required
    tool_manager: ToolManager = None,      # Recommended
    role: str = "agent",
    system_prompt: str = None,
    description: str = "",
    expected_output_format: Literal["json", "text", "code"] = "json",
    enable_dialogue_logging: bool = False,
    register_topic: str = "agent/tools/register",
    tool_call_topic_base: str = "agent/tools/call",
    tool_response_topic_base: str = "agent/tools/response",
    status_topic: str = "agent/status/info",
    graph_id: str = None,
    state_mode: AgentStateMode = STATEFUL,
)

# Key Methods
await agent.async_init()                   # Initialize
response = await agent.query(question)     # Direct query
await agent.process_task(task)             # Process task message
agent.print_dialogue_summary()             # Debug info
```

### Messager

```python
Messager(
    broker_address: str = "localhost",
    port: int = 1883,
    client_id: str = "",
    username: str = None,
    password: str = None,
    keepalive: int = 60,
)

# Key Methods
await messager.connect()
await messager.disconnect()
await messager.subscribe(topic, handler, message_cls)
await messager.publish(topic, message_obj)
```

### ToolManager

```python
ToolManager(
    messager: Messager,
    register_topic: str = "agent/tools/register",
    tool_call_topic_base: str = "agent/tools/call",
    tool_response_topic_base: str = "agent/tools/response",
    status_topic: str = "agent/status/info",
    default_timeout: int = 30,
)

# Key Methods
await tool_manager.async_init()
result = await tool_manager.execute_tool(name, args, timeout=30)
tools_desc = tool_manager.get_tools_description()
```

### Supervisor

```python
Supervisor(
    llm: ModelProvider,
    messager: Messager,
    role: str = "supervisor",
    system_prompt: str = None,
    enable_dialogue_logging: bool = False,
)

# Key Methods
supervisor.add_agent(agent)
await supervisor.async_init()
task_id = await supervisor.start_task(task, callback)
```

## Important Implementation Details

### Tool Registration Flow

1. **Tool → ToolManager**: Tool publishes `RegisterToolMessage` to `agent/tools/register`
2. **ToolManager**: Generates unique `tool_id` (UUID), stores metadata
3. **ToolManager → Tool**: Publishes `ToolRegisteredMessage` to `agent/status/info`
4. **Tool**: Receives confirmation, subscribes to `agent/tools/call/{tool_id}`

### Tool Execution Flow

1. Agent calls `tool_manager.execute_tool(name, args)`
2. ToolManager publishes `TaskMessage` to `agent/tools/call/{tool_id}`
3. Tool receives message, validates args, executes `_execute()`
4. Tool publishes `TaskResultMessage` to `agent/tools/response/{tool_id}`
5. ToolManager returns result to agent

### Message Protocol

All messages inherit from `BaseMessage` (Pydantic models):

```python
from argentic.core.protocol.message import (
    AgentTaskMessage,
    AgentTaskResultMessage,
)
from argentic.core.protocol.task import (
    TaskMessage,
    TaskResultMessage,
    TaskErrorMessage,
)
from argentic.core.protocol.tool import (
    RegisterToolMessage,
    ToolRegisteredMessage,
)
```

## Best Practices

### 1. Always Use Async/Await
```python
# GOOD
await messager.connect()
result = await agent.query(question)
await messager.disconnect()

# BAD - will not work
messager.connect()  # Missing await
```

### 2. Shared ToolManager for Multi-Agent
```python
# GOOD - One shared instance
tool_manager = ToolManager(messager)
agent1 = Agent(llm, messager, tool_manager, ...)
agent2 = Agent(llm, messager, tool_manager, ...)

# BAD - Multiple instances cause conflicts
agent1 = Agent(llm, messager, ToolManager(messager), ...)
agent2 = Agent(llm, messager, ToolManager(messager), ...)
```

### 3. Proper Resource Cleanup
```python
# GOOD
try:
    await messager.connect()
    # Use agents
finally:
    await messager.disconnect()
```

### 4. Wait After Tool Registration
```python
# GOOD
await tool.register(...)
await asyncio.sleep(1)  # Give time for registration
agent = Agent(...)
```

### 5. Separate Topics for Multi-Agent
```python
# GOOD - Each agent has own namespace
researcher = Agent(
    ...,
    register_topic="agent/researcher/tools/register",
    tool_call_topic_base="agent/researcher/tools/call",
)
coder = Agent(
    ...,
    register_topic="agent/coder/tools/register",
    tool_call_topic_base="agent/coder/tools/call",
)
```

### 6. Clear System Prompts
```python
# GOOD
system_prompt = "You are a researcher. Your job is to find and synthesize information from various sources. Be thorough and cite sources."

# BAD
system_prompt = "You are helpful."
```

### 7. Enable Logging for Debug
```python
agent = Agent(
    ...,
    enable_dialogue_logging=True,  # Shows all interactions
)
# Later
agent.print_dialogue_summary()
```

## Common Patterns

### Running Argentic Components

```bash
# CLI (after installation)
argentic agent --config-path config.yaml --log-level INFO
argentic rag --config-path config.yaml
argentic cli --config-path config.yaml

# Python module
python -m argentic agent --config-path config.yaml
```

### Import Patterns

```python
# Top-level imports
from argentic import Agent, Messager, LLMFactory

# Core modules
from argentic.core.tools import ToolManager, BaseTool
from argentic.core.graph.supervisor import Supervisor
from argentic.core.protocol.message import AgentTaskMessage

# LLM providers
from argentic.core.llm.providers.google_gemini import GoogleGeminiProvider
from argentic.core.llm.providers.ollama import OllamaProvider
```

### Testing Tools Independently

```python
# Test without full agent setup
async def test_tool():
    messager = Messager(broker_address="localhost")
    await messager.connect()
    
    tool = MyTool(messager)
    
    # Direct execution (bypass MQTT)
    result = await tool._execute(param="test")
    print(result)
    
    await messager.disconnect()

asyncio.run(test_tool())
```

## Troubleshooting

### MQTT Connection Failed
**Error**: `Connection refused` or `No connection to MQTT broker`

**Solution**:
```bash
# Check if mosquitto is running
docker ps | grep mosquitto
# Or
sudo systemctl status mosquitto

# Start mosquitto
docker run -d -p 1883:1883 --name mosquitto eclipse-mosquitto:2.0
```

### Tool Not Registered
**Error**: Tool doesn't appear in agent's tool list

**Solution**:
- Add delay after registration: `await asyncio.sleep(1)`
- Verify topics match between tool and ToolManager
- Enable logging to see registration messages
- Check tool is registered BEFORE agent initialization

### LLM API Error
**Error**: `Invalid API key` or `Authentication failed`

**Solution**:
- Verify `.env` file exists with correct key
- Load environment: `from dotenv import load_dotenv; load_dotenv()`
- Check environment variable: `os.getenv("GOOGLE_GEMINI_API_KEY")`

### Tool Timeout
**Error**: `TimeoutError: Tool execution exceeded timeout`

**Solution**:
- Increase timeout: `await tool_manager.execute_tool(..., timeout=60)`
- Check tool implementation for blocking operations
- Ensure tool publishes result message

### Multi-Agent Not Routing
**Error**: Supervisor doesn't route to agents

**Solution**:
- Verify supervisor `system_prompt` includes routing instructions
- Check all agents have unique `role` values
- Ensure agent topics don't overlap
- Enable `enable_dialogue_logging=True` on supervisor

## Advanced Features

### Endless Cycle Support

For long-running agents:
```python
agent = Agent(
    ...,
    adaptive_max_iterations=True,
    max_consecutive_tool_calls=3,
    tool_call_window_size=5,
    enable_completion_analysis=True,
)
```

### State Management

```python
from argentic.core.agent.agent import AgentStateMode

# Stateful (default) - maintains conversation
agent = Agent(..., state_mode=AgentStateMode.STATEFUL)

# Stateless - each query independent
agent = Agent(..., state_mode=AgentStateMode.STATELESS)
```

## When to Use This Skill

Use this skill when:
- Building AI agent applications
- Implementing custom tools for agents
- Creating multi-agent systems
- Debugging Argentic applications
- Configuring LLM providers
- Setting up MQTT messaging
- Working with async Python patterns in Argentic

## Key Reminders

1. **Always async/await** - Everything is asynchronous
2. **One ToolManager** - Share across all agents
3. **Separate topics** - Each agent needs its own namespace
4. **Wait after registration** - Tools need time to register
5. **Clean up connections** - Always disconnect messager
6. **Enable logging** - Use `enable_dialogue_logging=True` for debug
7. **Pydantic validation** - Use for all tool inputs
8. **Clear prompts** - Make system prompts specific and detailed

## Resources

- Documentation: Check `docs/` directory or `ARGENTIC_QUICKREF.md`
- Examples: See `examples/single_agent_example.py`, `examples/multi_agent_example.py`
- Source: `src/argentic/` for implementation details

