---
name: page-annotator
description: AI驱动的网页标注工具，支持高亮元素和添加文字批注。智能防重复、自动滚动、碰撞检测。兼容 GitHub 等严格 CSP 网站。适用场景：(1) 标记网页元素进行讲解 (2) 添加文字批注和注释 (3) 代码审查和设计评审 (4) 教学演示和用户引导 (5) Bug 报告和问题标记
---

## 使用说明

在当前网页上创建可视化标注层，支持高亮和文字批注两种标注方式，所有标注可一键清除。

**兼容性**: 通过浏览器扩展权限绕过 CSP 限制，可在 GitHub、Google 等严格安全策略的网站上正常使用。

**⚠️ 防止重复标注的关键：**
1. **强制使用长文本匹配**：必须使用完整句子（15+ 字符），避免短词匹配多处
   - ❌ 错误：`"text": "YAML"` → 匹配 12 处（侧边栏、正文、小标题）
   - ✅ 正确：`"text": "Beyond the markdown content, you can configure skill behavior using YAML"` → 唯一匹配
2. **添加结构限制**：组合使用 `selector` 限定范围（如 `"selector": "main", "text": "长文本"`）
3. **默认只标注一个**：系统默认 `maxMatches: 1`，只标注第一个匹配项
4. **自动排除干扰区域**：系统自动跳过导航栏、侧边栏、页脚等区域

**💡 长文本匹配的优势：**
- 精确定位：长文本在页面中通常是唯一的
- 避免歧义：不会匹配到导航、标题、侧边栏等重复内容
- 提高准确性：直接定位到目标段落或句子

### 核心功能

1. **高亮元素** - 为指定元素添加高亮效果和标签
2. **文字批注** - 在元素旁添加批注气泡（支持气泡/便签/行内三种样式），自动在原文下方添加下划线标识
3. **智能匹配** - 支持 CSS 选择器和文本搜索双重匹配
4. **自动滚动** - 自动滚动到标注位置，避免标注"消失"
5. **碰撞检测** - 自动调整位置避免批注重叠
6. **防遮挡原文** - 批注始终显示在元素旁边，不会遮挡页面原有内容
7. **防重复执行** - 2秒内阻止相同标注重复执行
8. **诊断工具** - 检测和修复标注问题
9. **清除标注** - 一键移除所有标注

### 脚本列表

| 脚本 | 功能 | 主要参数 |
|------|------|----------|
| `annotate.js` | 高亮元素并添加标签 | `selector/text`, `label`, `color` |
| `comment.js` | 添加文字批注 | `selector/text`, `comment`, `position`, `style`, `color` |
| `clear.js` | 清除所有标注 | 无 |
| `debug.js` | 诊断标注问题 | 无 |

### 使用流程

1. 用户描述需要标注的内容（如"在登录按钮旁添加批注"）
2. **提取目标位置的完整句子或段落**（15+ 字符，确保唯一性）
3. 使用长文本进行精确匹配
4. 调用相应脚本执行标注
5. 如遇问题，使用 debug.js 诊断

**重要提示**：始终使用长文本（完整句子）而不是短词，这是避免重复标注的最有效方法。

### 调用示例

**高亮元素（使用长文本匹配）：**
```json
{
  "skill_name": "page-annotator",
  "script_path": "scripts/annotate.js",
  "arguments": {
    "text": "Click the button below to start the installation process",
    "label": "安装步骤",
    "color": "yellow"
  }
}
```

**添加文字批注（长文本匹配）：**
```json
{
  "skill_name": "page-annotator",
  "script_path": "scripts/comment.js",
  "arguments": {
    "text": "Beyond the markdown content, you can configure skill behavior using YAML",
    "comment": "这里说明了如何使用 YAML 配置",
    "position": "right",
    "style": "bubble",
    "color": "blue"
  }
}
```

**添加文字批注（便签样式）：**
```json
{
  "skill_name": "page-annotator",
  "script_path": "scripts/comment.js",
  "arguments": {
    "text": "Make sure to backup your data before proceeding with the upgrade",
    "comment": "⚠️ 升级前务必备份数据",
    "style": "sticky",
    "color": "orange"
  }
}
```

**清除标注：**
```json
{
  "skill_name": "page-annotator",
  "script_path": "scripts/clear.js",
  "arguments": {}
}
```

**诊断问题：**
```json
{
  "skill_name": "page-annotator",
  "script_path": "scripts/debug.js",
  "arguments": {}
}
```

### 参数说明
```json
{
  "skill_name": "page-annotator",
  "script_path": "scripts/comment.js",
  "arguments": {
    "text": "用户登录",
    "comment": "这是登录功能",
    "position": "right",
    "color": "blue",
    "style": "bubble"
  }
}
```

**组合使用选择器和文本：**
```json
{
  "skill_name": "page-annotator",
  "script_path": "scripts/annotate.js",
  "arguments": {
    "selector": "h1",
    "text": "欢迎",
    "label": "标题",
    "color": "yellow"
  }
}
```

**禁用自动滚动：**
```json
{
  "skill_name": "page-annotator",
  "script_path": "scripts/comment.js",
  "arguments": {
    "selector": ".footer",
    "comment": "页脚信息",
    "autoScroll": false
  }
}
```

**限制匹配数量（避免重复）：**
```json
{
  "skill_name": "page-annotator",
  "script_path": "scripts/comment.js",
  "arguments": {
    "text": "MCP",
    "comment": "Model Context Protocol",
    "maxMatches": 1
  }
}
```

**标注多个匹配项：**
```json
{
  "skill_name": "page-annotator",
  "script_path": "scripts/annotate.js",
  "arguments": {
    "text": "重要",
    "label": "注意",
    "color": "red",
    "maxMatches": 5
  }
}
```

**包括隐藏元素：**
```json
{
  "skill_name": "page-annotator",
  "script_path": "scripts/comment.js",
  "arguments": {
    "text": "折叠菜单",
    "comment": "这是隐藏的菜单项",
    "onlyVisible": false
  }
}
```

### 参数说明

**通用参数（annotate.js 和 comment.js）：**
- `selector` (string, 可选*): CSS 选择器，用于定位元素
- `text` (string, 可选*): 文本内容，用于搜索包含该文本的元素
- `color` (string, 可选): 颜色主题，支持 `yellow`/`red`/`blue`/`green`/`orange`，默认 `yellow`
- `autoScroll` (boolean, 可选): 是否自动滚动到元素位置，默认 `true`
- `maxMatches` (number, 可选): 限制标注数量，默认 `1`（只标注第一个匹配项）
- `onlyVisible` (boolean, 可选): 是否只标注可见元素，默认 `true`（跳过隐藏元素）

*注：`selector` 和 `text` 至少提供一个，推荐使用 `text` 更可靠

**高亮专用（annotate.js）：**
- `label` (string, 可选): 标签文字，显示在元素上方

**批注专用（comment.js）：**
- `comment` (string, 必需): 批注内容
- `position` (string, 可选): 批注位置，支持 `right`/`left`/`top`/`bottom`，默认 `right`
- `style` (string, 可选): 批注样式
  - `bubble`（气泡）- 默认，圆角矩形，适合一般说明
  - `sticky`（便签）- 方形带图标，适合重要提醒
  - `inline`（行内）- 紧凑型，适合简短说明
- **注意**: 所有批注样式都会在原文下方自动添加下划线，清晰标识批注对应的内容

### 颜色方案

| 颜色 | 用途 | 适用场景 |
|------|------|----------|
| `yellow` | 一般信息（默认） | 普通说明、提示 |
| `red` | 错误、警告 | 问题标记、重要警告 |
| `blue` | 信息、说明 | 功能讲解、步骤说明 |
| `green` | 成功、推荐 | 正确做法、成功提示 |
| `orange` | 注意、待优化 | 需要改进的地方 |

### 选择器建议

**强制使用长文本匹配（推荐）：**

长文本匹配是最可靠的方式，可以避免 99% 的重复标注问题。

```json
// ✅ 最佳实践：使用完整句子
{
  "text": "Beyond the markdown content, you can configure skill behavior using YAML frontmatter",
  "comment": "YAML 配置说明"
}

// ✅ 推荐：使用段落开头
{
  "text": "The installation process is straightforward and takes only a few minutes to complete",
  "label": "安装说明"
}
```

**文本长度建议：**
- 最少：15 个字符
- 推荐：30-100 个字符（完整句子）
- 最佳：使用在页面中唯一出现的文本

**如何选择文本：**
1. 在页面上找到目标位置
2. 复制该位置的完整句子或段落
3. 使用 Ctrl+F / Cmd+F 验证文本唯一性
4. 如果有多个匹配，选择更长的文本

**文本搜索的智能优先级：**

当使用 `text` 参数时，系统会按以下优先级搜索元素：

1. **主内容区标题**（`<main>` 或 `<article>` 中的 `<h1>`-`<h6>`）
2. **主内容区文本**（`<main>` 或 `<article>` 中的段落、列表等）
3. **`.content` 区域**的标题和文本
4. **页面中的所有标题**
5. **其他文本元素**

**自动排除的区域：**
- 导航栏：`<nav>`, `.navigation`, `.navbar`
- 侧边栏：`<aside>`, `.sidebar`
- 页眉页脚：`<header>`, `<footer>`, `.header`, `.footer`

**优先级评分机制：**
- `<main>` 标签内：权重 10（最优先）
- `<article>` 标签内：权重 8
- `<section>` 标签内：权重 6
- 其他区域：权重 5
- 导航/侧边栏/页脚：权重 1（通常被过滤）

这样可以确保标注始终定位到最相关的内容区域。

**使用 CSS 选择器（不推荐）：**

只在以下情况使用 CSS 选择器：
- 目标元素没有文本内容（如图片、图标）
- 需要标注特定的 HTML 结构

```json
// 可以使用的选择器
{
  "selector": "#unique-id",
  "label": "特定元素"
}

// 避免使用的选择器
{
  "selector": "div > div > div > button:nth-child(3)"  // 太复杂，容易失效
}
```

### 最佳实践

1. **强制使用长文本匹配（最重要）**
   ```json
   // ❌ 错误：短词匹配多处
   { "text": "YAML" }  // 可能匹配 12+ 处
   
   // ❌ 错误：短句仍可能重复
   { "text": "配置文件" }  // 可能匹配多处
   
   // ✅ 正确：使用完整句子（15+ 字符）
   { 
     "text": "Beyond the markdown content, you can configure skill behavior using YAML",
     "comment": "YAML 配置说明"
   }
   
   // ✅ 正确：使用唯一的长文本
   { 
     "text": "Click the button below to start the installation process and follow the on-screen instructions",
     "comment": "安装步骤说明"
   }
   ```

2. **提取目标位置的完整句子**
   - 从页面复制完整的句子或段落
   - 确保文本长度 ≥ 15 个字符
   - 选择在页面中唯一出现的文本

3. **组合使用结构限制（可选）**
   ```json
   // 在主内容区搜索长文本
   {
     "selector": "main",
     "text": "This section describes the advanced configuration options available",
     "comment": "高级配置说明"
   }
   ```

4. **避免使用的文本类型**
   - ❌ 单个词：`"YAML"`, `"配置"`, `"按钮"`
   - ❌ 常见短语：`"点击这里"`, `"了解更多"`, `"查看详情"`
   - ❌ 标题文字：通常在导航、侧边栏、正文中重复出现

5. **推荐使用的文本类型**
   - ✅ 完整句子：包含主语、谓语、宾语的完整表达
   - ✅ 段落开头：段落的前 50-100 个字符
   - ✅ 唯一描述：页面中只出现一次的描述性文本

6. **验证文本唯一性**
   - 使用浏览器的查找功能（Ctrl+F / Cmd+F）
   - 搜索你选择的文本
   - 确保只有 1 个匹配结果

7. **遇到问题时先诊断**
   ```json
   { "script_path": "scripts/debug.js" }
   ```

### 常见问题

**Q: 标注位置不准确？**
- 确保浏览器缩放为 100%
- 等待页面完全加载后再标注
- 使用 `autoScroll: true` 自动滚动到元素

**Q: 出现重复的标注？**
- **最重要**：使用长文本（15+ 字符的完整句子）而不是短词
- 示例对比：
  - ❌ `"text": "YAML"` → 匹配 12 处
  - ✅ `"text": "Beyond the markdown content, you can configure skill behavior using YAML"` → 唯一匹配
- 系统会自动阻止 2 秒内的重复执行
- 默认 `maxMatches: 1` 只标注第一个匹配项
- 使用浏览器查找功能（Ctrl+F）验证文本唯一性
- 使用 `debug.js` 检查重复情况
- 使用 `clear.js` 清除后重新标注

**Q: 找不到元素？**
- **首先检查文本长度**：确保使用 15+ 字符的长文本
- 使用完整句子而不是短词或短语
- 从页面复制完整的文本内容
- 检查返回的 `suggestions` 字段获取建议
- 确认元素是否在 iframe 或 Shadow DOM 中
- 等待页面完全加载后再标注

**Q: 批注相互重叠？**
- 系统会自动进行碰撞检测和位置调整
- 如果仍然重叠，说明空间不足，考虑清除部分标注

**Q: 批注会遮挡原文吗？**
- 不会！系统会自动检测批注与原文的重叠
- 批注会自动调整位置，确保始终显示在元素旁边
- 即使空间有限，批注也会尽量避开原文内容
- 批注使用 `pointer-events: none`，鼠标可以穿透批注点击下方内容

**Q: 如何只标注主内容区的元素？**
```json
// 方法 1：使用长文本（推荐）
{
  "text": "This is the main content paragraph that appears only once in the article",
  "comment": "主要内容说明"
}

// 方法 2：组合选择器和长文本
{
  "selector": "main",
  "text": "Complete sentence from the main content area",
  "comment": "说明"
}
```

**Q: 如何避免标注到导航栏的内容？**
- 系统默认会自动排除 `<nav>`, `<aside>`, `<header>`, `<footer>` 等区域
- 使用 `onlyVisible: true`（默认）会跳过隐藏元素
- 组合使用 `selector: "main"` 或 `selector: "article"` 限制范围

### 高级功能

详细文档请参考：
- [长文本匹配最佳实践](LONG_TEXT_MATCHING.md) - **必读**：如何使用长文本避免重复标注
- [批注功能指南](references/comment-guide.md) - 批注样式和位置详解
- [选择器指南](references/selector-guide.md) - CSS 选择器使用说明
- [故障排除](references/troubleshooting.md) - 常见问题解决方案
- [使用场景](references/use-cases.md) - 实际应用示例
