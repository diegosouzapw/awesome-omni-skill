---
name: hap-mcp-usage
description: 明道云 HAP MCP 自动化配置技能。**立即触发条件**：用户提到"配置 MCP"、"添加 MCP"、"MCP 配置"、"MCP 连接"、"设置 MCP"、提供包含"hap-mcp-"的配置、提供包含"HAP-Appkey"和"HAP-Sign"的 URL。支持 9 种 AI 工具的自动化配置，配置完成后自动验证连通性。
license: MIT
---

# HAP MCP 自动化配置技能

本技能帮助用户在 9 种 AI 工具中**自动化配置** HAP MCP 服务器，并验证连通性。

## 🎯 技能触发场景

当用户说以下任何内容时，**立即使用本技能**：
- "配置这个 MCP"
- "添加 MCP 服务器"
- "帮我配置 HAP MCP"
- "设置 MCP 连接"
- 提供了包含 `hap-mcp-` 的配置信息
- 提供了包含 `HAP-Appkey` 和 `HAP-Sign` 的 URL

## 关于 HAP MCP

HAP 提供两种不同类型的 MCP，**作用和使用场景完全不同**：

### 🔷 类型 1: HAP API 文档 MCP

**作用**: 让 AI 读懂 HAP 接口文档（只读，不执行操作）

**配置格式**（官方固定）:
```json
{
  "mcpServers": {
    "应用 API - API 文档": {
      "command": "npx",
      "args": ["-y", "apifox-mcp-server@latest", "--site-id=5442569"]
    }
  }
}
```

**适用场景**:
- 📖 查询 HAP API 文档
- 🛠️ 学习接口结构和参数
- 📝 生成 API 调用代码

### 🔶 类型 2: HAP 应用执行 MCP

**作用**: 让 AI 执行 HAP 应用接口（可操作真实数据）

**配置格式**（应用专属）:
```json
{
  "mcpServers": {
    "hap-mcp-应用名": {
      "url": "https://api.mingdao.com/mcp?HAP-Appkey=xxx&HAP-Sign=xxx",
      "type": "streamable"
    }
  }
}
```

**⚠️ 重要**: HAP MCP 的 `"type": "streamable"` 配置**取决于 IDE 平台**：
- **需要指定**: Cursor, TRAE, GitHub Copilot, Antigravity, OpenCode, Windsurf 等
- **不需要指定**: Claude Code（命令行自动处理）
- **建议**: 如果不确定，建议添加 `"type": "streamable"`，兼容性更好

**适用场景**:
- 🔍 查询应用真实数据
- ✏️ 创建/修改/删除记录
- 📊 数据统计和分析
- 🔄 执行工作流

---

## 🤖 AI 执行步骤（自动化配置）

当用户提供 MCP 配置时，AI 必须按以下步骤自动化完成配置：

### Step 1: 识别当前 AI 工具平台

首先确定用户当前使用的是哪个 AI 工具（从以下 9 个平台中识别）：

1. **Claude Code** - Anthropic 官方 CLI
2. **TRAE** - 标准化 `.trae/` 目录
3. **Cursor** - 最流行的 AI 编辑器
4. **GitHub Copilot** - GitHub 官方工具
5. **Google Antigravity** - Google 实验工具
6. **OpenCode** - 开源 AI 工具
7. **Windsurf** - Codeium 出品
8. **Gemini CLI** - Google Gemini 命令行
9. **Codex** - OpenAI 编程助手

**识别方法**:
- 如果用户明确提到工具名称，使用该工具
- 否则，询问用户："您当前使用的是哪个 AI 工具？（Claude Code / Cursor / TRAE / 其他）"

### Step 2: 解析 MCP 配置信息

从用户提供的配置中提取：
- **服务器名称**: 如 `hap-mcp-客户管理`
- **URL**: 包含 `HAP-Appkey` 和 `HAP-Sign` 的完整 URL
- **MCP 类型**: 根据配置格式判断是 API 文档 MCP 还是应用执行 MCP

**重要**: 如果服务器名称包含中文，需要为 Codex 平台生成英文名称：
- **原始名称**: `hap-mcp-客户管理` → 保留（用于其他平台）
- **英文名称**: `hap-mcp-customer-management` → 用于 Codex
- **转换规则**: 将中文部分翻译成英文拼音或英文单词，保持 kebab-case 格式

### Step 3: 根据平台自动化配置

根据识别到的平台，执行对应的配置步骤：

#### 🔧 Claude Code

**配置方式**: 命令行
```bash
# 添加 HTTP MCP 服务器
claude mcp add <server-name> --url "<server-url>"

# 示例
claude mcp add hap-mcp-客户管理 --url "https://api.mingdao.com/mcp?HAP-Appkey=xxx&HAP-Sign=xxx"
```

**验证命令**:
```bash
claude mcp list
```

#### 🔧 Cursor

**MCP 配置文件**: `.cursor/mcp.json`（项目级，推荐）或 `~/.cursor/mcp.json`（全局）
**Skills 安装位置**: `~/.cursor/skills/` 或 `~/.cursor/skills-cursor/`

**自动化步骤**:
1. 检查并创建 `.cursor` 目录（如果不存在）
2. **读取现有配置文件**（如果存在）- **重要：保留所有已有配置**
3. **增量添加或更新** MCP 配置：
   - 如果 MCP 名称不存在 → 新增
   - 如果 MCP 名称已存在 → 更新（覆盖）
   - 其他 MCP → 保留（不删除）
4. 保存到 `.cursor/mcp.json`
5. **启用 MCP 服务器**（配置后自动生效，无需重启）

**配置格式**:
```json
{
  "mcpServers": {
    // 保留用户已有的 MCP 配置
    "existing-mcp-server": {
      "url": "https://example.com/mcp"
    },
    // 新增 HAP MCP 配置（必须指定 type: streamable）
    "hap-mcp-应用名": {
      "url": "https://api.mingdao.com/mcp?HAP-Appkey=xxx&HAP-Sign=xxx",
      "type": "streamable"
    }
  }
}
```

**⚠️ 关键原则**:
- ✅ **增量更新**: 名称不存在则新增，已存在则更新，其他保留
- ❌ **禁止覆盖**: 不要清空或删除用户已有的 MCP 服务器
- ✅ **streamable 类型**: 根据平台决定是否需要（建议添加以提高兼容性）
- ✅ **启用并验证**: 配置后立即启用并验证连通性（无需重启）

**注意**: `"type": "streamable"` 是否需要取决于 IDE 平台，建议添加以提高兼容性

#### 🔧 TRAE

**配置文件**: `.trae/mcp.json`（项目级）或 `~/.trae/mcp.json`（全局）

**自动化步骤**:
1. 检查并创建 `.trae` 目录
2. 读取或创建 `mcp.json`
3. 添加 MCP 配置（格式同 Cursor）
4. 保存文件

#### 🔧 GitHub Copilot

**配置文件**: `~/.copilot/mcp-config.json`

**自动化步骤**:
1. 检查并创建 `~/.copilot` 目录
2. 读取或创建 `mcp-config.json` 文件
3. 添加 MCP 配置
4. 保存文件

**配置格式**: 同 Cursor

**⚠️ 注意**: GitHub Copilot 使用 `mcp-config.json` 而不是 `mcp.json`

#### 🔧 Google Antigravity

**配置文件**: `~/.gemini/antigravity/mcp_config.json`

**自动化步骤**:
1. 检查并创建目录 `~/.gemini/antigravity/`
2. 读取或创建 `mcp_config.json` 文件
3. 在 `mcpServers` 部分添加配置
4. 保存文件

**配置格式**: 同 Cursor

**⚠️ 注意**: Antigravity 使用 `mcp_config.json` 而不是 `config.json`

#### 🔧 OpenCode

**配置文件**: `~/.config/opencode/opencode.json`

**自动化步骤**:
1. 检查并创建目录 `~/.config/opencode/`
2. 读取或创建 `opencode.json` 文件
3. 在 `mcp` 部分添加配置
4. 保存文件

**配置格式**: 同 Cursor

**⚠️ 注意**: OpenCode 使用 `opencode.json` 而不是 `mcp.json`

#### 🔧 Windsurf

**配置文件**: `~/.codeium/windsurf/mcp_config.json`

**自动化步骤**:
1. 检查并创建目录 `~/.codeium/windsurf/`
2. 读取或创建 `mcp_config.json` 文件
3. 添加 MCP 配置
4. 保存文件

**配置格式**: 同 Cursor

**⚠️ 注意**: Windsurf 使用 `mcp_config.json` 而不是 `mcp.json`

#### 🔧 Gemini CLI

**配置文件**: `settings.json`（位置由 Gemini CLI 管理）

**配置方式**: 命令行或配置文件

**命令行方式**:
```bash
gemini mcp add <server-name> --url "<server-url>"
```

**配置文件方式**:
- 使用 `/mcp` 命令打开配置文件
- 在 `mcpServers` 中添加配置

**⚠️ 注意**: Gemini CLI 使用 `settings.json`，具体路径由工具管理

**参考文档**: https://github.com/google-gemini/gemini-cli/blob/main/docs/tools/mcp-server.md

#### 🔧 OpenAI Codex

**配置文件**: `~/.codex/config.toml`

**⚠️ 重要限制**: Codex 的 TOML 格式**不支持中文 key 名称**

**自动化步骤**:
1. **中文名称转换**: 如果服务器名称包含中文，转换为英文
   - 示例: `hap-mcp-客户管理` → `hap-mcp-customer-management`
   - 规则: 中文翻译成英文单词或拼音，使用 kebab-case 格式
2. 读取现有 `config.toml`
3. 添加 MCP 服务器配置（使用英文名称）
4. 保存文件

**配置格式** (TOML):
```toml
# ✅ 正确 - 使用英文名称
[mcp_servers."hap-mcp-customer-management"]
url = "https://api.mingdao.com/mcp?HAP-Appkey=xxx&HAP-Sign=xxx"

# ❌ 错误 - 中文名称不支持
# [mcp_servers."hap-mcp-客户管理"]
# url = "https://api.mingdao.com/mcp?HAP-Appkey=xxx&HAP-Sign=xxx"
```

**名称转换示例**:
- `hap-mcp-客户管理` → `hap-mcp-customer-management`
- `hap-mcp-订单系统` → `hap-mcp-order-system`
- `hap-mcp-人力资源` → `hap-mcp-hr` 或 `hap-mcp-human-resources`
- `hap-mcp-财务管理` → `hap-mcp-finance-management`

### Step 4: 启用并验证 MCP 连通性

**重要**: 配置完成后，必须**立即启用 MCP 服务器并验证**是否可以正常连接。

#### 4.1 启用 MCP 服务器

**配置后自动生效**，无需重启工具：
- 配置文件保存后，MCP 服务器会自动加载
- 可以立即开始使用 MCP 工具

**注意**: 如果 MCP 未自动启用，可能需要手动刷新或重新加载配置

#### 4.2 验证连通性

**验证方法**:
1. 调用 MCP 工具：`get_app_info`（获取应用信息）
2. 检查返回结果：
   - ✅ 成功：返回应用信息（应用名称、工作表列表等）
   - ❌ 失败：返回错误信息（鉴权失败、网络错误等）

**验证示例**:
```javascript
// 调用 MCP 工具验证连通性
const result = await mcpClient.call('get_app_info');

if (result.success) {
  console.log('✅ MCP 连接成功！');
  console.log('应用名称：', result.appName);
  console.log('工作表数量：', result.worksheets.length);
} else {
  console.error('❌ MCP 连接失败：', result.error);
}
```

#### 4.3 连接失败诊断

如果 MCP 连接失败，按以下步骤诊断：

**诊断清单**:
1. **检查配置文件**:
   - 配置文件路径是否正确
   - JSON/TOML 格式是否正确
   - `"type": "streamable"` 是否已添加（HAP MCP 必需）
2. **检查鉴权信息**:
   - `HAP-Appkey` 和 `HAP-Sign` 是否正确
   - URL 是否完整且格式正确
3. **检查应用状态**:
   - 应用是否已启用 MCP 功能
   - 应用是否可以正常访问
4. **检查网络连接**: 确认可以访问 `api.mingdao.com` 或 `www.nocoly.com`
5. **检查 MCP 服务器状态**:
   - MCP 服务器是否已启用
   - 是否需要手动刷新配置

**常见错误及解决方案**:

| 错误类型 | 可能原因 | 解决方案 |
|---------|---------|---------|
| 鉴权失败 | Appkey/Sign 错误 | 重新从 HAP 获取正确的鉴权信息 |
| 连接超时 | 网络问题 | 检查网络连接，尝试访问 api.mingdao.com |
| MCP 未找到 | 配置未生效 | 检查配置文件路径和格式，手动刷新配置 |
| 配置格式错误 | JSON/TOML 语法错误 | 检查并修正配置文件格式 |
| 缺少 streamable | 未指定 type | 添加 `"type": "streamable"` 到配置中 |
| 中文 key 错误 | Codex 使用了中文名称 | 将服务器名称转换为英文 |

**提供给用户的诊断步骤**:
```
❌ MCP 连接失败，请按以下步骤排查：

1. 检查配置文件
   → 位置: [配置文件路径]
   → 格式: [JSON/TOML]
   → 打开文件检查是否有语法错误
   → 确认已添加 "type": "streamable"（HAP MCP 必需）

2. 检查鉴权信息
   → HAP-Appkey: [显示前5位]...
   → HAP-Sign: [显示前5位]...
   → 如果不确定，请重新从 HAP 应用获取

3. 验证 URL 格式
   → 确认 URL 完整且无多余空格
   → 格式: https://api.mingdao.com/mcp?HAP-Appkey=xxx&HAP-Sign=xxx
   → 或: https://www.nocoly.com/mcp?HAP-Appkey=xxx&HAP-Sign=xxx

4. 测试网络连接
   → 在浏览器访问: https://api.mingdao.com
   → 确认网络可以访问明道云 API

5. 检查应用设置
   → 登录 HAP 应用
   → 确认 MCP 功能已启用
   → 检查 API 权限设置

6. 手动刷新配置
   → 如果配置未自动生效，尝试手动刷新
   → 或重启 AI 工具使配置重新加载

如果以上步骤都无法解决，请提供完整错误信息以便进一步诊断。
```

### Step 5: 向用户报告结果

配置完成后，向用户报告：

**成功时**:
```
✅ MCP 配置成功！

📋 配置信息：
- 平台：Cursor
- 服务器名称：hap-mcp-客户管理
- 配置文件：.cursor/mcp.json
- 已保留其他 MCP 配置

✅ 连通性验证通过：
- 应用名称：客户管理系统
- 工作表数量：5 个

💡 下一步：
- MCP 已启用并可正常使用
- 现在可以使用 MCP 工具操作数据了
```

**失败时**:
```
❌ MCP 配置已保存，但连通性验证失败

📋 配置信息：
- 平台：Cursor
- 配置文件：.cursor/mcp.json
- 已保留其他 MCP 配置

❌ 连接错误：
- 错误类型：鉴权失败
- 错误信息：Invalid HAP-Appkey or HAP-Sign

🔧 诊断步骤：

1️⃣ 检查配置文件
   → 打开文件: .cursor/mcp.json
   → 检查 JSON 格式是否正确
   → 确认已添加 "type": "streamable"
   → 确认 URL 完整且无多余空格

2️⃣ 检查鉴权信息
   → HAP-Appkey: abc12... (前5位)
   → HAP-Sign: xyz78... (前5位)
   → 如果不确定，请从 HAP 应用重新获取

3️⃣ 验证 URL 格式
   → 格式: https://api.mingdao.com/mcp?HAP-Appkey=xxx&HAP-Sign=xxx
   → 或: https://www.nocoly.com/mcp?HAP-Appkey=xxx&HAP-Sign=xxx

4️⃣ 测试网络
   → 在浏览器访问: https://api.mingdao.com
   → 确认网络可以连接明道云

5️⃣ 检查应用设置
   → 登录 HAP 应用后台
   → 确认 MCP 功能已启用
   → 检查 API 访问权限

6️⃣ 手动刷新配置
   → 如果配置未自动生效，尝试手动刷新
   → 或重启 AI 工具使配置重新加载

📞 需要帮助？
- 提供完整错误信息以便进一步诊断
- 或访问 HAP 帮助中心查看 MCP 配置文档
```

---

## 📋 MCP 配置文件和 Skills 安装位置速查表

### MCP 配置文件位置

| 平台 | 项目级配置 | 全局配置 | 格式 | Skills 位置 |
|------|-----------|---------|------|------------|
| **Claude Code** | - | 命令行配置 | 命令 | `~/.claude/skills/` |
| **Cursor** | `.cursor/mcp.json` | `~/.cursor/mcp.json` | JSON | `~/.cursor/skills/` 或 `~/.cursor/skills-cursor/` |
| **TRAE** | `.trae/mcp.json` | `~/.trae/mcp.json` | JSON | `~/.trae/skills/` |
| **GitHub Copilot** | - | `~/.copilot/mcp-config.json` | JSON | `~/.copilot/skills/` |
| **Antigravity** | - | `~/.gemini/antigravity/mcp_config.json` | JSON | `~/.gemini/antigravity/skills/` |
| **OpenCode** | - | `~/.config/opencode/opencode.json` | JSON | `~/.config/opencode/skills/` |
| **Windsurf** | - | `~/.codeium/windsurf/mcp_config.json` | JSON | `~/.codeium/windsurf/skills/` |
| **Gemini CLI** | - | `settings.json` (工具管理) | JSON | 工具管理 |
| **Codex** | - | `~/.codex/config.toml` | TOML | `~/.codex/skills/` |

### 重要说明

**MCP 配置策略**:
- 支持项目级配置的平台（Cursor, TRAE）：优先使用项目级
- 其他平台：使用全局配置
- **streamable 类型**: 根据平台决定是否需要，建议添加以提高兼容性

**Skills 安装**:
- Skills 通常安装在对应工具的 `skills/` 目录下
- Cursor 可能有 `skills/` 或 `skills-cursor/` 两个目录
- 安装 skills 后需要重启工具使其生效

**配置后必须操作**:
1. ✅ 保存配置文件
2. ✅ **启用 MCP 服务器**（配置后自动生效，无需重启）
3. ✅ **立即验证连通性**（调用 get_app_info 测试）
4. ✅ 如果失败，提供详细诊断步骤给用户

---

## ⚠️ 重要注意事项

### 配置时必须做到

- ✅ **自动化执行**: 直接帮用户配置，不要只告诉步骤
- ✅ **平台识别**: 准确识别用户使用的工具
- ✅ **streamable 类型**: 根据平台决定是否需要（建议添加以提高兼容性）
- ✅ **中文转换**: Codex 平台必须将中文服务器名称转换为英文
- ✅ **增量更新**: **名称不存在则新增，已存在则更新，其他保留**
- ✅ **格式检查**: 确保 JSON/TOML 格式正确
- ✅ **启用配置**: 配置后立即启用（无需重启）
- ✅ **验证连通**: 配置后**立即验证** MCP 是否可用
- ✅ **失败诊断**: 连接失败时提供详细诊断步骤和解决方案
- ✅ **错误处理**: 如果配置或验证失败，提供清晰的错误信息

### 配置时不要做

- ❌ **不要覆盖配置**: 禁止清空或删除用户已有的 MCP 服务器（致命错误！）
- ❌ **增量更新原则**: 名称不存在则新增，已存在则更新，其他必须保留
- ❌ **streamable 类型**: 根据平台决定是否需要，建议添加以提高兼容性
- ❌ 不要只告诉用户如何配置，要直接执行
- ❌ 不要跳过连通性验证步骤
- ❌ 不要在验证失败时直接放弃，要提供诊断步骤
- ❌ 不要在 Codex 中使用中文服务器名称（必须转换为英文）
- ❌ 不要使用错误的配置格式（根据平台使用正确的 type）

### 配置优先级

1. **项目级配置** - 如果平台支持（Cursor, TRAE）
2. **全局配置** - 其他平台或用户明确要求

### 安全提示

- 🔐 提醒用户保护 `HAP-Appkey` 和 `HAP-Sign`
- 🔐 不要将配置文件提交到 Git（建议添加到 `.gitignore`）
- 🔐 定期检查和更新鉴权信息

---

## 📚 配置示例

### 示例 1: Cursor 项目级配置

**用户提供**:
```json
{"hap-mcp-客户管理":{"url":"https://api.mingdao.com/mcp?HAP-Appkey=abc123&HAP-Sign=xyz789"}}
```

**AI 执行**:
1. 创建 `.cursor` 目录（如果不存在）
2. **读取现有 `.cursor/mcp.json` 并保留所有已有配置**
3. **增量添加**新的 HAP MCP 配置:
```json
{
  "mcpServers": {
    // 保留所有已有的 MCP 配置
    "existing-server-1": { "url": "..." },
    "existing-server-2": { "url": "..." },
    // 新增 HAP MCP（必须指定 type: streamable）
    "hap-mcp-客户管理": {
      "url": "https://api.mingdao.com/mcp?HAP-Appkey=abc123&HAP-Sign=xyz789",
      "type": "streamable"
    }
  }
}
```
4. 保存文件
5. **启用 MCP 服务器**（配置后自动生效，无需重启）
6. **立即验证连通性**（调用 get_app_info 测试）
7. 如果失败，提供详细的诊断步骤
8. 报告结果给用户

### 示例 2: Claude Code 命令行配置

**用户提供**: 同上

**AI 执行**:
```bash
claude mcp add hap-mcp-客户管理 --url "https://api.mingdao.com/mcp?HAP-Appkey=abc123&HAP-Sign=xyz789"
```

验证:
```bash
claude mcp list
```

### 示例 3: Codex TOML 配置（中文名称转换）

**用户提供**:
```json
{"hap-mcp-客户管理":{"url":"https://api.mingdao.com/mcp?HAP-Appkey=abc123&HAP-Sign=xyz789"}}
```

**AI 执行**:
1. 识别服务器名称包含中文: `hap-mcp-客户管理`
2. 转换为英文名称: `hap-mcp-customer-management`
3. 编辑 `~/.codex/config.toml`:
```toml
[mcp_servers."hap-mcp-customer-management"]
url = "https://api.mingdao.com/mcp?HAP-Appkey=abc123&HAP-Sign=xyz789"
```
4. 向用户说明名称转换:
```
✅ MCP 配置成功！

📋 配置信息：
- 平台：Codex
- 原始名称：hap-mcp-客户管理
- 转换后名称：hap-mcp-customer-management（Codex 不支持中文 key）
- 配置文件：~/.codex/config.toml

💡 说明：Codex 的 TOML 格式不支持中文 key 名称，已自动转换为英文。
```

---

## 🎯 总结

本技能的核心价值：
1. **自动化** - 用户只需提供配置，AI 自动完成所有步骤
2. **全平台支持** - 支持 9 种主流 AI 工具
3. **增量更新** - 只添加新配置，保留用户所有已有 MCP
4. **启用验证** - 配置后启用并验证连通性，确保可用
5. **失败诊断** - 连接失败时提供详细诊断和解决方案
6. **错误处理** - 清晰的错误信息和用户指导

**关键原则**:
- ✅ **增量更新**：名称不存在则新增，已存在则更新，其他保留
- ✅ 配置后**立即启用并验证**（无需重启）
- ✅ 失败时提供诊断步骤，不直接放弃
- ✅ Codex 平台自动转换中文名称
- ✅ `"type": "streamable"` 根据平台决定，建议添加以提高兼容性

**记住**: 用户说"配置 MCP"时，不要问"需要我帮您配置吗？"，而是立即执行配置流程！
