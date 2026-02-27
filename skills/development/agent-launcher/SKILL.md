---
name: "Agent Launcher"
description: "Launches specialized Claude agents for targeted tasks. Analyzes requirements, selects appropriate agent, and executes with optimized configuration."
---

# Agent Launcher Skill

## Overview

The Agent Launcher Skill enables intelligent, purpose-driven task execution by automatically selecting and launching the most appropriate Claude agent from your framework's agent registry. Instead of manually configuring agents, simply describe what you need done and let Agent Launcher select the right agent with optimal settings.

**Use this skill when**:
- You need to execute a task that would benefit from a specialized agent
- You want automatic agent selection based on task requirements
- You need consistent, optimized agent execution across projects
- You're working with the Claude Agent Framework

## How It Works

### 1. Task Analysis
Provide a description of what needs to be done. Agent Launcher analyzes:
- Task complexity (simple, medium, hard)
- Required capabilities (code, docs, analysis, etc.)
- Recommended agent type (specialist, generalist, coordinator)

### 2. Agent Selection
Based on analysis, Agent Launcher selects from your framework's agents:
- **Code Specialist**: Complex code generation, refactoring, optimization
- **Documentation Expert**: API docs, guides, tutorials, explanations
- **Test Generator**: Unit tests, integration tests, test strategies
- **Project Analyzer**: Codebase structure, architecture review, patterns
- **Generalist**: Simple tasks, quick implementations, debugging

### 3. Configuration Optimization
Configures the selected agent with:
- Appropriate model (Haiku for simple, Sonnet for complex)
- Required tools and permissions
- Relevant project context
- Task-specific parameters

### 4. Execution & Results
- Launches the agent with optimized settings
- Monitors execution progress
- Returns results in structured format
- Provides execution metrics (tokens, duration, cost estimate)

## Agent Selection Guide

### When to Use Code Specialist
- Implementing new features
- Complex refactoring
- Performance optimization
- Architecture design
- Multi-file coordination

**Example**: "Implement a new authentication module with OAuth support"

### When to Use Documentation Expert
- API documentation
- User guides
- Implementation tutorials
- Architecture decision records
- Process documentation

**Example**: "Create comprehensive API documentation for our REST endpoints"

### When to Use Test Generator
- Writing unit tests
- Integration test suites
- E2E test scenarios
- Test strategy development
- Coverage analysis

**Example**: "Generate comprehensive unit tests for the payment module"

### When to Use Project Analyzer
- Understanding new codebases
- Architecture reviews
- Identifying refactoring opportunities
- Pattern analysis
- Complexity assessment

**Example**: "Analyze this Python project and identify architectural patterns"

### When to Use Generalist Agent
- Simple tasks under 30 minutes
- Quick debugging
- Minor improvements
- Clarification and questions
- Simple automations

**Example**: "Fix a typo in the README.md file"

## Usage Examples

### Example 1: Code Implementation
```
Request: "Implement a Fibonacci cache system using the Singleton pattern
in Python with comprehensive tests"

Analysis Result:
- Complexity: Medium
- Best Agent: Code Specialist
- Required Tools: Code execution, file system, testing
- Model: Claude Sonnet
- Estimated Tokens: 8,000-12,000
- Estimated Duration: 5-10 minutes

Result: Code Specialist launches with Python execution,
generates implementation + tests, returns code files
```

### Example 2: Documentation
```
Request: "Create a comprehensive user guide for our API including
examples and best practices"

Analysis Result:
- Complexity: Medium
- Best Agent: Documentation Expert
- Required Tools: File generation, markdown processing
- Model: Claude Sonnet
- Estimated Duration: 10-15 minutes

Result: Documentation Expert generates guide with examples,
formats with proper structure, returns markdown + PDF
```

### Example 3: Testing
```
Request: "Write integration tests for the checkout workflow
covering success and failure paths"

Analysis Result:
- Complexity: Medium
- Best Agent: Test Generator
- Required Tools: Test framework, assertions, mock setup
- Model: Claude Sonnet
- Estimated Duration: 8-12 minutes

Result: Test Generator creates comprehensive test suite,
includes fixtures and mocks, returns test files ready to run
```

### Example 4: Quick Task
```
Request: "Update the copyright year in all files to 2025"

Analysis Result:
- Complexity: Simple
- Best Agent: Generalist
- Required Tools: File processing, regex
- Model: Claude Haiku
- Estimated Duration: 2-3 minutes

Result: Generalist updates files efficiently,
returns summary of changes made
```

## Advanced Features

### Multi-Step Execution
For complex tasks, Agent Launcher can chain multiple agents:

```
Step 1: Project Analyzer
  └─ "Analyze codebase structure and identify gaps"

Step 2: Code Specialist
  └─ "Implement missing features based on gaps"

Step 3: Test Generator
  └─ "Create tests for new features"
```

### Task Prioritization
Specify execution priority:
- **Immediate**: Launch agent now
- **Scheduled**: Queue for next batch
- **Background**: Run when resources available

### Resource Constraints
Specify limitations:
- **Max tokens**: 4K, 8K, 16K, unlimited
- **Max duration**: 5 min, 15 min, 1 hour
- **Model preference**: Haiku, Sonnet, Opus

### Execution Monitoring
Real-time monitoring of:
- Agent status (initializing, executing, complete)
- Token usage and estimated cost
- Progress on sub-tasks
- Any errors or warnings

## Reference Information

See REFERENCE.md for:
- Complete agent registry schema
- Configuration parameters
- API endpoint specifications
- Integration examples
- Troubleshooting guide

## Integration with Claude Agent Framework

This skill works seamlessly with:
- **SYSTEM_GENERATOR_PROMPT**: Uses generated agent configs
- **Agent Registry**: Reads from `.claude-library/REGISTRY.json`
- **Agent Patterns**: Selects based on framework patterns
- **Simplicity Enforcement**: Respects circuit breaker settings

## Command Examples

### Basic Execution
```
"Use the Agent Launcher to write unit tests for my validation module"
```

### With Constraints
```
"Launch the Code Specialist to refactor this function,
but keep token usage under 8K"
```

### With Priority
```
"Queue a project analysis to run after current tasks complete"
```

### With Feedback
```
"Run the Documentation Expert on our API, and I want the result
as both markdown and PDF formats"
```

## Performance Notes

- **Selection**: Instant (local registry analysis)
- **Configuration**: <1 second
- **Execution**: Depends on task complexity (1-30 minutes typically)
- **Overhead**: ~500 tokens for selection and setup
- **Context Efficiency**: Uses progressive disclosure to minimize context

## Limitations

This skill cannot:
- Launch agents that require external API access (use agents directly for that)
- Exceed API rate limits on Messages API
- Execute tasks requiring packages not in Python standard library
- Access resources outside the framework configuration

For these cases, launch agents directly using Claude Code's Task tool.

## Success Criteria

Agent Launcher succeeds when:
✅ Correct agent selected for task type
✅ Configuration matches task requirements
✅ Execution completes within estimated time
✅ Results meet quality standards
✅ Token usage is optimized
✅ Output is in requested format

## Troubleshooting

**Problem**: Wrong agent selected
- **Solution**: Provide more specific task description with context

**Problem**: Configuration doesn't match needs
- **Solution**: Specify exact requirements (model, tokens, tools)

**Problem**: Execution exceeds estimated time
- **Solution**: Break task into smaller steps, or increase token limit

**Problem**: Not enough context provided
- **Solution**: Include relevant code files or documentation references

---

For questions or issues, refer to REFERENCE.md or your framework documentation.
