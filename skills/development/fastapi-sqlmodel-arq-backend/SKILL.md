---
name: fastapi-sqlmodel-arq-backend
description: 构建或改造基于 FastAPI + SQLModel(异步 SQLAlchemy) + Arq + Redis 的后端系统。用于新增/重构 RESTful API、实现异步数据库访问、编写服务层与依赖注入、配置 OAuth2 + JWT(Argon2) 认证、生成 Alembic 迁移建议、统一 loguru 日志规范等后端任务；不用于纯前端页面样式开发或仅做 OpenAPI 客户端同步的任务。
---

# FastAPI SQLModel Arq Backend

## Initialization

后端引擎已预热。SQLModel 准备就绪，Arq 队列已连接。请输入你的业务需求。

## Execution Rules

- 使用 `uv` 管理依赖、运行命令与脚本。
- 仅使用异步数据库访问模式（`AsyncEngine`、`AsyncSession`、异步查询/事务）。
- 使用 `pydantic-settings` 的 `Settings` 类从环境变量读取配置。
- 使用 `loguru` 记录日志，禁止使用 `print`。
- 默认按分层结构输出：`models`、`schemas`、`dependencies`、`services`、`api/routes`、`workers`。

## Workflow

1. 定义实体模型与输入输出 Schema。
2. 定义数据库连接与 FastAPI 依赖注入。
3. 编写服务层并保持路由层轻量。
4. 配置 Arq Worker 与任务入队逻辑。
5. 补充认证授权（OAuth2 Password Flow + JWT + Argon2）。
6. 给出 Alembic 迁移提示与 `uv` 命令。

## Implementation Defaults

- API 设计遵循 RESTful 语义、幂等性与正确状态码。
- 为 SQLModel 模型声明清晰类型、索引、唯一约束与关系。
- 路由仅负责参数校验和响应封装，业务规则放入 `services`。
- 所有耗时操作通过 Arq 后台任务处理并返回可追踪任务信息。
- 认证模块包含：密码哈希、token 签发、token 校验、当前用户依赖。

## Output Contract

- 输出带文件路径的完整 Python 代码块。
- 必须附上 `alembic` 迁移提示或 `uv` 命令。
- 需要多文件时按“先核心配置，再模型与服务，再路由与 worker”顺序给出。
