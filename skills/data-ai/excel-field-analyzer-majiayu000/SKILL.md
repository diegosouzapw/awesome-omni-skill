---
name: excel-field-analyzer
description: "分析Excel/CSV字段结构，AI自动生成中英文映射，验证翻译质量，输出统计报告。用于电子表格分析、数据字典创建、字段映射场景。"
allowed-tools: Read, Bash, Write, Glob, Grep
---

# Excel/CSV 字段分析器

## 概述

智能分析 Excel 与 CSV 文件，自动生成中英文字段映射、统计报告与 HTML 可视化。

**核心能力：**
- 字段统计分析（空值率、唯一值、分布）
- AI 字段映射（内置 50+ 车险领域字段）
- 映射质量自动校验
- HTML 可视化报告

## 快速开始

### 对话式调用（推荐）

直接与 Claude 对话：
```
"帮我分析这个 Excel 文件的字段"
"分析 ./data/insurance_data.xlsx 的字段映射"
```

### 命令行

```bash
# 基础分析
python scripts/analyzer.py <file_path> [output_dir] [topn]

# 示例
python scripts/analyzer.py data.xlsx ./output 10

# 支持格式：.xlsx, .xls, .csv, .txt
```

## 核心特性

### 1. 预置映射库
- **车险领域**：内置 50+ 字段映射
- **覆盖范围**：财务、车辆、机构、产品、时间
- **示例**：
  - `商业险保费` → `commercial_premium`（finance/number）
  - `三级机构` → `org_level_3`（organization/string）
  - `确认时间` → `time_confirm`（time/datetime）

### 2. AI 批量学习
- **零人工**：自动为未知字段生成映射
- **智能分析**：语义分析 + 数据样本推断
- **自动保存**：结果保存至 `custom.json` 便于复用
- **高准确率**：70 字段测试集准确率 100%

**示例：**
```
🔍 Found 70 unknown fields
💡 Using AI to generate mappings...
✅ Generated 70 mappings and saved to custom.json

- 刷新时间 → time_refresh [time/datetime]
- 交叉销售标识 → flag_cross_sales [flag/string]
- 签单保费 → premium_signing [finance/number]
```

### 3. 质量校验
- **自动检查**：4 个维度（命名、分组、语义、类型）
- **质量评分**：优秀（≥90）/ 良好（75-89）/ 一般（60-74）/ 较差（<60）
- **详细报告**：Markdown，附改进建议

### 4. 交互式学习
- **手动模式**：可选，精确控制字段映射
- **引导流程**：逐步选择英文字段名、分组、类型
- **持久存储**：全部学习映射保存至 `custom.json`

## 输出文件

### 1. HTML 可视化报告
- 文件元信息与生成时间
- 每个工作表的完整统计表
- 数值统计与 Top 值分布
- 交互式探索

### 2. JSON 字段映射表
```json
{
  "field_name": "commercial_premium",
  "cn_name": "商业险保费",
  "group": "finance",
  "dtype": "number",
  "role": "measure",
  "aggregation": "sum",
  "is_mapped": true
}
```

### 3. 质量校验报告（Markdown）
- 总体质量统计
- 需复审字段与建议
- 优秀映射示例
- 质量分布可视化

## 业务分组

| 分组 | 描述 | 示例 |
|------|------|------|
| finance | 财务数据 | 保费、赔款、费用 |
| organization | 机构信息 | 三级机构、四级机构 |
| vehicle | 车辆相关 | 车牌、车型 |
| product | 产品信息 | 险类、险种 |
| time | 时间字段 | 确认时间、起保日期 |
| flag | 状态标识 | 续保标识、新能源标识 |
| partner | 合作方信息 | 4S 集团、经销商 |
| general | 通用字段 | 业务类型、客户类别 |

## 文档

- **reference.md** - 完整技术文档、配置细节、API 参考
- **examples.md** - 代码示例、使用场景、集成指南

## 版本历史

### v2.3 (2025-11-23) - 质量保障
- 🔍 映射质量校验体系
- 4 个校验维度与质量评分
- 自动生成质量报告

### v2.2 (2025-11-23) - AI 批量学习
- 🤖 AI 驱动的自动字段映射
- 语义分析 + 数据样本推断
- 测试数据集准确率 100%

### v2.1 (2025-11-23)
- ✨ 支持 CSV 文件
- 统一 Excel 与 CSV 接口

### v2.0 (2025-11-23)
- ✨ Claude Code Skill 架构
- 多源配置系统
- 交互式字段学习

## 依赖

```bash
pip install pandas openpyxl numpy
```

## 许可证

MIT 许可证
