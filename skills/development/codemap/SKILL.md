---
name: codemap
description: 代码流程分析与可视化工具，将复杂代码执行流转化为清晰的可视化图谱。支持本地文件分析、历史版本管理、导入导出等功能。
---

# CodeMap

高级代码架构分析和可视化工具，将用户的查询和代码库上下文转化为结构化的 CodeMap JSON，并提供多种可视化表达（文本图、Mermaid 图、Markdown 指南）。

## 执行环境

| 路径类型       | 路径                                  | 基准目录 |
| -------------- | ------------------------------------- | -------- |
| **技能目录**   | `~/.pi/agent/skills/codemap/`         | 固定位置 |
| **SKILL 文件** | `~/.pi/agent/skills/codemap/SKILL.md` | 技能目录 |
| **客户端目录** | `~/.pi/agent/skills/codemap/client/`  | 技能目录 |

## 功能特性

### 核心功能

- **代码流程分析**：自动识别 Controller、Service、Mapper 等关键代码节点
- **可视化输出**：生成三种可视化形式（文本调用图、Mermaid 流程图、Markdown 指南）
- **本地处理**：所有代码分析在本地完成，保护隐私

### 高级功能

- **历史版本管理**：保存和管理多个 CodeMap 版本，支持版本对比
- **目录树浏览**：交互式文件目录树，支持多文件/文件夹选择
- **导入导出**：支持 JSON、Markdown、自包含 HTML 格式
- **分享功能**：导出独立 HTML 文件，包含所有可视化和数据

## 客户端架构

### 技术栈

- **后端**：Rust + Tauri
- **前端**：Web（HTML + CSS + JavaScript）
- **存储**：基于文件系统（JSON 格式，存储在 `docs/.codemap/` 目录）
- **可视化库**：mermaid.js、marked.js

### 目录结构

```
~/.pi/agent/skills/codemap/
├── SKILL.md                    # 技能文档
├── client/                     # Tauri 客户端
│   ├── src-tauri/             # Rust 后端
│   │   ├── src/
│   │   │   ├── main.rs        # Tauri 入口
│   │   │   ├── commands.rs    # Tauri 命令
│   │   │   ├── analyzer.rs    # 代码分析逻辑
│   │   │   ├── codemap.rs     # CodeMap 生成
│   │   │   └── storage.rs     # 文件系统存储
│   │   └── Cargo.toml
│   ├── src/                   # Web 前端
│   │   ├── components/        # 组件
│   │   ├── styles/            # 样式
│   │   ├── app.js             # 主应用
│   │   └── index.html
│   └── package.json
└── README.md
```

## 使用方式

### 方式 1：通过 Tauri 客户端（推荐）

```bash
# 启动客户端（开发模式）
cd ~/.pi/agent/skills/codemap/client
npm run tauri dev

# 构建生产版本
npm run tauri build
```

### 方式 2：集成到 AI 工作流

在 AI 对话中调用 codemap 技能：

```
用户：请分析"用户登录全流程"的代码执行路径

AI（调用 codemap）：
1. 读取代码库上下文
2. 识别关键节点（Controller、Service、Mapper）
3. 生成 CodeMap JSON
4. 返回可视化结果
```

## CodeMap JSON Schema

```json
{
  "schemaVersion": 1,
  "title": "流程标题",
  "description": "流程的简要概述",
  "mermaidDiagram": "全局视角的 Mermaid graph TB 流程图代码",
  "traces": [
    {
      "id": "序号 (e.g., 1, 2)",
      "title": "步骤标题",
      "description": "步骤简述",
      "locations": [
        {
          "id": "节点ID (e.g., 1a, 1b)",
          "path": "文件绝对路径",
          "lineNumber": 整数行号,
          "lineContent": "关键代码行内容",
          "title": "节点行为标题",
          "description": "在该行发生了什么"
        }
      ],
      "traceTextDiagram": "基于文本的树状调用图",
      "traceGuide": "Markdown 格式的详细指南"
    }
  ]
}
```

## Tauri 命令 API

### `analyze_code`

分析代码并生成 CodeMap。

```rust
#[tauri::command]
async fn analyze_code(
    files: Vec<String>,      // 文件路径列表
    query: String,           // 用户查询
    project_root: String,    // 项目根目录
) -> Result<CodeMap, String>
```

### `save_codemap`

保存 CodeMap 到数据库。

```rust
#[tauri::command]
async fn save_codemap(
    codemap: CodeMap,        // CodeMap 数据
    tags: Vec<String>,       // 标签
    notes: String,           // 备注
) -> Result<u64, String>    // 返回 ID
```

### `list_codemaps`

列出所有保存的 CodeMap。

```rust
#[tauri::command]
async fn list_codemaps() -> Result<Vec<CodeMapMeta>, String>
```

### `load_codemap`

加载指定 ID 的 CodeMap。

```rust
#[tauri::command]
async fn load_codemap(id: u64) -> Result<CodeMap, String>
```

### `delete_codemap`

删除指定 ID 的 CodeMap。

```rust
#[tauri::command]
async fn delete_codemap(id: u64) -> Result<(), String>
```

### `export_codemap`

导出 CodeMap 为指定格式。

```rust
#[tauri::command]
async fn export_codemap(
    id: u64,
    format: String,          // "json" | "markdown" | "html"
    output_path: String,
) -> Result<(), String>
```

### `import_codemap`

导入 CodeMap 从文件。

```rust
#[tauri::command]
async fn import_codemap(
    file_path: String,       // JSON 或 Markdown 文件路径
) -> Result<CodeMap, String>
```

## 前端界面设计

### 主界面布局

```
┌─────────────────────────────────────────────────────┐
│  CodeMap - 代码流程可视化工具           [设置] [帮助] │
├──────────────┬──────────────────────────────────────┤
│              │                                       │
│  📁 目录树   │  查询输入                            │
│              │  ┌────────────────────────────────┐  │
│  ├─ src/     │  │ 请输入要分析的流程...          │  │
│  │  ├─ ctrl/ │  └────────────────────────────────┘  │
│  │  ├─ svc/  │                                       │
│  │  └─ dao/  │  [分析] [清除选择]                   │
│  └─ tests/   │                                       │
│              │  ┌────────────────────────────────┐  │
│  历史版本    │  │                                │  │
│  📋 版本列表 │  │                                │  │
│  ├─ 登录流程 │  │     可视化展示区               │  │
│  ├─ 订单流程 │  │     (Mermaid / 文本图 / 指南) │  │
│  └─ 支付流程 │  │                                │  │
│              │  │                                │  │
│              │  └────────────────────────────────┘  │
│              │                                       │
│              │  [导出 JSON] [导出 Markdown] [分享]  │
└──────────────┴──────────────────────────────────────┘
```

### 组件说明

#### 1. 目录树组件（DirectoryTree）

- 显示项目目录结构
- 支持多文件选择（复选框）
- 支持文件夹全选
- 显示文件类型图标

#### 2. 查询输入组件（QueryInput）

- 文本输入框
- 支持历史查询记录
- 提供示例查询提示

#### 3. 可视化展示区（VisualizationPanel）

- 三个 Tab：Mermaid 图、文本调用图、Markdown 指南
- 支持缩放和全屏
- 支持节点点击查看代码

#### 4. 历史版本列表（HistorySidebar）

- 显示所有保存的 CodeMap
- 支持搜索和过滤
- 显示创建时间和标签
- 支持删除和重命名

#### 5. 导出组件（ExportDialog）

- 格式选择：JSON、Markdown、HTML
- 文件路径选择
- 导出进度显示

## 数据存储

### 文件系统结构

CodeMap 使用基于文件系统的存储方式，所有数据存储在项目的 `docs/.codemap/` 目录中。

```
项目根目录/
├── docs/
│   ├── .codemap/              # CodeMap 数据目录
│   │   ├── index.json         # CodeMap 索引（元数据）
│   │   ├── codemaps/          # CodeMap 数据文件
│   │   │   ├── 20250106-001-login-flow.json
│   │   │   ├── 20250106-002-order-process.json
│   │   │   └── ...
│   │   ├── exports/           # 导出文件
│   │   │   ├── login-flow.html
│   │   │   ├── order-process.md
│   │   │   └── ...
│   │   └── cache/             # 缓存目录（可选）
│   ├── issues/                # Workhub Issues
│   └── pr/                    # Workhub PRs
└── src/                       # 源代码
```

### index.json 结构

```json
{
  "version": 1,
  "projectRoot": "/path/to/project",
  "codemaps": [
    {
      "id": "20250106-001",
      "filename": "20250106-001-login-flow.json",
      "title": "用户登录流程",
      "description": "从前端登录请求到后端验证的完整流程",
      "query": "用户登录全流程",
      "createdAt": "2025-01-06T10:30:00Z",
      "updatedAt": "2025-01-06T15:20:00Z",
      "tags": ["v1.0", "生产环境"],
      "note": "包含异常处理逻辑"
    }
  ]
}
```

### CodeMap 文件结构

每个 CodeMap 保存为独立的 JSON 文件：

```json
{
  "schemaVersion": 1,
  "metadata": {
    "id": "20250106-001",
    "title": "用户登录流程",
    "description": "从前端登录请求到后端验证的完整流程",
    "query": "用户登录全流程",
    "projectRoot": "/path/to/project",
    "createdAt": "2025-01-06T10:30:00Z",
    "updatedAt": "2025-01-06T15:20:00Z",
    "tags": ["v1.0", "生产环境"],
    "note": "包含异常处理逻辑"
  },
  "title": "用户登录流程",
  "description": "从前端登录请求到后端验证的完整流程",
  "mermaidDiagram": "graph TB\n  A[前端提交] --> B[Controller.login]\n  B --> C[Service.validateUser]",
  "traces": [...]
}
```

## 代码分析逻辑

### 关键节点识别

#### Controller 层

- 特征：类名包含 `Controller`、`Api`、`Endpoint`
- 关键方法：`@RequestMapping`、`@GetMapping`、`@PostMapping` 等

#### Service 层

- 特征：类名包含 `Service`、`Manager`、`Handler`
- 关键逻辑：业务判断、数据处理

#### Mapper/DAO 层

- 特征：类名包含 `Mapper`、`Dao`、`Repository`
- 关键操作：数据库查询、插入、更新

#### 分支判断

- 特征：`if`、`switch`、`try-catch`、三元运算符
- 记录条件表达式

### 文本调用图生成

```
流程名称
├── Phase 1: 前端提交
│   ├── LoginController.login() < -- 1a
│   │   └── 调用 UserService.validateUser() < -- 1b
│   └── 返回登录结果 < -- 1c
└── Phase 2: 后端处理
    ├── UserService.validateUser() < -- 2a
    │   ├── 检查用户存在性 < -- 2b
    │   ├── 验证密码 < -- 2c
    │   └── 生成 Token < -- 2d
    └── UserMapper.selectById() < -- 2e
```

## 示例使用

### 示例 1：分析登录流程

**输入**：

- 查询：`用户登录全流程`
- 文件：`src/controller/LoginController.java`, `src/service/UserService.java`, `src/mapper/UserMapper.java`

**输出**：

```json
{
  "schemaVersion": 1,
  "title": "用户登录全流程",
  "description": "从前端登录请求到后端验证和Token生成的完整流程",
  "mermaidDiagram": "graph TB\n  A[前端提交] --> B[Controller.login]\n  B --> C[Service.validateUser]\n  C --> D[Mapper.selectById]\n  D --> C\n  C --> E[生成Token]\n  E --> B\n  B --> A",
  "traces": [...]
}
```

### 示例 2：历史版本管理

1. 生成多个 CodeMap 版本
2. 在历史版本列表中查看
3. 点击版本加载到可视化区域
4. 对比不同版本的差异

### 示例 3：分享功能

1. 选择要分享的 CodeMap
2. 点击"分享"按钮
3. 选择"导出 HTML"
4. 生成独立的 HTML 文件，包含所有可视化
5. 发送给同事，直接在浏览器中打开查看

## 最佳实践

### 代码选择

- 选择完整的业务流程相关文件
- 包含 Controller、Service、Mapper 层
- 避免选择过多无关文件

### 查询编写

- 使用清晰的业务术语：`用户登录流程`、`订单创建流程`
- 可以包含特定关注点：`支付流程 - 异常处理`
- 中英文均可，系统会自动识别

### 版本管理

- 为重要的 CodeMap 添加标签：`v1.0`、`生产环境`
- 添加备注说明版本变更点
- 定期清理过期的历史版本

### 导出格式

- **JSON**：用于程序化处理和二次开发
- **Markdown**：用于文档归档和版本控制
- **HTML**：用于分享和演示

## 扩展计划

- [ ] 支持更多编程语言（Python、Go、TypeScript）
- [ ] 添加代码复杂度分析
- [ ] 集成 AI 智能建议
- [ ] 支持团队协作（云端同步）
- [ ] 添加插件系统

## 故障排查

### 常见问题

**Q：Mermaid 图表不显示？**
A：检查网络连接，mermaid.js 需要从 CDN 加载。可配置本地离线版本。

**Q：代码分析不准确？**
A：确保选择的文件包含完整的业务流程，代码需要符合常见的命名规范。

**Q：历史版本丢失？**
A：检查 SQLite 数据库文件权限，确保应用有读写权限。

## 技术支持

- 技能目录：`~/.pi/agent/skills/codemap/`
- 客户端目录：`~/.pi/agent/skills/codemap/client/`
- 数据存储：`项目根目录/docs/.codemap/`
