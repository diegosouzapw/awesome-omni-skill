---
name: apifox-core
description: Apifox 生成器核心工具（由 Agent 内部使用，不对外暴露）
---

# Apifox Core Plugin Utilities

提供智能推荐、配置管理等核心功能。

**注意：** 此模块由 `apifox-generator` Agent 内部使用，用户无需直接调用。

## 功能

### 1. 智能推荐

基于项目特征推荐最佳生成器和输出路径。

### 2. 配置管理

读写 `apifox.config.json` 配置文件。

### 3. 配置验证

验证配置文件格式（基于 JSON Schema）。

### 4. 代码生成

封装 Docker OpenAPI Generator 命令。

## 架构

核心模块：
- `SmartRecommender` - 智能推荐算法
- `ConfigManager` - 配置文件管理
- `CodeGenerator` - Docker 命令封装（已简化，Agent 可直接调用 Docker）

## 使用

此 Skill 不提供独立接口，完全由 Agent 内部使用。Agent 会直接调用文件操作和 Docker 命令。
