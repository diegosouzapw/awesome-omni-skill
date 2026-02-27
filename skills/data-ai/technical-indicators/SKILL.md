---
name: technical-indicators
description: 计算股票技术指标（MA、MACD、RSI、BOLL等）。适用于技术分析、量化回测、趋势判断等场景。支持单个指标计算或批量计算所有指标，返回包含指标值的DataFrame。
license: MIT
allowed-tools:
  - calculate_ma
  - calculate_macd
  - calculate_rsi
  - calculate_boll
  - calculate_all_indicators
metadata:
  version: "1.0.0"
  author: "YFOOOO"
  indicators:
    - MA (移动平均线)
    - MACD (指数平滑异同移动平均线)
    - RSI (相对强弱指标)
    - BOLL (布林带)
---

# Technical Indicators Skill

专业的股票技术指标计算工具，支持主流技术分析指标。

## Overview

本 Skill 提供以下核心功能：
1. **单个指标计算** - 计算 MA、MACD、RSI、BOLL 等单一指标
2. **批量计算** - 一次性计算所有指标
3. **指标解读** - 提供指标含义和使用建议

所有计算基于 pandas 和 talib，支持向量化高性能计算。

## Quick Start

### 1. 准备数据

技术指标计算需要 OHLCV 数据（开盘、最高、最低、收盘、成交量）。
通常先调用 `financial-data-fetch` Skill 获取数据。

### 2. 选择指标

根据分析需求选择合适的指标：
- **趋势分析** → MA (移动平均线)
- **动量分析** → MACD, RSI
- **波动分析** → BOLL (布林带)
- **综合分析** → calculate_all_indicators

### 3. 调用工具

传入 DataFrame 数据，返回包含指标值的新 DataFrame。

## Indicators

### 1. MA (移动平均线)

**Tool**: `calculate_ma`

**Description**: 计算简单移动平均线（SMA），用于判断价格趋势。

**Parameters**:
- `data` (DataFrame, required): 包含"收盘"列的价格数据
- `periods` (list[int], optional): MA 周期列表，默认 [5, 10, 20, 60]

**Returns**: DataFrame，新增列：MA5, MA10, MA20, MA60

**Example**:
```python
result = calculate_ma(data=stock_df, periods=[5, 10, 20])
# 返回包含 MA5, MA10, MA20 列的 DataFrame
```

**Interpretation**:
- 短期MA（5日）上穿长期MA（20日） → 金叉，买入信号
- 短期MA下穿长期MA → 死叉，卖出信号

---

### 2. MACD (指数平滑异同移动平均线)

**Tool**: `calculate_macd`

**Description**: 计算 MACD 指标，用于判断买卖时机和趋势强度。

**Parameters**:
- `data` (DataFrame, required): 包含"收盘"列的价格数据
- `fast_period` (int, optional): 快线周期，默认 12
- `slow_period` (int, optional): 慢线周期，默认 26
- `signal_period` (int, optional): 信号线周期，默认 9

**Returns**: DataFrame，新增列：MACD_DIF, MACD_DEA, MACD_BAR

**Example**:
```python
result = calculate_macd(data=stock_df)
# 返回包含 MACD_DIF, MACD_DEA, MACD_BAR 的 DataFrame
```

**Interpretation**:
- DIF 上穿 DEA → 金叉，买入信号
- DIF 下穿 DEA → 死叉，卖出信号
- BAR 柱状图放大 → 趋势增强

---

### 3. RSI (相对强弱指标)

**Tool**: `calculate_rsi`

**Description**: 计算 RSI 指标，用于判断超买超卖状态。

**Parameters**:
- `data` (DataFrame, required): 包含"收盘"列的价格数据
- `period` (int, optional): RSI 周期，默认 14

**Returns**: DataFrame，新增列：RSI

**Example**:
```python
result = calculate_rsi(data=stock_df, period=14)
# 返回包含 RSI 列的 DataFrame
```

**Interpretation**:
- RSI > 70 → 超买区域，可能回调
- RSI < 30 → 超卖区域，可能反弹
- RSI 在 30-70 之间 → 正常区域

---

### 4. BOLL (布林带)

**Tool**: `calculate_boll`

**Description**: 计算布林带指标，用于判断价格波动范围。

**Parameters**:
- `data` (DataFrame, required): 包含"收盘"列的价格数据
- `period` (int, optional): MA 周期，默认 20
- `std_dev` (float, optional): 标准差倍数，默认 2.0

**Returns**: DataFrame，新增列：BOLL_MIDDLE, BOLL_UPPER, BOLL_LOWER

**Example**:
```python
result = calculate_boll(data=stock_df, period=20, std_dev=2.0)
# 返回包含 BOLL_MIDDLE, BOLL_UPPER, BOLL_LOWER 的 DataFrame
```

**Interpretation**:
- 价格触及上轨 → 超买，可能回调
- 价格触及下轨 → 超卖，可能反弹
- 价格突破上轨 → 强势上涨
- 价格跌破下轨 → 强势下跌

---

### 5. 批量计算所有指标

**Tool**: `calculate_all_indicators`

**Description**: 一次性计算所有主流技术指标（MA、MACD、RSI、BOLL）。

**Parameters**:
- `data` (DataFrame, required): 包含 OHLCV 列的价格数据

**Returns**: DataFrame，新增所有指标列

**Example**:
```python
result = calculate_all_indicators(data=stock_df)
# 返回包含所有指标的 DataFrame
```

## Data Requirements

### 输入数据格式

所有工具要求输入 DataFrame 包含以下列：
- ✅ **必需**: `收盘` (float) - 收盘价
- ⚠️ **可选**: `开盘`, `最高`, `最低`, `成交量` (部分指标需要)

### 数据质量要求

1. **最小数据量**:
   - MA: 至少 max(periods) + 1 行
   - MACD: 至少 34 行（slow_period + signal_period）
   - RSI: 至少 period + 1 行
   - BOLL: 至少 period + 1 行

2. **数据连续性**: 建议使用连续交易日数据（无缺失）

3. **数据排序**: 按日期升序排列

## Error Handling

### 1. 数据不足
```
错误：数据量不足，需要至少 34 行数据计算 MACD
```

### 2. 列名缺失
```
错误：DataFrame 缺少 '收盘' 列
```

### 3. 数据类型错误
```
错误：'收盘' 列必须为数值类型（float/int）
```

### 4. 参数超出范围
```
错误：RSI 周期必须 > 0（当前: -1）
```

## Performance Notes

- **计算时间**: 
  - 单个指标: < 100ms (1000行数据)
  - 批量计算: < 500ms (1000行数据)
- **内存使用**: 约为原始数据的 2-3 倍
- **优化建议**: 使用向量化操作，避免循环

## References

详细指标公式和解读见：
- [指标公式详解](references/indicators_formula.md)
- [指标解读指南](references/interpretation.md)

## Script Implementation

指标计算逻辑实现在：
- `scripts/calculate_ma.py` - MA 计算
- `scripts/calculate_macd.py` - MACD 计算
- `scripts/calculate_all.py` - 批量计算（复用 `core/indicators.py`）

这些脚本复用项目中的 `core/indicators.py` 模块，确保计算一致性。
