---
name: rule-creator
description: Create effective AI agent rules for CLAUDE.md, AGENTS.md, .cursorrules, .clinerules, or .windsurfrules. Use when user mentions "创建规则", "写规则", "create rule", "write rule", "添加规则", "更新规则", "update rule", "修改规则", "规则模板", "rule template", or wants to define coding standards, project conventions, or AI behavior guidelines.
---

# Rule Creator

为不同 IDE 工具创建有效的 AI 代理规则。

## 规则文件格式

| 工具 | 文件位置 | 格式 |
|------|----------|------|
| Claude Code | `CLAUDE.md` | Markdown |
| Cursor | `.cursorrules` 或 `.cursor/rules/*.md` | Markdown + YAML frontmatter |
| Cline | `.clinerules/*.md` | Markdown + YAML frontmatter |
| Windsurf | `.windsurfrules` | Markdown |
| Augment | `.augment/skills/*/SKILL.md` | 用于复杂工作流，简单规则用 CLAUDE.md |
| 通用 | `AGENTS.md` | 纯 Markdown (跨工具兼容) |

## 规则类型 (Cursor/Cline)

**始终应用:**
```yaml
---
alwaysApply: true
---
# 规则内容...
```

**按文件匹配:**
```yaml
---
globs: ["src/components/**", "**/*.tsx"]
---
# 规则内容...
```

**由 Agent 决定:**
```yaml
---
description: "React 组件开发指南"
---
# 规则内容...
```

## 写作原则

### 具体明确
❌ "写好异步代码"
✅ "所有数据库调用使用 async/await，禁止回调"

### 展示模式
❌ "遵循错误处理规范"
✅ "参考 `src/utils/errors.ts` 的错误处理模式"

### 关注结果
❌ "第一步打开文件，第二步..."
✅ "确保所有 API 响应包含 error code 和 message"

### 尽量量化
❌ "函数不要太长"
✅ "单个函数不超过 50 行"

## 规则模板

```markdown
# [规则名称]

## 代码规范
- [具体规范 1]
- [具体规范 2]

## 架构约束
- [约束 1]
- [约束 2]

## 禁止事项
- [禁止 1]
- [禁止 2]

## 参考文件
- `path/to/example.ts` - [说明]
```

## 适合写成规则的内容

✅ **适合写成规则**:
- 代码规范 (命名、格式、风格)
- 架构决策 (设计模式、依赖规则)
- 团队约定 (PR 流程、分支命名、commit 格式)
- 技术约束 (必须使用的库、禁用的 API)
- 安全要求 (敏感信息处理、权限检查)

❌ **不适合写成规则**:
- 一次性指令 (直接在对话中说)
- 复杂多步骤工作流 (用 Skill 或 Workflow)
- 依赖运行时上下文的动态决策

## 工作流程

1. **确认目标**: 询问规则要解决什么问题
2. **识别 IDE**: 检查项目中已有的规则文件 (`.cursorrules`, `.windsurfrules`, `CLAUDE.md` 等)
3. **选择格式**:
   - 单一 IDE → 使用对应格式
   - 多 IDE 环境 → 优先 `AGENTS.md` (通用兼容)
   - 复杂工作流 → 建议使用 Skill 而非 Rule
4. **起草规则**: 使用模板，保持简洁具体
5. **验证规则**:
   - 规则是否可被检查遵守？
   - 是否有正确/错误示例？
   - 是否与现有规则冲突？
6. **放置文件**: 保存到正确位置，遵循命名约定

## 示例: Backend API 规则 (带 frontmatter)

```yaml
---
globs: ["src/routes/**", "src/controllers/**"]
---

# Backend API 规范

## 路由处理
- 使用 async/await，禁止回调
- 请求体使用 Zod schema 验证
- 错误返回使用 `src/errors.ts` 中的类型

## 数据库访问
- 所有查询通过 `src/repositories/` 中的类
- 多表更新使用事务
- 禁止向客户端暴露原始数据库错误

## 测试要求
- 业务逻辑: `src/services/` 单元测试
- 路由处理: `src/routes/` 集成测试
- Mock 外部 API，不 mock 内部模块
```

## 条件规则 (Cursor/Cline)

限定规则只在特定文件生效:

```yaml
---
globs:
  - "src/components/**"
  - "src/hooks/**"
---

# React 组件规范

使用函数组件和 hooks。可复用逻辑提取为自定义 hook。
```

## 提示

1. **一个规则一个关注点**: 避免规则过于庞大
2. **可验证**: 规则应该能被检查是否遵守
   - ✅ "函数不超过 50 行" (可检查)
   - ❌ "代码要简洁" (无法检查)
3. **有示例**: 包含正确和错误的代码示例
4. **定期更新**: 根据实际使用反馈迭代规则
5. **团队共识**: 规则应反映团队共识，而非个人偏好

## Rule vs Skill

| 场景 | 使用 |
|------|------|
| 简单约定、代码规范 | Rule (本 skill) |
| 复杂多步骤工作流 | Skill (skill-creator) |
| 需要脚本/资源文件 | Skill |
| 一次性指令 | 直接在对话中说 |

