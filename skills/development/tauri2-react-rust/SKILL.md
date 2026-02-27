---
name: tauri2-react-rust
description: Guides development of cross-platform desktop apps with Tauri 2, TypeScript, React, and Rust. Use when building Tauri apps, implementing IPC, designing Rust backend or TypeScript/React frontend, when researching or cloning a website (open site, snapshot elements), when verifying local dev or built frontend in browser, or when the user mentions Tauri, Tauri 2, Rust backend, React frontend, desktop app architecture, invoke/commands, cross-platform, 调研网站, 验证效果, agent-browser.
---

# Tauri 2 + React + Rust 跨平台开发

基于 Tauri 2、TypeScript、React、Rust 技术栈的跨平台桌面应用开发指引。整合架构、IPC、后端与前端最佳实践。

## 何时使用本 Skill

- 设计或实现 Tauri 应用架构、IPC 通信
- 编写 Rust 后端命令、状态、错误处理
- 编写 TypeScript/React 前端、类型与 invoke 封装
- 新建项目、实现功能、异步/复杂类型场景
- 配置安全（allowlist）、构建与调试
- **调研目标网站**：克隆/仿站时，自动打开网站并查看页面元素、结构、交互，指导实现
- **验证开发效果**：开发完成后，自动打开本地开发站点或打包前端，验证元素与交互是否按预期完成

## 开发流程建议

1. **先设计后编码**：brainstorming → 架构设计 → 实现
2. **测试驱动**：先写测试再写实现
3. **类型一致**：Rust 模型与 TypeScript 类型严格对应
4. **异步优先**：I/O 用 async/await；CPU 密集用 `spawn_blocking`
5. **安全第一**：配置 allowlist、校验并规范化输入（如路径）

## 项目结构

```
project/
├── src-tauri/                 # Rust 后端
│   ├── src/
│   │   ├── main.rs            # 入口，注册 commands、manage state
│   │   ├── lib.rs             # 模块导出
│   │   ├── commands/          # IPC 命令（按域拆分）
│   │   ├── models/            # 与前端共享的数据结构
│   │   ├── db/                # 数据库访问（Repository 模式）
│   │   ├── state.rs           # AppState
│   │   └── error.rs           # 统一可序列化错误类型
│   ├── Cargo.toml
│   └── tauri.conf.json
├── src/                       # 前端 React + TypeScript
│   ├── components/            # 原子化、职责单一
│   ├── hooks/                 # React Query 等服务器状态
│   ├── services/              # 封装 invoke 的 API 层
│   ├── store/                 # Zustand 等 UI 状态
│   ├── types/                 # 与 Rust 模型一致的 TS 类型
│   └── App.tsx
├── package.json
└── tsconfig.json
```

## IPC 通信

### 前端 → 后端：Command

```typescript
// src/services/xxx.ts
import { invoke } from '@tauri-apps/api/core';

export async function getItems(): Promise<Item[]> {
  return await invoke<Item[]>('get_items');
}
```

```rust
// src-tauri/src/commands/xxx.rs
#[tauri::command]
pub async fn get_items(state: State<'_, AppState>) -> Result<Vec<Item>, AppError> {
    let db = state.db.lock().map_err(|_| AppError::Lock)?;
    Ok(repo::list(&db)?)
}
```

### 后端 → 前端：Event

```rust
use tauri::Manager;
app.emit_all("event-name", payload).ok();
```

```typescript
import { listen } from '@tauri-apps/api/event';
const unlisten = await listen<T>('event-name', (e) => { ... });
// 组件卸载时 unlisten()
```

### 契约一致

- 所有跨边界数据结构在 Rust（`models/`）与 TypeScript（`types/`）中成对定义
- 使用 `serde` 与 TS 接口一致命名与类型（如 `snake_case` ↔ 前端可选 camelCase 转换）

## 后端 (Rust) 要点

- **状态**：`AppState` 在 `main.rs` 中 `manage()`，命令内用 `State<'_, AppState>` 注入
- **错误**：统一错误类型实现 `serde::Serialize`，便于前端展示；用 `thiserror` + `From` 收敛
- **I/O**：异步命令用 `async` + `tokio`；CPU 密集用 `tokio::task::spawn_blocking`
- **路径/输入**：校验类型与范围，必要时 `canonicalize` 并限制在允许目录内

深层 Rust 能力（所有权、并发、性能、测试）参考：Rust 专家级实践（rust-pro、rust-async-patterns）。

## 前端 (React + TypeScript) 要点

- **服务层**：`src/services/` 统一用 `invoke` 调用命令，返回类型与 Rust 一致
- **状态**：UI 状态用 Zustand（`store/`）；服务端数据用 React Query（`hooks/`）
- **错误**：对 `invoke` 的 Promise 做 try/catch，将后端错误转为用户可读信息
- **类型**：严格 TS 配置；泛型与工具类型保证与 Rust 契约一致

复杂类型与架构参考：TypeScript 专家级实践（typescript-pro）。

## 安全

- **allowlist**：在 `tauri.conf.json` 中按需开启 `fs`、`shell` 等，并设置 `scope`
- **命令参数**：校验类型、长度、业务规则；路径类参数做规范化与白名单校验

## 开发与构建

```bash
npm run tauri dev    # 开发
npm run tauri build  # 生产构建
```

## 测试

- **Rust**：单元测试放在 `commands/`、`db/` 等模块内；集成测试可放在 `src-tauri/tests/`
- **前端**：使用 `@tauri-apps/api/mocks` 的 `mockIPC` 模拟命令返回值，再测组件与 hooks
- **调研与验证**：见下一节 agent-browser 流程。

## 调研与验证：agent-browser

在**调研**和**验证**阶段用浏览器自动化（如 agent-browser）可显著提效：克隆网站时先打开目标站调研元素，开发完成后打开本地站点验证效果。

### 核心流程（通用）

1. **打开页面**：`agent-browser --cdp 9222 open <url>`
2. **获取可交互元素**：`agent-browser --cdp 9222 snapshot -i` → 得到 `@e1`、`@e2` 等引用及类型/文案
3. **操作**：用 refs 执行 `click`、`fill`、`select`、`check` 等
4. **页面变化后**：再次 `snapshot -i` 获取新 refs，再继续操作或断言

### 场景一：调研目标网站（克隆/仿站）

目标：了解要仿造的网站的 DOM 结构、可交互元素、文案与交互路径。

```bash
agent-browser --cdp 9222 open https://目标网站.com/页面
agent-browser --cdp 9222 snapshot -i
# 根据输出识别：导航、列表、表单、按钮等对应 @e1 @e2 ...
# 需要文案或结构时：
agent-browser --cdp 9222 get text @e1
agent-browser --cdp 9222 get text body > page.txt
# 可选：截图留档
agent-browser --cdp 9222 screenshot
```

据此确定组件划分、数据结构与 IPC 设计，再进入开发。

### 场景二：验证本地开发效果

目标：开发完成后，在浏览器中打开本地前端，确认元素存在、可点击、表单可填写、流程可走通。

```bash
# 先启动本地前端（另起终端）：npm run dev 或 npm run tauri dev，记下 URL（如 http://localhost:1420
agent-browser --cdp 9222 open http://localhost:1420
agent-browser --cdp 9222 snapshot -i
# 按设计逐项验证：点击侧栏、打开列表、新建任务、填写表单、提交等
agent-browser --cdp 9222 click @e1
agent-browser --cdp 9222 wait --load networkidle
agent-browser --cdp 9222 snapshot -i
agent-browser --cdp 9222 fill @e2 "测试任务"
agent-browser --cdp 9222 click @e3
# 需要时截图或保存页面文本做回归
agent-browser --cdp 9222 screenshot
```

Tauri dev 为 `http://localhost:1420` 。

### 常用命令速查

| 用途     | 命令 |
|----------|------|
| 打开页面 | `agent-browser --cdp 9222 open <url>` |
| 可交互元素 | `agent-browser --cdp 9222 snapshot -i` |
| 点击/填写 | `agent-browser --cdp 9222 click @e1`、`agent-browser --cdp 9222 fill @e2 "内容"` |
| 等待     | `agent-browser --cdp 9222 wait --load networkidle`、`agent-browser --cdp 9222 wait 2000` |
| 取文案   | `agent-browser --cdp 9222 get text @e1`、`agent-browser --cdp 9222 get text body > out.txt` |
| 截图     | `agent-browser --cdp 9222 screenshot`、`agent-browser --cdp 9222 screenshot --full` |

**注意**：页面跳转或 DOM 更新后必须重新 `snapshot -i`，否则旧 ref 会失效。完整命令与语义定位符见 [agent-browser SKILL](file:///Users/akm/Documents/agent-browser/skills/agent-browser/SKILL.md)。

## 项目专用约定（本仓库）

- 前端：React 19、TypeScript、Tailwind CSS、Zustand、React Query
- 后端：Tauri 2、Rust、SQLite（如 rusqlite）
- 更多细节见项目根目录下的 `.agent/rules/Tauri跨平台开发助手.md`

## 参考资源

| 用途           | 参考 |
|----------------|------|
| Tauri 官方     | https://tauri.app/ 、Tauri 2 文档与 API |
| Rust 深度      | rust-pro、rust-async-patterns 等 skill |
| TypeScript 深度| typescript-pro、typescript-advanced-types 等 skill |
| 流程与关键词   | 项目内或个人维护的 tauri-skills-quick-reference |
| 浏览器自动化   | [agent-browser SKILL](file:///Users/akm/Documents/agent-browser/skills/agent-browser/SKILL.md)（完整命令、语义定位符、多会话等） |

## 常见场景速查

| 场景       | 侧重 |
|------------|------|
| 新建项目   | 架构设计 → 目录与 tauri.conf → 第一个 command + 前端 invoke |
| 实现功能   | 先定 Rust 模型与 TS 类型 → 命令与 services → 组件与状态 |
| 调研仿站   | agent-browser --cdp 9222 open 目标站 → snapshot -i → get text / screenshot，再设计组件与数据 |
| 验证效果   | 启动本地前端 → agent-browser --cdp 9222 open localhost → snapshot -i → click/fill 逐项验证 |
| 异步操作   | Rust async + spawn_blocking 区分；前端 debounce/批量 |
| 复杂类型   | 泛型与 trait（Rust）+ 泛型与工具类型（TS）保持一致 |
