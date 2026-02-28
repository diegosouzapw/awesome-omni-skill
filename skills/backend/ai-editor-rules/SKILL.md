---
name: ai-editor-rules
description: AI代码编辑器规则模板集合 - 为Cursor、Windsurf、Claude Code、Cline等AI编辑器提供项目规则配置。适用于需要配置AI编码助手规则的开发者，包含全栈Web、移动端、Vue3+SpringBoot等技术栈模板。
---

# AI Editor Rules 技能

为各种 AI 代码编辑器提供项目规则配置模板。

## 快速开始

### 复制规则文件到项目

根据你使用的编辑器，复制对应文件到项目根目录：

```
your-project/
├── .cursorrules           # Cursor
├── .windsurfrules         # Windsurf
├── .claude/CLAUDE.md      # Claude Code
└── AGENTS.md
```

###              # 通用 使用 crossrule 转换（推荐）

```bash
# 安装 crossrule
npm install -g crossrule

# 初始化项目
crossrule init

# 转换格式
crossrule convert --from cursor --to windsurf
```

## 支持的编辑器

| 编辑器 | 规则文件 | 位置 |
|--------|----------|------|
| Cursor | `.cursorrules` | 项目根目录 |
| Claude Code | `CLAUDE.md` | `.claude/` 目录 |
| Windsurf | `.windsurfrules` | 项目根目录 |
| Cline | `.clinerules` | 项目根目录 |
| OpenCode | `AGENTS.md` | 项目根目录 |

## 包含的模板

### 1. 全栈 Web (React + Node.js)
- 技术栈：React 19, TypeScript, Node.js, PostgreSQL, Prisma
- 适合：Web 应用开发

### 2. 移动端 (React Native)
- 技术栈：React Native, Expo, TypeScript, Zustand
- 适合：iOS/Android 应用

### 3. Vue3 + SpringBoot3 + MySQL8
- 技术栈：Vue 3, TypeScript, Vite, Spring Boot, MySQL
- 适合：Java 全栈开发

### 4. Python 后端 (FastAPI)
- 技术栈：Python 3.12, FastAPI, SQLAlchemy, PostgreSQL
- 适合：API 开发

## 自定义模板

复制模板后，根据你的项目修改：

1. **技术栈** - 更新框架和库版本
2. **代码风格** - 调整命名规范
3. **项目结构** - 适配你的目录
4. **约束规则** - 添加特定领域规则

## 规则类型说明

### Always (始终生效)
应用于所有代码：
```yaml
alwaysApply: true
```

### Pattern (模式匹配)
特定文件类型：
```yaml
globs:
  - "*.tsx"
  - "*.jsx"
```

### Manual (手动触发)
需要明确调用：
```yaml
trigger: manual
```

## 验证规则

使用对应编辑器的命令验证：

```bash
# Cursor
cursor --rule-check

# Claude Code
claude --verify-rules
```

## 更新日志

- v1.0.0: 初始版本，包含 4 种技术栈模板
