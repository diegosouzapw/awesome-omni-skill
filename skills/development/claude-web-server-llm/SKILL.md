---
name: claude-web-server-llm
description: "多线程Claude Web服务器，使用FastAPI将Claude暴露为后端API服务。此服务器会话持久化、支持并发处理、可基于用户语义调用所有skills/agents/subagents直至完成输出给用户。使用场景：（1）启动Claude作为Web服务供其他应用调用，（2）通过API集成Claude能力到现有系统，（3）构建基于Claude的应用后端。此skill仅用于启动和管理服务器本身，不直接处理知识库查询或代码分析任务。(4)通过任务的形式用_execute_subtask_direct发送邮件。"

# ============ 第一部分：意图识别和理解 ============

# 此skill的专属用途（直接由此skill处理）：
exclusive_purpose: |
  用户明确要求以下与服务器相关的操作：
  - "启动"、"运行"、"开启" web服务器/API服务器
  - "将Claude做成Web服务"、"启动后端服务"
  - "访问localhost:8085"、"连接到Claude API"
  - 管理服务器相关配置（端口、线程数、session超时等）
  - 停止、重启、监控服务器
  - 查看服务器状态、统计信息

# 此skill的扩展用途（由服务器内的LLM语义调度处理）：
delegated_purpose: |
  用户通过服务器API发送的请求，服务器内部会根据以下规则判断：
  1. 检查请求是否为复杂多步骤任务
  2. 分解复杂请求为子任务并识别依赖关系
  3. 根据子任务特征路由到相应的skill/agent/subagent
  4. 管理各子任务的超时和失败恢复

# ============ 第二部分：复杂请求的意图理解规则 ============

complex_request_analysis: |
  复杂请求识别特征：
  - 包含"和"、"然后"、"之后"、"完成后"等连接词
  - 要求多个工具协作（如网络搜索 + skill执行 + 文件处理 + 邮件发送等）
  - 包含形式转换（Markdown → Word → Email）
  - 数据流向多个目标（生成 → 转换 → 发送）

  意图理解流程：
  1. 关键词提取：识别所有动词和工具名称
  2. 优先级分析：区分"主要意图"vs"辅助要求"
     - 主要意图：核心业务逻辑（生成内容）
     - 辅助要求：形式和分发（Word格式、邮件发送）
  3. 依赖关系图：建立任务执行顺序
  4. 超时管理：每个子任务配置独立超时

  示例分析 - 微信文章+Word+邮件任务：
    输入："访问GitHub获取指定内容，利用wechat article editor skill生成文章，最后转为Word发到邮箱"

    错误理解（丢失关键步骤）：
      ❌ 主要意图 = "生成Word并发邮件"
      ❌ 丢失步骤1：WebFetch GitHub内容
      ❌ 丢失步骤2：调用wechat article editor skill

    正确理解（完整的多步骤流程）：
      ✓ 子任务1: WebFetch GitHub → (5分钟超时)
      ✓ 子任务2: 调用wechat-article-editor skill → (5分钟超时) [依赖任务1]
      ✓ 子任务3: 调用docx skill转Word → (5分钟超时) [依赖任务2]
      ✓ 子任务4: 发送邮件 → (5分钟超时) [依赖任务3]
      总超时 = 20分钟

# 与其他skill的区分：
exclusion_rules:
  1: "如果用户只是想查询知识库，使用knowledge-base skill"
  2: "如果用户想分析代码库结构，使用Explore Agent"
  3: "如果用户想实现复杂功能需要规划，使用Plan Agent"
  4: "如果用户问Claude使用相关的问题，使用claude-code-guide Agent"

# 何时使用LLM自主判断：
fallback_rules:
  1: "如果用户的请求模棱两可，询问确认"
  2: "如果用户的需求超出了web server的范畴，告知客户不能直接处理但可以通过服务器API调用相关skill"
  3: "如果是技术架构问题或需要多个服务配合，让服务器内部LLM判断并进行任务分解"
  4: "遇到复杂请求时，优先进行意图分析和任务分解，而不是直接生成输出"

# ============ 第三部分：超时管理策略 ============

timeout_strategy: |
  超时层级结构：
  1. 全局超时：整个任务链的最大时间
     - 简单任务（单一skill调用）：默认5分钟
     - 复杂任务（多步骤）：根据子任务数 × 5分钟

  2. 步骤超时：每个子任务的独立超时
     - WebFetch/API调用：5分钟
     - Skill调用（生成类）：5分钟
     - 文件转换（格式转换）：5分钟
     - 邮件发送/外部API：5分钟

  3. 超时恢复策略：
     - 若某个子任务超时，立即标记为失败
     - 检查是否有后续依赖任务，若有则阻止执行
     - 返回详细错误信息给用户（哪个步骤超时、原因、建议）

  4. 超时通知：
     - 后端记录每个步骤的执行时间
     - 若某步接近超时（>80%），发送警告
     - 完成后返回执行时间统计

execution:
  type: builtin
  config:
    server_port: 8085
    max_sessions: 100
    worker_threads: 16
    claude_executable: claude
    session_timeout: 3600

---

# Claude Web Server LLM

多线程Claude Web服务器，使用Claude作为后端进行语义路由和智能调度。

## 功能特性

- 持久化Claude会话（通过claude --print命令交互）
- 多线程并发处理（ThreadPoolExecutor）
- 自动发现所有skills、agents、subagents
- 基于LLM语义分析智能路由到skills、agents、subagents
- 无法判定时直接调用Claude处理直至完成
- RESTful API接口
- 会话管理和隔离
- WebUI前端界面
- 支持多模型后端（Claude、NVIDIA、Deepseek）

## 快速开始

```bash
# 启动服务器
python3 /home/will/.claude/skills/claude-web-server-llm/scripts/server.py

# 或使用启动脚本
bash /home/will/.claude/skills/claude-web-server-llm/run.sh

# 访问Web界面
# http://localhost:8085
```

## 架构设计

```
Web Server (8085端口)
├── 会话管理
│   ├── Session 1 → Claude/模型调用
│   ├── Session 2 → Claude/模型调用
│   └── Session N → Claude/模型调用
├── 语义调度系统
│   ├── Skill发现模块 (~/.claude/skills/)
│   ├── Agent类型识别 (Bash, Explore, Plan等)
│   └── LLM语义分析器 (智能路由)
├── 多模型后端
│   ├── Claude (opus, sonnet, haiku)
│   ├── NVIDIA API
│   └── Deepseek API
└── 前端WebUI (浏览器界面)
    ├── 输入窗口 (底部)
    ├── 输出窗口 (顶部)
    └── 模型选择
```

## API端点

### 会话管理
- `POST /api/session` - 创建新会话
- `GET /api/session/{session_id}` - 获取会话信息
- `DELETE /api/session/{session_id}` - 删除会话

### 消息交互
- `POST /api/send` - 发送消息到Claude
- `POST /api/claude` - 直接调用Claude处理
- `GET /api/messages/{session_id}` - 获取会话消息历史

### 资源发现
- `GET /api/skills` - 列出所有可用skills
- `GET /api/agents` - 列出所有agents
- `GET /api/skills/summary` - 获取skills摘要

## 使用场景

| 场景 | 使用方法 |
|------|----------|
| 启动服务器 | 用户说"启动Claude web llm服务器" |
| 查看服务状态 | 访问 http://localhost:8085/api/status |
| 获取可用资源 | 访问 http://localhost:8085/api/skills |
| 发送消息 | POST 到 /api/send 端点 |

## 配置说明

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `server_port` | 8085 | 服务器监听端口 |
| `max_sessions` | 100 | 最大同时会话数 |
| `worker_threads` | 16 | 工作线程数 |
| `claude_executable` | claude | Claude可执行文件路径 |
| `session_timeout` | 3600 | 会话超时时间（秒） |

## 注意事项

1. 此skill仅用于**启动和管理web服务器本身**
2. 通过API调用Claude时，语义调度系统会智能判断使用何种skill/agent/subagent
3. 无法明确判断的请求会直接发给Claude处理
4. 服务器运行后，可以通过Web界面或API与Claude交互
