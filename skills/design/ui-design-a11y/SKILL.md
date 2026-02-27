---
name: ui-design-a11y
description: 无障碍设计审查与修复能力。
---

# Accessibility (A11y) Skill

## 技能描述
专注于 Web 无障碍标准 (WCAG) 的检查与修复。帮助开发者发现潜在的访问性问题并提供修复代码。

## ♿ 常用指令

### 1. 快速审查 (Audit)
```bash
/ui-design 审查这段代码的无障碍问题
# 关注点:
# - 语义化标签 (nav, main, aside)
# - ARIA 属性使用
# - 键盘可访问性 (tabindex, focus)
# - 图片 Alt 文本
```

### 2. 对比度检查 (Contrast)
```bash
/ui-design 检查这个配色的对比度是否符合 WCAG AA 标准
```

### 3. 修复建议 (Fix)
```bash
/ui-design 修复这个表单的 Label 关联和 ARIA 描述问题
/ui-design 为这个模态框添加 Focus Trap (焦点陷阱) 功能
```

## ✅ Self-Checklist (自查清单)

在提交代码前，请检查：
- [ ] 所有的 `<img>` 都有 `alt` 属性吗？
- [ ] 所有的表单输入框都有对应的 `<label>` 吗？
- [ ] 按钮是否有清晰的文字描述（避免仅图标）？
- [ ] 页面是否可以仅用键盘顺畅操作？
- [ ] 颜色对比度是否足够清晰？
