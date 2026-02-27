---
name: browser-dev-tools
description: 使用 Chrome DevTools MCP 进行前端页面调试、布局优化、性能诊断及交互验证。
---

# Browser DevTools - 浏览器级研发调试

此 Skill 允许 AI 直接与实际运行的浏览器窗口交互，通过 DevTools 协议实现精准的 UI 修补和性能优化。

## 何时使用此 Skill

- **样式微调**：需要观察组件在不同屏幕尺寸下的布局表现。
- **深度调试**：控制台报错、网络请求失败或复杂的各种竞态条件。
- **性能评估**：测量 C# 或 Vue 变更后对浏览器渲染管道（LCP, CLS）的实际影响。
- **UI 验收**：自动执行点击流并截图以确认交互逻辑。

## 核心工具流

### 1. 现状感知 (Initial Assessment)

在开始任何修改前，先观察页面：

- `list_pages`: 找到目标页面索引。
- `capture_screenshot`: 获取视觉反馈，确认当前样式。
- `text_snapshot`: 获取无障碍树和 UID，确定要操作的元素。

### 2. 交互验证 (Interaction)

- `click`: 模拟点击。
- `fill_form`: 批量填充测试数据。
- `evaluate_script`: 执行 JS 检查当前组件状态（如 `__vue_app__` 数据）。

### 3. 样式与布局调优 (Design Inspection)

- 使用 `resize_page` 切换移动端/桌面端。
- 使用 `capture_screenshot` 配合 `evaluate_script` 修改运行时 CSS，实时验证效果。

### 4. 性能与错误诊断 (Diagnostics)

- `list_console_messages`: 定位 runtime errors。
- `performance_start_trace`: 开始性能采样。
- `performance_analyze_insight`: 分析特定性能瓶颈（如 DocumentLatency 或 LCPBreakdown）。

## 最佳实践规范

- **调试优先**：在修改持久化代码前，先尝试用 `evaluate_script` 手动改动运行时 DOM/CSS 以验证猜想。
- **对比一致性**：在优化样式后，必须取 `capture_screenshot` 并查看其快照，确保符合 [设计系统规范](src/frontend/docs/design-system.md)。
- **清理工作**：如果打开了新页面或开始了追踪，操作结束后记得关闭或停止。

## 使用示例

- “请帮我检查 `/articles/new` 页面在 iPhone 12 尺寸下的布局，如果提交按钮被遮挡请修复 CSS。”
- “这个组件渲染太慢了，请运行性能追踪，找出占用主线程最长的任务。”
- “自动填充该注册表单，并告诉我在点击注册后，控制台是否输出了任何网络错误。”
