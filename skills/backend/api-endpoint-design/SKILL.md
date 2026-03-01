---
name: api-endpoint-design
description: API endpoint design and testing for vehicle insurance data platform. Use when designing new API endpoints, testing existing ones, validating response formats, or debugging API issues. Covers 11 core endpoints including 3 new pie chart distribution endpoints, parameter validation, error handling, and integration patterns.
allowed-tools: Read, Edit, Grep, Glob
---

# API Endpoint Design

REST API design specifications for vehicle insurance data analysis platform.

**Key Files**:
- [backend/api_server.py](../../backend/api_server.py) - Flask API server
- [frontend/src/services/api.js](../../frontend/src/services/api.js) - Frontend API client

**Related Docs**:
- See [TESTING_TEMPLATE.md](../testing-and-debugging/TESTING_TEMPLATE.md) for unit test templates
- See [COMMON_ISSUES.md](../testing-and-debugging/COMMON_ISSUES.md) for troubleshooting

---

## 📚 一、API 端点清单

### 概览表格

| 端点 | 方法 | 功能 | 优先级 |
|------|------|------|--------|
| `/api/health` | GET | 健康检查 | P0 |
| `/api/latest-date` | GET | 获取最新日期 | P0 |
| `/api/refresh` | POST | 刷新数据 | P0 |
| `/api/filter-options` | GET | 获取筛选选项 | P0 |
| `/api/kpi-windows` | POST | KPI 三口径数据 | P0 |
| `/api/week-comparison` | POST | 周对比图表数据 | P0 |
| `/api/insurance-type-distribution` | POST | 险别组合占比 | P0 |
| `/api/premium-range-distribution` | POST | 业务员保费区间占比 | P0 |
| `/api/renewal-type-distribution` | POST | 新转续占比 | P0 |
| `/api/policy-mapping` | GET | 保单映射信息 | P1 |
| `/api/staff-performance-distribution` | POST | 业务员业绩分布 | P1 |

---

### 1.1 `/api/health` - 健康检查

**位置**: [api_server.py:114-120](../../backend/api_server.py#L114-L120)

**请求**:
```http
GET /api/health
```

**响应**:
```json
{
  "status": "healthy",
  "message": "API服务运行正常"
}
```

**测试示例**:
```bash
curl http://localhost:5001/api/health
```

---

### 1.2 `/api/latest-date` - 获取最新日期

**位置**: [api_server.py:96-111](../../backend/api_server.py#L96-L111)

**响应**:
```json
{
  "success": true,
  "latest_date": "2025-11-08"
}
```

**错误响应**:
```json
{
  "success": false,
  "message": "获取最新日期失败: 未找到数据文件"
}
```

**测试示例**:
```bash
curl http://localhost:5001/api/latest-date
```

---

### 1.3 `/api/refresh` - 刷新数据

**位置**: [api_server.py:21-37](../../backend/api_server.py#L21-L37)

**请求**: `POST /api/refresh`

**响应**:
```json
{
  "success": true,
  "message": "数据刷新成功",
  "latest_date": "2025-11-08"
}
```

**注意事项**:
- ⚠️ 此操作较耗时(大文件可能 10-30 秒)
- ⚠️ 前端应显示加载状态
- ⚠️ 建议设置 60 秒超时

**测试示例**:
```bash
curl -X POST http://localhost:5001/api/refresh
```

---

### 1.4 `/api/filter-options` - 获取筛选选项

**位置**: [api_server.py:123-138](../../backend/api_server.py#L123-L138)

**响应**:
```json
{
  "success": true,
  "data": {
    "三级机构": ["达州", "德阳", "绵阳", "南充"],
    "团队": ["达州业务一部", "德阳业务三部"],
    "是否续保": ["是", "否"],
    "是否新能源": ["是", "否"],
    "是否过户车": ["是", "否"],
    "险种大类": ["商业险", "交强险"],
    "吨位": ["<2吨", "2-5吨", "5-10吨", ">10吨"],
    "是否电销": ["全部", "是", "否"],
    "机构团队映射": {
      "达州": ["业务一部", "业务二部"],
      "德阳": ["业务三部"]
    },
    "保单号": ["P202511080001", "P202511080002", "..."]
  }
}
```

**测试示例**:
```bash
curl http://localhost:5001/api/filter-options
```

---

### 1.5 `/api/kpi-windows` - KPI 三口径数据

**位置**: [api_server.py:205-241](../../backend/api_server.py#L205-L241)

**请求**:
```http
POST /api/kpi-windows
Content-Type: application/json

{
  "filters": {
    "三级机构": "达州",
    "团队": "业务一部",
    "是否续保": "是"
  },
  "date": "2025-11-08"
}
```

**参数说明**:
- `filters` (object, 可选): 筛选条件
- `date` (string, 可选): 锚定日期 (YYYY-MM-DD), 默认为最新日期

**响应**:
```json
{
  "success": true,
  "data": {
    "anchor_date": "2025-11-08",
    "premium": {
      "day": 125000.50,
      "last7d": 875420.30,
      "last30d": 3250800.75
    },
    "policy_count": {
      "day": 234,
      "last7d": 1680,
      "last30d": 6420
    },
    "commission": {
      "day": 5000.00,
      "last7d": 35016.81,
      "last30d": 130032.03
    }
  }
}
```

**业务逻辑**:
- **当日(day)**: 指定日期当天的数据
- **近7天(last7d)**: 从指定日期往前推 6 天(共 7 天)
- **近30天(last30d)**: 从指定日期往前推 29 天(共 30 天)

**测试示例**:
```bash
curl -X POST http://localhost:5001/api/kpi-windows \
  -H "Content-Type: application/json" \
  -d '{"filters": {"三级机构": "达州"}}'
```

---

### 1.6 `/api/week-comparison` - 周对比图表数据

**位置**: [api_server.py:164-202](../../backend/api_server.py#L164-L202)

**请求**:
```http
POST /api/week-comparison
Content-Type: application/json

{
  "metric": "premium",
  "filters": {"三级机构": "达州"},
  "date": "2025-11-08"
}
```

**参数说明**:
- `metric` (string, 必填): 指标类型 (`premium` 或 `count`)
- `filters` (object, 可选): 筛选条件
- `date` (string, 可选): 锚定日期

**响应**:
```json
{
  "success": true,
  "data": {
    "x_axis": ["周三", "周四", "周五", "周六", "周日", "周一", "周二"],
    "series": [
      {
        "name": "D-14 (10-25)",
        "data": [120000, 135000, 98000, 110000, 125000, 95000, 98000]
      },
      {
        "name": "D-7 (11-01)",
        "data": [95000, 88000, 92000, 105000, 98000, 87000, 92000]
      },
      {
        "name": "D (11-08)",
        "data": [112000, 118000, 105000, 120000, 115000, 110000, 111000]
      }
    ]
  }
}
```

**周期划分**: 以锚定日期为结束日，往前推 7 天为一个周期
- D: 最近 7 天(11-08 ~ 11-14)
- D-7: 次近 7 天(11-01 ~ 11-07)
- D-14: 第三个 7 天(10-25 ~ 10-31)

**测试示例**:
```bash
curl -X POST http://localhost:5001/api/week-comparison \
  -H "Content-Type: application/json" \
  -d '{"metric": "premium"}'
```

---

### 1.7 `/api/policy-mapping` - 保单映射信息

**位置**: [api_server.py:141-161](../../backend/api_server.py#L141-L161)

**响应**:
```json
{
  "success": true,
  "data": {
    "policy_to_staff": {
      "P202511080001": "张三",
      "P202511080002": "李四"
    },
    "staff_to_info": {
      "张三": {
        "三级机构": "达州",
        "团队简称": "业务一部"
      }
    },
    "conflicts": ["王五"]
  }
}
```

**测试示例**:
```bash
curl http://localhost:5001/api/policy-mapping
```

---

### 1.8 `/api/staff-performance-distribution` - 业务员业绩分布

**位置**: [api_server.py:244-314](../../backend/api_server.py#L244-L314)

**请求**:
```http
POST /api/staff-performance-distribution
Content-Type: application/json

{
  "period": "day",
  "filters": {"三级机构": "达州"}
}
```

**参数说明**:
- `period` (string, 必填): `day`, `last7d`, `last30d`
- `filters` (object, 可选): 筛选条件

**响应**:
```json
{
  "success": true,
  "data": {
    "distribution": [
      {"range": "<1万", "count": 15, "percentage": 37.5},
      {"range": "1-2万", "count": 12, "percentage": 30.0},
      {"range": "2-3万", "count": 8, "percentage": 20.0},
      {"range": "3-5万", "count": 3, "percentage": 7.5},
      {"range": ">=5万", "count": 2, "percentage": 5.0}
    ]
  }
}
```

**测试示例**:
```bash
curl -X POST http://localhost:5001/api/staff-performance-distribution \
  -H "Content-Type: application/json" \
  -d '{"period": "day"}'
```

---

## 🎯 二、统一响应格式

### 2.1 成功响应

```json
{
  "success": true,
  "data": {
    // 业务数据
  }
}
```

### 2.2 失败响应

```json
{
  "success": false,
  "message": "错误描述信息"
}
```

**HTTP 状态码**:
- `200`: 请求成功
- `400`: 参数错误
- `404`: 资源不存在
- `500`: 服务器内部错误

### 最佳实践

```json
// ✅ 推荐: 始终包含 success 字段
{"success": true, "data": {...}}

// ✅ 推荐: 错误消息清晰具体
{"success": false, "message": "数据刷新失败: Permission denied"}

// ✅ 推荐: 保持字段命名一致(统一下划线)
{"latest_date": "2025-11-08", "total_staff": 40}

// ✅ 推荐: 避免返回 null
{"success": false, "message": "未找到数据"}
```

---

## 📋 三、参数验证规范

### 3.1 参数类型校验

#### 筛选条件 (filters)

```python
{
  "三级机构": str,      # "达州" / "德阳" / "全部"
  "团队": str,          # "业务一部" / "全部"
  "是否续保": str,      # "是" / "否" / "全部"
  "是否新能源": str,    # "是" / "否" / "全部"
  "保单号": str         # "P202511080001"
}
```

**验证逻辑**:
```python
# ✅ 推荐: 忽略无效字段
valid_keys = {'三级机构', '团队', '是否续保', ...}
filters = {k: v for k, v in filters.items() if k in valid_keys}
```

#### 日期参数 (date)

**格式**: `YYYY-MM-DD`
**默认值**: 最新数据日期

```python
date_str = data.get('date', None)
if date_str:
    try:
        date = pd.to_datetime(date_str)
    except Exception:
        return jsonify({
            "success": False,
            "message": "日期格式不正确，请使用 YYYY-MM-DD 格式"
        }), 400
```

#### 指标类型 (metric)

**有效值**: `"premium"` | `"count"`

```python
metric = data.get('metric', 'premium')
if metric not in ['premium', 'count']:
    return jsonify({
        "success": False,
        "message": "metric 参数必须为 'premium' 或 'count'"
    }), 400
```

#### 时间段 (period)

**有效值**: `"day"` | `"last7d"` | `"last30d"`

```python
period = data.get('period', 'day')
if period not in ['day', 'last7d', 'last30d']:
    return jsonify({
        "success": False,
        "message": "period 参数必须为 'day', 'last7d' 或 'last30d'"
    }), 400
```

### 3.2 默认值处理

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `filters` | `{}` | 空对象表示不筛选 |
| `date` | 最新日期 | 自动获取 |
| `metric` | `"premium"` | 默认查询保费 |
| `period` | `"day"` | 默认当日 |

```python
data = request.get_json() or {}
filters = data.get('filters', {})
date = data.get('date', None)
```

### 3.3 最佳实践

```python
# ✅ 宽进严出: 接受多种输入格式
if filters.get('三级机构') in ['全部', 'all', None, '']:
    # 不筛选

# ✅ 提前验证，快速失败
if metric not in ['premium', 'count']:
    return jsonify({"success": False, "message": "..."}), 400

# ✅ 友好的错误提示
return jsonify({
    "success": False,
    "message": "日期格式不正确，请使用 YYYY-MM-DD 格式: 2025-11-08"
}), 400

# ✅ 处理空值
data = request.get_json() or {}
filters = data.get('filters', {})
```

---

## ⚠️ 四、错误码体系

### HTTP 状态码

| 状态码 | 含义 | 使用场景 |
|--------|------|----------|
| `200` | OK | 请求成功 |
| `400` | Bad Request | 参数错误 |
| `404` | Not Found | 资源不存在 |
| `500` | Internal Server Error | 服务器内部错误 |

### 业务错误码(未来扩展)

```json
{
  "success": false,
  "message": "业务员映射缺失",
  "code": "STAFF_MAPPING_MISSING",
  "details": {
    "unmatched_staff": ["张三", "李四"]
  }
}
```

### 错误处理最佳实践

```python
# ✅ 区分客户端错误和服务器错误
if metric not in ['premium', 'count']:
    return jsonify({"success": False, "message": "..."}), 400  # 400

try:
    result = processor.get_kpi_windows(...)
except Exception as e:
    return jsonify({"success": False, "message": f"...{str(e)}"}), 500  # 500

# ✅ 记录详细错误日志
try:
    result = processor.get_kpi_windows(...)
except Exception as e:
    logging.error(f"获取KPI数据失败: {e}", exc_info=True)
    return jsonify({"success": False, "message": "获取KPI数据失败"}), 500

# ✅ 避免泄露敏感信息
# ❌ 不推荐(泄露文件路径)
return jsonify({"success": False, "message": f"文件不存在: {file_path}"}), 404

# ✅ 推荐
return jsonify({"success": False, "message": "数据文件不存在"}), 404
```

---

## 🧪 五、测试方法

### 5.1 cURL 快速测试

```bash
# 健康检查
curl http://localhost:5001/api/health

# 最新日期
curl http://localhost:5001/api/latest-date

# 刷新数据
curl -X POST http://localhost:5001/api/refresh

# KPI 三口径
curl -X POST http://localhost:5001/api/kpi-windows \
  -H "Content-Type: application/json" \
  -d '{"filters": {"三级机构": "达州"}}'

# 周对比
curl -X POST http://localhost:5001/api/week-comparison \
  -H "Content-Type: application/json" \
  -d '{"metric": "premium"}'

# 业绩分布
curl -X POST http://localhost:5001/api/staff-performance-distribution \
  -H "Content-Type: application/json" \
  -d '{"period": "day"}'
```

### 5.2 单元测试

**测试工具**: pytest + Flask Test Client

**完整测试模板**: [TESTING_TEMPLATE.md](../testing-and-debugging/TESTING_TEMPLATE.md)

**快速示例**:
```python
import pytest
from backend.api_server import app

@pytest.fixture
def client():
    app.config['TESTING'] = True
    with app.test_client() as client:
        yield client

def test_health_check(client):
    response = client.get('/api/health')
    assert response.status_code == 200
```

---

## 🎯 六、RESTful 最佳实践

### 资源命名

✅ **推荐**:
```
GET  /api/filter-options
POST /api/kpi-windows
GET  /api/policy-mapping
```

❌ **避免**:
```
GET  /api/getFilterOptions  # 动词命名
POST /api/doRefresh         # 动词命名
```

### HTTP 方法语义

| 方法 | 语义 | 示例 |
|------|------|------|
| GET | 获取资源 | `/api/health` |
| POST | 创建/复杂查询 | `/api/kpi-windows` |

**为什么查询用 POST?**
- 请求体包含复杂的筛选条件(嵌套对象)
- URL 长度有限，不适合传递复杂参数
- POST 请求体无长度限制，支持 JSON

### URL 层级设计

✅ **扁平化设计**(推荐):
```
/api/kpi-windows
/api/week-comparison
```

❌ **过度嵌套**(避免):
```
/api/data/kpi/windows  # 层级太深
/api/data/week/comparison
```

---

## 📖 七、前端集成示例

### 实际代码位置

**Axios 配置**: [frontend/src/services/api.js](../../frontend/src/services/api.js)

**关键特性**:
- 统一的请求/响应拦截器
- 错误处理和 Toast 通知
- 请求/响应日志(开发环境)

**使用示例**:
```javascript
import { apiClient } from '@/services/api'

// GET 请求
const response = await apiClient.get('/api/health')

// POST 请求
const response = await apiClient.post('/api/kpi-windows', {
  filters: { 三级机构: '达州' },
  date: '2025-11-08'
})

// 统一错误处理由拦截器完成
```

---

## ✅ 八、总结

### 核心要点

1. **统一响应格式**: `{ success: true/false, data/message }`
2. **RESTful 原则**: 资源命名、HTTP 方法语义
3. **参数验证**: 类型校验、默认值、友好错误
4. **错误处理**: 区分 4xx/5xx
5. **测试方法**: cURL + pytest

### API 端点速查表

| 端点 | 方法 | 功能 | 必填参数 |
|------|------|------|----------|
| `/api/health` | GET | 健康检查 | - |
| `/api/latest-date` | GET | 最新日期 | - |
| `/api/refresh` | POST | 刷新数据 | - |
| `/api/filter-options` | GET | 筛选选项 | - |
| `/api/kpi-windows` | POST | KPI 三口径 | - |
| `/api/week-comparison` | POST | 周对比 | `metric` |
| `/api/policy-mapping` | GET | 保单映射 | - |
| `/api/staff-performance-distribution` | POST | 业绩分布 | `period` |

---

### Token 节省估算

- **每次对话节省**: 2000-3000 tokens
- **年使用次数**: 约 40 次
- **年总节省**: 80,000 - 120,000 tokens

---

### 适用场景

✅ **适用**:
- 新增 API 端点设计
- API 参数验证逻辑
- 错误响应格式
- API 测试与调试
- 前端集成

❌ **不适用**:
- 数据处理逻辑 → `backend-data-processor`
- 业务逻辑查询 → `analyzing-auto-insurance-data`
- 前端组件开发 → `vue-component-dev`

---

### 关键代码位置

- [backend/api_server.py](../../backend/api_server.py) - Flask API 服务器
  - [L20-40](../../backend/api_server.py#L20-L40): 数据刷新
  - [L96-111](../../backend/api_server.py#L96-L111): 最新日期
  - [L205-241](../../backend/api_server.py#L205-L241): KPI 三口径

- [frontend/src/services/api.js](../../frontend/src/services/api.js) - API 客户端

### 相关 Skills

- [backend-data-processor](../backend-data-processor/SKILL.md) - 后端数据处理
- [analyzing-auto-insurance-data](../analyzing-auto-insurance-data/SKILL.md) - 数据分析
- [testing-and-debugging](../testing-and-debugging/SKILL.md) - 测试与调试

---

**文档维护者**: Claude Code AI Assistant
**版本**: v2.0 (重构版)
**最后更新**: 2025-11-08
**下次审查**: 2025-11-22
