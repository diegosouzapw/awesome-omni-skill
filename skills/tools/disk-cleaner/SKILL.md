---
name: disk-cleaner
description: Mac 智能磁盘清理助手，基于 Mole (https://github.com/tw93/Mole) 的用户友好包装器
---

# Disk Cleaner - Mac 智能磁盘清理助手

基于 Mole (https://github.com/tw93/Mole) 的用户友好包装器，提供环境检测、自动安装、预览分析、清理执行和成就展示。

## Description

Disk Cleaner 是对 tw93 开发的 Mole 清理工具的增强封装。它提供了更友好的中文界面、分类报告、三档清理策略建议，以及清理完成后的精美成就页面（包含省钱计算和趣味统计）。

## When to Use

Use this skill when users:
- 说硬盘空间不够、磁盘满了、存储不足
- 想清理 Mac 缓存或临时文件
- 询问如何释放磁盘空间
- 提到 Mole 工具

## Features

- **🔍 环境检测**: 自动检测 Homebrew 和 Mole 安装状态
- **📦 自动安装**: 支持一键安装缺失依赖
- **📊 分类报告**: 按类别展示可清理项目（浏览器缓存、包管理器、AI 模型等）
- **🧭 三档策略**: 低风险 / 默认 / 最大拯救三种清理建议
- **🎉 成就页面**: 清理后展示省钱计算、趣味统计、随机夸夸 tw93
- **🔒 安全保护**: 显示已保护项目，支持确认机制

## Usage

### 环境检查

```bash
python scripts/mole_cleaner.py --check
```

### 预览清理内容

```bash
python scripts/mole_cleaner.py --preview
python scripts/mole_cleaner.py --preview --json  # JSON 格式输出
```

### 执行清理

```bash
python scripts/mole_cleaner.py --clean --confirm
```

### 查看磁盘状态

```bash
python scripts/mole_cleaner.py --status
```

### 显示成就页（测试）

```bash
python scripts/mole_cleaner.py --show-achievement
```

## Workflow

1. **环境检查**: 检测 Homebrew 和 Mole 是否已安装
2. **预览分析**: 运行 `mo clean --dry-run` 并解析结果
3. **生成报告**: 按类别统计可清理空间，提供三档建议
4. **确认执行**: 用户确认后执行清理
5. **成就展示**: 显示释放空间、省钱金额、趣味统计，自动打开 HTML 成就页

## Safety Features

- **预览优先**: 默认只预览，需要 `--confirm` 才执行清理
- **保护清单**: 显示 Mole 的 whitelist 保护项目
- **分类建议**: 区分安全清理和谨慎清理项目
- **日志记录**: 所有操作保存日志到 `~/.config/mole-cleaner/logs/`

## Achievement Page

清理完成后自动生成并打开 Notion 风格的成就页面：

- **省钱计算**: 按加装 1TB SSD ≈ ¥3000 计算省下的钱
- **趣味统计**: 等价于多少张照片 / 首歌曲
- **随机夸夸**: 每次随机展示一条对 tw93 的幽默夸赞
- **GitHub 链接**: 直接链接到 Mole 项目，方便 Star

## Dependencies

- macOS
- Homebrew
- Mole (`brew install tw93/tap/mole`)

## Credits

- **Mole**: https://github.com/tw93/Mole
- **作者**: tw93 (https://tw93.fun)
