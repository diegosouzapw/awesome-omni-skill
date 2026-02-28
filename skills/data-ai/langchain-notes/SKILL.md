---
name: langchain-notes
description: LangChain 框架学习笔记 - 快速查找概念、代码示例和最佳实践。包含 Core components、Middleware、Advanced usage、Multi-agent patterns、RAG retrieval、Long-term memory 等主题。当用户询问 LangChain、Agent、RAG、向量存储、工具使用、记忆系统时使用此 Skill。
allowed-tools:
  - Read
  - Grep
  - Glob
---

# LangChain 学习笔记 Skill

快速访问 LangChain 框架的学习资料、代码示例和最佳实践。

## 快速导航

### 核心组件 (Core Components)
- **[01-models.md](langchain_notes/core-components/01-models.md)** - 模型使用 (Chat、Completion、Streaming)
- **[02-messages.md](langchain_notes/core-components/02-messages.md)** - 消息类型 (Human、AI、System、Tool、Custom)
- **[03-tools.md](langchain_notes/core-components/03-tools.md)** - 工具定义和使用
- **[04-memory.md](langchain_notes/core-components/04-memory.md)** - 短期记忆管理
- **[05-structured-output.md** - 结构化输出
- **[06-streaming.md](langchain_notes/core-components/06-streaming.md)** - 流式输出
- **[07-agents.md](langchain_notes/core-components/07-agents.md)** - Agent 基础

### 中间件 (Middleware)
- **[01-built-in.md](langchain_notes/middleware/01-built-in.md)** - 内置中间件
- **[02-custom.md](langchain_notes/middleware/02-custom.md)** - 自定义中间件

### 高级用法 (Advanced Usage)
- **[01-guardrails.md](langchain_notes/advanced-usage/01-guardrails.md)** - 输出验证和安全
- **[02-runtime.md](langchain_notes/advanced-usage/02-runtime.md)** - 运行时配置
- **[03-context-engineering.md](langchain_notes/advanced-usage/03-context-engineering.md)** - 提示工程
- **[04-mcp.md](langchain_notes/advanced-usage/04-mcp.md)** - MCP 协议
- **[05-human-in-the-loop.md](langchain_notes/advanced-usage/05-human-in-the-loop.md)** - 人机协作
- **[06-retrieval.md](langchain_notes/advanced-usage/06-retrieval.md)** - RAG 检索增强生成
- **[07-long-term-memory.md](langchain_notes/advanced-usage/07-long-term-memory.md)** - 长期记忆存储

### Multi-agent 模式
- **[01-subagents.md](langchain_notes/advanced-usage/multi-agent/01-subagents.md)** - Supervisor 模式
- **[02-handoffs.md](langchain_notes/advanced-usage/multi-agent/02-handoffs.md)** - Handoff 模式
- **[03-skills.md](langchain_notes/advanced-usage/multi-agent/03-skills.md)** - Skills 模式
- **[04-router.md](langchain_notes/advanced-usage/multi-agent/04-router.md)** - Router 模式
- **[05-custom-workflow.md](langchain_notes/advanced-usage/multi-agent/05-custom-workflow.md)** - 自定义工作流

### 集成笔记 (Integrations Notes)
- **[chroma.md](integrations_notes/vectorstores/chroma.md)** - Chroma 向量数据库集成

## 使用指南

### 查找概念
使用 Grep 工具搜索特定概念：
```bash
# 搜索关键词
grep -r "关键词" langchain_notes/

# 示例：搜索 RAG 相关内容
grep -r "RAG\|retrieval\|向量存储" langchain_notes/
```

### 查找代码示例
笔记中包含大量可运行的代码示例，按以下格式组织：
- 概述 → 核心概念
- 基础实现 → 简单示例
- 完整实现 → 生产就绪代码
- 最佳实践 → 推荐做法

### 风格规范
所有笔记遵循以下风格：
- **简洁优先** - 避免冗余,突出重点
- **结构清晰** - 层级分明,易于查找
- **代码优先** - 用代码说明概念
- **格式统一** - 使用表格、列表、加粗

## 常见任务

### 创建 Agent
参考：[07-agents.md](langchain_notes/core-components/07-agents.md)

```python
from langchain.agents import create_agent
from langchain.tools import tool

@tool
def my_tool(input: str) -> str:
    """工具描述."""
    return f"处理: {input}"

agent = create_agent(
    model="claude-sonnet-4-5-20250929",
    tools=[my_tool],
    system_prompt="你是一个有用的助手"
)
```

### 实现 RAG
参考：[06-retrieval.md](langchain_notes/advanced-usage/06-retrieval.md)

**2-Step RAG** (简单快速):
```python
@dynamic_prompt
def prompt_with_context(request: ModelRequest) -> str:
    last_query = request.state["messages"][-1].text
    docs = vector_store.similarity_search(last_query)
    return f"Context: {docs}"
```

**Agentic RAG** (灵活):
```python
@tool
def retrieve_context(query: str):
    """检索信息."""
    return vector_store.similarity_search(query)
```

### Multi-agent 模式
参考：[multi-agent/](langchain_notes/advanced-usage/multi-agent/)

| 模式 | 适用场景 | 文件 |
|-----|---------|------|
| **Supervisor** | 中央协调多个子 Agent | 01-subagents.md |
| **Handoff** | Agent 间协作转移 | 02-handoffs.md |
| **Skills** | 专业化能力按需加载 | 03-skills.md |
| **Router** | 分类路由到专门 Agent | 04-router.md |
| **Custom Workflow** | 完全自定义执行流程 | 05-custom-workflow.md |

### 记忆管理
参考：[07-long-term-memory.md](langchain_notes/advanced-usage/07-long-term-memory.md)

**读取长期记忆**:
```python
@tool
def get_user_info(runtime: ToolRuntime[Context]) -> str:
    store = runtime.store
    user_id = runtime.context.user_id
    return store.get(("users",), user_id)
```

**写入长期记忆**:
```python
store.put(("users",), user_id, {"name": "John"})
```

## 搜索技巧

### 按主题搜索
```bash
# RAG 相关
grep -r "RAG\|检索\|向量存储\|similarity_search" langchain_notes/

# Agent 相关
grep -r "Agent\|create_agent\|tool" langchain_notes/

# Multi-agent 相关
grep -r "Supervisor\|Handoff\|Router\|Orchestrator" langchain_notes/

# 记忆相关
grep -r "memory\|store\|checkpointer" langchain_notes/
```

### 按文件类型搜索
```bash
# 查找所有 Markdown 文件
find langchain_notes/ -name "*.md"

# 查找特定目录
ls langchain_notes/core-components/
ls langchain_notes/advanced-usage/
```

## 参考资料位置

- **笔记根目录**: `langchain_notes/`
- **集成笔记**: `integrations_notes/`
- **官方文档**: `oss_python_docs/`
- **示例代码**: 每个笔记文件中的代码块

## 官方文档结构

当笔记中找不到相关内容时，从 `oss_python_docs/` 中查找官方文档。

### 一级目录结构

```
oss_python_docs/
├── langchain/          # LangChain 核心文档
├── langgraph/          # LangGraph 工作流文档
├── integrations/       # 第三方集成文档
│   ├── vectorstores/   # 向量存储集成 (Chroma, FAISS, Pinecone 等)
│   ├── retrievers/     # 检索器集成
│   └── ...
└── ...
```

### 搜索官方文档

```bash
# 搜索集成文档
grep -r "chroma\|faiss\|pinecone" oss_python_docs/integrations/vectorstores/

# 搜索核心概念
grep -r "agent\|retrieval\|memory" oss_python_docs/langchain/

# 查找特定文件
find oss_python_docs/ -name "*.md" | grep -i "rag"
```

**注意**: 官方文档为英文原始文档，用于深入查阅。笔记是提取的精华内容。

## 注意事项

1. **环境配置**:
   ```bash
   conda activate ai_tools
   ```

2. **代码优先**: 所有示例都包含完整可运行的代码

3. **中英混合**: 概念和描述使用中文,代码和 API 使用英文

4. **状态标记**: 每个笔记底部标记完成状态

## 获取帮助

当用户询问以下问题时使用此 Skill：
- "LangChain 如何实现 X?"
- "Agent/RAG/向量存储怎么用?"
- "有什么最佳实践?"
- "给我看代码示例"
- "Multi-agent 模式有哪些?"

搜索笔记目录找到相关文件,提取关键信息和代码示例。
