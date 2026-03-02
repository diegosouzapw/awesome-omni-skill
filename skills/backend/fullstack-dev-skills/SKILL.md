---
name: "fullstack-dev-skills"
description: "Use when building full-stack applications requiring frontend, backend, DevOps, security, AI Agent, or ML expertise. Invoke for Vue, React, Node.js, Docker, Kubernetes, authentication, database optimization, RAG, or long-running agent patterns."
license: MIT
metadata:
  author: "Trae AI"
  version: "2.6.0"
  triggers: "Vue, React, Next.js, Node.js, API, database, Docker, Kubernetes, security, Agent, testing, CI/CD, authentication, ML, RAG, Python, TypeScript, Go, Rust"
  role: specialist
  scope: implementation
  output-format: code
  domain: fullstack
  related-skills: "code-generator, code-reviewer, security-auditor, test-generator, deepagents"
---

# 全栈开发专家技能集

参考 jeffallan/claude-skills 项目设计理念，整合全栈开发核心能力，采用渐进式披露架构。

## Role Definition

全栈开发专家，具备 10+ 年全栈开发经验，精通前端框架、后端技术、DevOps、安全实践、AI Agent 开发和机器学习。

## When to Use This Skill

- 构建前端应用（Vue、React、Next.js、Angular、Flutter）
- 开发后端 API（Node.js、Python、Go、Rust、Java）
- 数据库设计与优化（SQL、NoSQL、PostgreSQL）
- DevOps 实践（Docker、Kubernetes、CI/CD）
- 安全开发（认证授权、OWASP 防护）
- AI Agent 开发（长运行代理、会话管理）
- 机器学习（RAG、Fine-tuning、Prompt Engineering）
- 测试策略（单元测试、集成测试、E2E）

## Core Workflow

1. **需求分析** - 理解业务需求，确定技术栈
2. **架构设计** - 设计系统架构，选择合适方案
3. **实现开发** - 编写代码，遵循最佳实践
4. **测试验证** - 编写测试，确保质量
5. **部署运维** - CI/CD 配置，监控告警

## Reference Guide

### 语言专家 (12)
| Topic | Reference | Load When |
|-------|-----------|-----------|
| Python Pro | `references/language-python.md` | Python 类型提示、异步、Pydantic |
| TypeScript Pro | `references/language-typescript.md` | 高级类型、泛型、工具类型 |
| JavaScript Pro | `references/language-javascript.md` | ES2024+、异步模式、模块化 |
| Go Pro | `references/language-go.md` | 并发、Goroutines、Channels |
| Rust Engineer | `references/language-rust.md` | 所有权、生命周期、异步 Rust |
| SQL Pro | `references/language-sql.md` | 窗口函数、CTE、查询优化 |
| C++ Pro | `references/language-cpp.md` | 现代 C++、RAII、智能指针 |
| Swift Expert | `references/language-swift.md` | SwiftUI、Combine、并发 |
| Kotlin Specialist | `references/language-kotlin.md` | 协程、Flow、Jetpack Compose |
| C# Developer | `references/language-csharp.md` | LINQ、async/await、Record |
| PHP Pro | `references/language-php.md` | 现代 PHP、Laravel 模式 |
| Java Architect | `references/language-java.md` | Spring 生态、虚拟线程 |

### 后端框架 (7)
| Topic | Reference | Load When |
|-------|-----------|-----------|
| NestJS Expert | `references/backend-nestjs.md` | NestJS 模块、依赖注入、守卫 |
| Django Expert | `references/backend-django.md` | DRF、异步视图、ORM 优化 |
| FastAPI Expert | `references/backend-python.md` | FastAPI、Pydantic、异步 |
| Spring Boot Engineer | `references/backend-spring-boot.md` | 自动配置、Actuator、微服务 |
| Laravel Specialist | `references/backend-laravel.md` | Eloquent、队列、事件 |
| Rails Expert | `references/backend-rails.md` | Hotwire、Turbo、ActiveJob |
| .NET Core Expert | `references/backend-dotnet.md` | Minimal APIs、EF Core |

### 前端与移动 (6)
| Topic | Reference | Load When |
|-------|-----------|-----------|
| React Expert | `references/frontend-react.md` | React Hooks、Server Components |
| Next.js Developer | `references/frontend-nextjs.md` | App Router、SSR、Server Actions |
| Vue Expert | `references/frontend-vue.md` | Composition API、Pinia |
| Angular Architect | `references/frontend-angular.md` | Signals、独立组件、RxJS |
| React Native Expert | `references/frontend-react-native.md` | 新架构、Fabric、性能优化 |
| Flutter Expert | `references/frontend-flutter.md` | Riverpod、Freezed、Widget |

### 基础设施与云 (5)
| Topic | Reference | Load When |
|-------|-----------|-----------|
| Kubernetes Specialist | `references/infrastructure-kubernetes.md` | K8s 部署、Helm、服务网格 |
| Terraform Engineer | `references/infrastructure-terraform.md` | IaC、多云配置、模块化 |
| Postgres Pro | `references/infrastructure-postgres.md` | 复制、分区、JSONB |
| Cloud Architect | `references/infrastructure-cloud.md` | AWS/Azure/GCP、多云策略 |
| Database Optimizer | `references/infrastructure-database-optimizer.md` | 索引策略、查询优化 |

### API 与架构 (7)
| Topic | Reference | Load When |
|-------|-----------|-----------|
| GraphQL Architect | `references/api-graphql.md` | Schema 设计、Federation |
| API Designer | `references/backend-api.md` | RESTful、OpenAPI、认证 |
| WebSocket Engineer | `references/api-websocket.md` | Socket.IO、实时通信 |
| Microservices Architect | `references/api-microservices.md` | 服务网格、Saga、CQRS |
| MCP Developer | `references/api-mcp.md` | Model Context Protocol |
| Architecture Designer | `references/architecture-designer.md` | ADR、决策树、权衡分析 |
| Feature Forge | `references/feature-forge.md` | 需求收集、验收标准 |

### 质量与测试 (5)
| Topic | Reference | Load When |
|-------|-----------|-----------|
| Test Master | `references/testing-guide.md` | 测试策略、覆盖率、First run the tests |
| Playwright Expert | `references/quality-playwright.md` | E2E、视觉测试、并行执行 |
| Code Reviewer | `references/quality-code-reviewer.md` | 审查清单、最佳实践 |
| Code Documenter | `references/quality-code-documenter.md` | API 文档、架构图 |
| TDD Agent Expert | `references/patterns-red-green-tdd-agent.md` | Red/Green TDD、测试优先、边缘情况 |

### DevOps 与运维 (5)
| Topic | Reference | Load When |
|-------|-----------|-----------|
| DevOps Engineer | `references/devops-docker.md` | CI/CD、容器化、GitOps |
| Monitoring Expert | `references/devops-monitoring.md` | 日志、指标、追踪、告警 |
| SRE Engineer | `references/devops-sre.md` | SLO/SLA、事件响应 |
| Chaos Engineer | `references/devops-chaos.md` | 故障注入、韧性测试 |
| CLI Developer | `references/devops-cli.md` | CLI 框架、Shell 集成 |

### 安全 (2)
| Topic | Reference | Load When |
|-------|-----------|-----------|
| Secure Code Guardian | `references/security-secure-code.md` | 安全编码、漏洞防护 |
| Security Reviewer | `references/security-reviewer.md` | 安全审计、渗透测试 |

### 数据与机器学习 (6)
| Topic | Reference | Load When |
|-------|-----------|-----------|
| Pandas Pro | `references/data-pandas.md` | DataFrame、聚合、时间序列 |
| Spark Engineer | `references/data-spark.md` | PySpark、分布式处理 |
| ML Pipeline | `references/data-ml-pipeline.md` | MLflow、Kubeflow、特征存储 |
| Prompt Engineer | `references/data-prompt-engineer.md` | 提示设计、CoT、Few-shot |
| RAG Architect | `references/data-rag.md` | 向量数据库、嵌入、语义搜索 |
| Fine-Tuning Expert | `references/data-fine-tuning.md` | LoRA、QLoRA、PEFT |

### 平台专家 (4)
| Topic | Reference | Load When |
|-------|-----------|-----------|
| Salesforce Developer | `references/platform-salesforce.md` | Apex、LWC、SOQL |
| Shopify Expert | `references/platform-shopify.md` | Liquid、Storefront API |
| WordPress Pro | `references/platform-wordpress.md` | 主题、插件、Gutenberg |
| Atlassian MCP | `references/platform-atlassian.md` | Jira、Confluence、JQL |

### 专业领域 (3)
| Topic | Reference | Load When |
|-------|-----------|-----------|
| Legacy Modernizer | `references/domain-legacy.md` | 迁移策略、重构、现代化 |
| Embedded Systems | `references/domain-embedded.md` | IoT、固件、实时系统 |
| Game Developer | `references/domain-game.md` | 游戏引擎、ECS、物理 |

### 工具角色 (2)
| Topic | Reference | Load When |
|-------|-----------|-----------|
| Debugging Wizard | `references/tool-debugging.md` | 系统化调试、根因分析 |
| Context Engineering | `references/context-engineering.md` | Common Ground、假设验证 |

### 工作流程与组合 (2)
| Topic | Reference | Load When |
|-------|-----------|-----------|
| Workflow Commands | `references/workflow-commands.md` | 功能开发、Bug 修复、代码审查 |
| Skill Combinations | `references/skill-combinations.md` | 多技能协作、工作流组合 |

### AI Agent 开发 (1)
| Topic | Reference | Load When |
|-------|-----------|-----------|
| AI Agent Development | `references/ai-agent-development.md` | 长运行代理、会话管理、功能列表 |

### AI Agent 协作 (8)
| Topic | Reference | Load When |
|-------|-----------|-----------|
| Agentic Engineering | `references/patterns-agentic-engineering.md` | 十大策略、代码成本革命、工程习惯 |
| LLM Collaboration | `references/patterns-llm-collaboration.md` | 人机协作、上下文管理、迭代对话 |
| Code Walkthrough | `references/patterns-code-walkthrough.md` | 线性演示、Showboat、代码理解 |
| Vibe Coding | `references/patterns-vibe-coding.md` | 直觉编程、Prompt 模板、快速原型 |
| Developer Action Guide | `references/guide-developer-action.md` | 技能培养、工具整合、风险管理 |
| MCP Protocol | `references/patterns-mcp-protocol.md` | 模型上下文协议、Tools/Resources/Prompts |
| Computer Use | `references/patterns-computer-use.md` | 桌面自动化、Claude Computer Use、CUA |
| Agentic System | `references/patterns-agentic-system.md` | 多代理协作、任务编排、状态管理 |

### FastCode 代码理解 (8)
| Topic | Reference | Load When |
|-------|-----------|-----------|
| Semantic-Structural Representation | `references/patterns-fastcode-understanding.md` | AST 解析、多层索引、图建模 |
| Lightning-Fast Navigation | `references/patterns-fastcode-navigation.md` | 两阶段搜索、代码略读、签名提取 |
| Cost-Efficient Context | `references/patterns-fastcode-context.md` | 预算感知、增量更新、MCP 集成 |
| Agent Tools | `references/patterns-fastcode-agent-tools.md` | 目录列表、代码搜索、结构摘要 |
| Dialogue Cache | `references/patterns-fastcode-cache.md` | 对话保存、历史检索、会话管理 |
| Multi-Repo Reasoning | `references/patterns-fastcode-multi-repo.md` | 仓库选择、文件选择、模糊匹配 |
| Streaming Answer | `references/patterns-fastcode-answer-gen.md` | 流式输出、摘要提取、Token 预算 |
| Hybrid Retrieval | `references/patterns-fastcode-hybrid-retrieval.md` | 多阶段检索、Agency 模式、图扩展 |

## Expert Roles Summary

**84 Skills | 15 Categories | 9 Workflows**

### Language Specialists (12)
Python Pro, TypeScript Pro, JavaScript Pro, Go Pro, Rust Engineer, SQL Pro, C++ Pro, Swift Expert, Kotlin Specialist, C# Developer, PHP Pro, Java Architect

### Backend Frameworks (7)
NestJS Expert, Django Expert, FastAPI Expert, Spring Boot Engineer, Laravel Specialist, Rails Expert, .NET Core Expert

### Frontend & Mobile (6)
React Expert, Next.js Developer, Vue Expert, Angular Architect, React Native Expert, Flutter Expert

### Infrastructure & Cloud (5)
Kubernetes Specialist, Terraform Engineer, Postgres Pro, Cloud Architect, Database Optimizer

### API & Architecture (7)
GraphQL Architect, API Designer, WebSocket Engineer, Microservices Architect, MCP Developer, Architecture Designer, Feature Forge

### Quality & Testing (5)
Test Master, Playwright Expert, Code Reviewer, Code Documenter, TDD Agent Expert

### DevOps & Operations (5)
DevOps Engineer, Monitoring Expert, SRE Engineer, Chaos Engineer, CLI Developer

### Security (2)
Secure Code Guardian, Security Reviewer

### Data & Machine Learning (6)
Pandas Pro, Spark Engineer, ML Pipeline, Prompt Engineer, RAG Architect, Fine-Tuning Expert

### Platform Specialists (4)
Salesforce Developer, Shopify Expert, WordPress Pro, Atlassian MCP

### Specialized Domains (3)
Legacy Modernizer, Embedded Systems, Game Developer

### Utility Roles (2)
Debugging Wizard, Context Engineering (The Fool)

### AI Agent Collaboration (8)
Agentic Engineering, LLM Collaboration, Code Walkthrough, Vibe Coding, Developer Action Guide, MCP Protocol, Computer Use, Agentic System

### FastCode Code Understanding (8)
Semantic-Structural Representation, Lightning-Fast Navigation, Cost-Efficient Context, Agent Tools, Dialogue Cache, Multi-Repo Reasoning, Streaming Answer, Hybrid Retrieval

### Skill Self-Improvement (1)
Skill Audit, Quality Evaluation, Improvement Suggestions, GitHub Sync

## Knowledge Reference

Vue, React, Next.js, Angular, Flutter, React Native, Node.js, NestJS, FastAPI, Django, Spring Boot, Laravel, Rails, .NET Core, Python, TypeScript, Go, Rust, Java, C++, Swift, Kotlin, C#, PHP, SQL, PostgreSQL, MongoDB, Redis, Docker, Kubernetes, Terraform, AWS, Azure, GCP, GraphQL, WebSocket, Microservices, MCP, JWT, OAuth2, RBAC, OWASP, CI/CD, GitHub Actions, Playwright, Vitest, MLflow, Kubeflow, Pandas, Spark, RAG, LoRA, QLoRA, Prompt Engineering, AI Agent, Long-Running Agent, Feature List, Session Management, Progressive Disclosure, Agentic Engineering, Red/Green TDD, Vibe Coding, Code Walkthrough, LLM Collaboration, Context Management, Iterative Dialogue, Sandbox Execution, MCP Protocol, Computer Use, CUA, Multi-Agent, Task Orchestration, State Management, Error Recovery, FastCode, AST Parsing, Semantic Index, BM25, Call Graph, Dependency Graph, Inheritance Graph, Token Budget, Incremental Update, Agent Tools, Directory Listing, Code Search, File Structure, Dialogue Cache, Session Management, Multi-Repo, Fuzzy Matching, Jaccard Similarity, Streaming Answer, Summary Extraction, Hybrid Retrieval, Agency Mode, Graph Expansion
