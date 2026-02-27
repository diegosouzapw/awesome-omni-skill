---
name: ace-tool
description: Semantic code search tool. Use `ace search "query"` for codebase search, `ace enhance "history+prompt"` for prompt enhancement. Prefer `rg` for exact matches.

---

# Ace Tool (AugmentCode)

## 包装器

`ace` 是 Bash 包装器（`/usr/local/bin/ace`），调用 `client.ts`。详见 `WRAPPER.md` 和 `COMPARISON.md`。

## 使用

```bash
ace search "Where is auth?"
ace s "auth"                    # 简写
ace enhance "Add login page"
ace e "Add login"               # 简写

# 直接调用
bun ~/.pi/agent/skills/ace-tool/client.ts search "query"
```

## Tools

### 1. Codebase Search (`search`)
**语义搜索 - 代码库搜索的首选工具**

- **何时使用**：需要理解代码库、查找功能、基于自然语言描述定位代码
- **使用案例**：
  - 不知道具体文件位置
  - 需要高层信息
  - **搜索代码/类/函数**优先使用此工具
  - **编辑前规则**：编辑任何文件前，先用此工具收集上下文

### 2. Prompt Enhancement (`enhance`)
**提示增强 - 基于代码库上下文优化需求**

- **触发方式**：
  - 用户消息包含 `-enhance` 或 `-enhancer`（不区分大小写）
  - 用户明确要求 "enhance my prompt"
- **注意**：仅用于需求/提示词优化，不用于代码优化

## Notes

- **包装器**：`/usr/local/bin/ace` 包装 `client.ts`，推荐使用
- **守护进程**：daemon.ts 自动启动并保持运行
- **日志**：项目根目录 `.ace-tool/ace-tool.log`
- **完整文档**：`INDEX.md` | `WRAPPER.md` | `COMPARISON.md`
