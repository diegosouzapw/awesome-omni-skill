---
name: qweather-now
description: 从和风天气API获取天气数据，包括实时天气、15天预报、气象灾害预警、空气质量、生活指数等。当用户需要查询天气信息、气象预警、空气质量或提到和风天气API时使用。
---

# 和风天气综合查询

从和风天气 API 获取多种气象数据：实时天气、15天预报、气象灾害预警、空气质量、生活指数。

## 快速开始

使用 `get_weather.py` 脚本查询天气数据：

```bash
python scripts/get_weather.py <type> <location> [options]
```

**参数说明**:
- `type`: 查询类型（now/forecast/warning/air/indices）
- `location`: 城市ID（如 `101010100`）或经纬度（如 `116.41,39.92`）
- `options`: 可选参数（仅生活指数需要）

## 前置要求

### 1. 获取 API Key

1. 访问 [和风天气开发平台](https://dev.qweather.com/)
2. 注册并创建应用
3. 获取 API Key（支持免费版和付费版）

### 2. 配置环境变量

将 API Key 添加到环境变量：

```bash
export QWEATHER_API_KEY="your_api_key_here"
```

建议将此行添加到 `~/.bashrc` 或 `~/.zshrc` 以持久保存。

### 3. 安装依赖

```bash
pip install requests
```

## 使用示例

### 1. 实时天气 (now)

查询当前天气状况：

```bash
python scripts/get_weather.py now 101010100
```

输出示例：
```json
{
  "type": "实时天气",
  "updateTime": "2026-01-30T14:35+08:00",
  "temperature": "5°C",
  "feelsLike": "2°C",
  "condition": "多云",
  "humidity": "45%",
  "windDir": "西北风",
  "windScale": "3级",
  "windSpeed": "18 km/h",
  "pressure": "1020 hPa",
  "visibility": "16 km"
}
```

### 2. 15天天气预报 (forecast)

查询未来15天天气：

```bash
python scripts/get_weather.py forecast 101010100
```

输出示例：
```json
{
  "type": "15天天气预报",
  "updateTime": "2026-01-30T14:00+08:00",
  "forecast": [
    {
      "date": "2026-01-30",
      "tempMax": "8°C",
      "tempMin": "2°C",
      "textDay": "多云",
      "textNight": "晴",
      "windDirDay": "西北风",
      "windScaleDay": "3级",
      "humidity": "45%",
      "precip": "0.0 mm",
      "uvIndex": "3"
    }
  ]
}
```

### 3. 气象灾害预警 (warning)

查询气象预警信息：

```bash
python scripts/get_weather.py warning 101010100
```

输出示例：
```json
{
  "type": "气象灾害预警",
  "updateTime": "2026-01-30T14:00+08:00",
  "warningCount": 1,
  "warnings": [
    {
      "title": "北京市气象台发布大风蓝色预警",
      "severity": "Minor",
      "typeName": "大风",
      "pubTime": "2026-01-30T10:00+08:00",
      "text": "预计30日14时至31日14时，北京市大部分地区将有4-5级偏北风..."
    }
  ]
}
```

如果无预警，返回：
```json
{
  "type": "气象灾害预警",
  "warningCount": 0,
  "warnings": "当前无预警信息"
}
```

### 4. 空气质量 (air)

查询空气质量指数：

```bash
python scripts/get_weather.py air 101010100
```

输出示例：
```json
{
  "type": "空气质量",
  "updateTime": "2026-01-30T14:00+08:00",
  "aqi": "85",
  "level": "3",
  "category": "轻度污染",
  "primary": "PM2.5",
  "pm10": "100",
  "pm2p5": "65",
  "no2": "45",
  "so2": "8",
  "co": "0.7",
  "o3": "80"
}
```

### 5. 生活指数 (indices)

查询生活指数（默认：运动、洗车、穿衣、紫外线、感冒、防晒）：

```bash
python scripts/get_weather.py indices 101010100
```

指定特定指数类型：

```bash
python scripts/get_weather.py indices 101010100 --type 1,2,3
```

输出示例：
```json
{
  "type": "生活指数",
  "updateTime": "2026-01-30T08:00+08:00",
  "indices": [
    {
      "type": "1",
      "name": "运动指数",
      "level": "2",
      "category": "较适宜",
      "text": "天气较好，但考虑风力较大，推荐进行室内运动"
    },
    {
      "type": "3",
      "name": "穿衣指数",
      "level": "4",
      "category": "较冷",
      "text": "建议着厚外套加毛衣等服装"
    }
  ]
}
```

**可用的生活指数类型**：

| 代码 | 名称 | 代码 | 名称 |
|------|------|------|------|
| 1 | 运动指数 | 9 | 感冒指数 |
| 2 | 洗车指数 | 10 | 空气污染扩散条件指数 |
| 3 | 穿衣指数 | 11 | 空调开启指数 |
| 5 | 紫外线指数 | 13 | 化妆指数 |
| 6 | 旅游指数 | 14 | 晾晒指数 |
| 7 | 花粉过敏指数 | 16 | 防晒指数 |

## 在代码中使用

```python
import subprocess
import json

# 查询实时天气
result = subprocess.run(
    ["python", "~/.qoder/skills/qweather-now/scripts/get_weather.py", "now", "101010100"],
    capture_output=True,
    text=True
)

weather = json.loads(result.stdout)
print(f"当前温度: {weather['temperature']}")
print(f"天气状况: {weather['condition']}")

# 查询15天预报
result = subprocess.run(
    ["python", "~/.qoder/skills/qweather-now/scripts/get_weather.py", "forecast", "101010100"],
    capture_output=True,
    text=True
)

forecast = json.loads(result.stdout)
for day in forecast['forecast'][:3]:  # 显示未来3天
    print(f"{day['date']}: {day['textDay']}, {day['tempMin']}-{day['tempMax']}")
```

## 城市 ID 查询

常用城市 ID：
- 北京: `101010100`
- 上海: `101020100`
- 广州: `101280101`
- 深圳: `101280601`
- 成都: `101270101`
- 杭州: `101210101`

完整城市列表：[和风天气城市列表](https://github.com/qwd/LocationList/blob/master/China-City-List-latest.csv)

## 查询类型对比

| 类型 | 命令 | 用途 | 主要字段 |
|------|------|------|---------|
| now | `now <location>` | 实时天气 | temperature, condition, humidity, wind |
| forecast | `forecast <location>` | 15天预报 | tempMax, tempMin, textDay, precip |
| warning | `warning <location>` | 气象预警 | severity, typeName, title, text |
| air | `air <location>` | 空气质量 | aqi, category, pm2p5, pm10 |
| indices | `indices <location> --type x,y` | 生活指数 | name, category, text |

## 错误处理

脚本会处理常见错误：

- **API Key 未配置**: 提示设置 `QWEATHER_API_KEY` 环境变量
- **无效的查询类型**: 显示支持的类型列表
- **无效的 location**: 返回错误信息并提示正确格式
- **API 请求失败**: 显示 HTTP 状态码和错误详情
- **网络连接问题**: 提示检查网络连接

## 配置选项

### 语言设置

编辑 `scripts/get_weather.py` 中的 `LANGUAGE` 变量：

```python
LANGUAGE = "zh"  # 中文（默认）
LANGUAGE = "en"  # 英文
```

### 单位制设置

编辑 `scripts/get_weather.py` 中的 `UNIT` 变量：

```python
UNIT = "m"  # 公制单位（默认）- 摄氏度、km/h
UNIT = "i"  # 英制单位 - 华氏度、mph
```

## 故障排查

### 返回 401 错误

**原因**: API Key 无效或未授权

**解决**:
1. 检查环境变量: `echo $QWEATHER_API_KEY`
2. 确认 API Key 在和风天气控制台中有效
3. 检查订阅状态是否正常

### 返回 404 错误

**原因**: Location 参数无效

**解决**:
1. 使用正确的城市 ID 格式（9位数字）
2. 使用正确的经纬度格式（`经度,纬度`）
3. 参考城市列表确认 ID 正确

### 返回 402 错误

**原因**: 超过访问次数或余额不足

**解决**:
1. 检查 API 调用次数限制
2. 升级订阅计划或等待配额重置

### 空气质量查询失败

**原因**: 部分城市 ID 可能不支持空气质量查询

**解决**:
1. 尝试使用经纬度代替城市 ID
2. 确认该地区有空气质量监测站

## API 使用限制

和风天气 API 有不同的订阅等级：

- **免费版**: 每天 1,000 次请求
- **标准版**: 每天 10,000+ 次请求
- **专业版**: 更高限额和完整功能

查看 [和风天气定价](https://dev.qweather.com/docs/finance/subscription/) 了解详情。

## 参考资料

- [和风天气开发文档](https://dev.qweather.com/docs/api/)
- [实时天气 API](https://dev.qweather.com/docs/api/weather/weather-now/)
- [天气预报 API](https://dev.qweather.com/docs/api/weather/weather-daily-forecast/)
- [气象预警 API](https://dev.qweather.com/docs/api/warning/weather-warning/)
- [空气质量 API](https://dev.qweather.com/docs/api/air-quality/)
- [生活指数 API](https://dev.qweather.com/docs/api/indices/)
- [城市列表下载](https://github.com/qwd/LocationList)
