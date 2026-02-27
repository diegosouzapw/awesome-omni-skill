---
name: wecom-bot
description: 企业微信群机器人消息推送 Skill。支持发送文本、Markdown、图片、图文、文件等多种消息类型到企业微信群。
homepage: https://developer.work.weixin.qq.com/document/path/99398
metadata: {"clawdbot":{"emoji":"🤖"}}
---

# 企业微信群机器人

通过 Webhook 向企业微信群推送消息，支持文本、Markdown、图片、图文、文件、模板卡片等多种消息类型。

> **文档来源**：https://developer.work.weixin.qq.com/document/path/99398

## ⚠️ 重要提示

### 使用前提
1. 需要先在企业微信群中添加群机器人，获取 Webhook 地址
2. Webhook 地址格式：`https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=YOUR_KEY`
3. 每个机器人的 `key` 是唯一的，请妥善保管

### 消息限制
- 每个机器人发送的消息不能超过 20 条/分钟
- 文本消息最长 2048 字节
- 图片大小不超过 2MB
- Markdown 不支持 @提及用户

## Setup 配置

### 1. 创建群机器人
1. 在企业微信 PC 端或手机端，进入目标群聊
2. 点击群设置 → 群机器人 → 添加机器人
3. 设置机器人名称和头像
4. 复制生成的 Webhook 地址

### 2. 存储凭证

凭证配置支持以下方式（按优先级排序）：

#### 方式一：环境变量（最高优先级）
```bash
export WECOM_BOT_WEBHOOK="https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=YOUR_KEY"
```

#### 方式二：clawdbot 配置文件（推荐）
在 `~/.clawdbot/clawdbot.json` 中配置：
```json
{
  "skills": {
    "entries": {
      "wecom-bot": {
        "env": {
          "WECOM_BOT_WEBHOOK": "https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=YOUR_KEY"
        }
      }
    }
  }
}
```

#### 方式三：多机器人配置
如果需要向多个群发送消息，可以配置多个 Webhook：
```json
{
  "skills": {
    "entries": {
      "wecom-bot": {
        "env": {
          "WECOM_BOT_WEBHOOK": "https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=DEFAULT_KEY",
          "WECOM_BOT_WEBHOOK_ALERT": "https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=ALERT_KEY",
          "WECOM_BOT_WEBHOOK_DAILY": "https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=DAILY_KEY"
        }
      }
    }
  }
}
```

> **安全提示**：Webhook 地址包含 key，泄露后可被他人恶意调用，请妥善保管

---

## 消息类型

### 1. 文本消息

最基础的消息类型，支持 @提及群成员。

**请求示例**：
```bash
curl -X POST "$WECOM_BOT_WEBHOOK" \
  -H "Content-Type: application/json" \
  -d '{
    "msgtype": "text",
    "text": {
      "content": "Hello, 这是一条测试消息"
    }
  }'
```

**@提及用户**：
```bash
curl -X POST "$WECOM_BOT_WEBHOOK" \
  -H "Content-Type: application/json" \
  -d '{
    "msgtype": "text",
    "text": {
      "content": "请查看今日报告",
      "mentioned_list": ["userid1", "userid2", "@all"],
      "mentioned_mobile_list": ["13800138000"]
    }
  }'
```

**参数说明**：
| 参数 | 必填 | 说明 |
|------|------|------|
| content | 是 | 文本内容，最长 2048 字节 |
| mentioned_list | 否 | userid 列表，`@all` 表示提醒所有人 |
| mentioned_mobile_list | 否 | 手机号列表，提醒群中的指定成员 |

---

### 2. Markdown 消息

支持 Markdown 格式的富文本消息。

**请求示例**：
```bash
curl -X POST "$WECOM_BOT_WEBHOOK" \
  -H "Content-Type: application/json" \
  -d '{
    "msgtype": "markdown",
    "markdown": {
      "content": "# 今日 AI 论文速递\n\n**新增论文**：<font color=\"info\">12 篇</font>\n\n> 重点关注：GPT-5 架构分析\n\n[点击查看详情](https://example.com)"
    }
  }'
```

**支持的 Markdown 语法**：
- 标题：`# 一级标题` `## 二级标题` ... `###### 六级标题`
- 加粗：`**粗体**`
- 链接：`[链接文字](URL)`
- 引用：`> 引用内容`
- 字体颜色：
  - `<font color="info">绿色</font>`
  - `<font color="comment">灰色</font>`
  - `<font color="warning">橙红色</font>`

> ⚠️ **注意**：Markdown 消息不支持 @提及用户

---

### 3. 图片消息

发送图片（需 base64 编码）。

**请求示例**：
```bash
# 先将图片转为 base64
IMAGE_BASE64=$(base64 -i image.png)
IMAGE_MD5=$(md5 -q image.png)  # macOS
# IMAGE_MD5=$(md5sum image.png | cut -d ' ' -f1)  # Linux

curl -X POST "$WECOM_BOT_WEBHOOK" \
  -H "Content-Type: application/json" \
  -d "{
    \"msgtype\": \"image\",
    \"image\": {
      \"base64\": \"$IMAGE_BASE64\",
      \"md5\": \"$IMAGE_MD5\"
    }
  }"
```

**参数说明**：
| 参数 | 必填 | 说明 |
|------|------|------|
| base64 | 是 | 图片内容的 base64 编码 |
| md5 | 是 | 图片内容的 MD5 值 |

> **限制**：图片大小不超过 2MB，支持 JPG、PNG 格式

---

### 4. 图文消息

类似公众号图文链接，支持点击跳转。

**请求示例**：
```bash
curl -X POST "$WECOM_BOT_WEBHOOK" \
  -H "Content-Type: application/json" \
  -d '{
    "msgtype": "news",
    "news": {
      "articles": [
        {
          "title": "今日 AI 论文精选",
          "description": "GPT-5 架构深度解析，附完整技术分析",
          "url": "https://example.com/article/1",
          "picurl": "https://example.com/images/cover.png"
        }
      ]
    }
  }'
```

**多条图文**：
```bash
curl -X POST "$WECOM_BOT_WEBHOOK" \
  -H "Content-Type: application/json" \
  -d '{
    "msgtype": "news",
    "news": {
      "articles": [
        {
          "title": "论文一：Scaling Law 新发现",
          "description": "MIT 最新研究揭示大模型训练规律",
          "url": "https://example.com/1",
          "picurl": "https://example.com/1.png"
        },
        {
          "title": "论文二：多模态融合新方法",
          "description": "Google DeepMind 发布新架构",
          "url": "https://example.com/2",
          "picurl": "https://example.com/2.png"
        }
      ]
    }
  }'
```

**参数说明**：
| 参数 | 必填 | 说明 |
|------|------|------|
| title | 是 | 标题，不超过 128 字节 |
| description | 否 | 描述，不超过 512 字节 |
| url | 是 | 点击后跳转的链接 |
| picurl | 否 | 图文消息的图片链接，支持 JPG、PNG |

> **限制**：支持 1-8 条图文

---

### 5. 文件消息

发送文件（需先上传获取 media_id）。

**Step 1：上传文件**
```bash
curl -X POST "https://qyapi.weixin.qq.com/cgi-bin/webhook/upload_media?key=YOUR_KEY&type=file" \
  -F "media=@report.pdf"
```

返回示例：
```json
{
  "errcode": 0,
  "errmsg": "ok",
  "type": "file",
  "media_id": "3a8asd892asd8asd",
  "created_at": "1380000000"
}
```

**Step 2：发送文件**
```bash
curl -X POST "$WECOM_BOT_WEBHOOK" \
  -H "Content-Type: application/json" \
  -d '{
    "msgtype": "file",
    "file": {
      "media_id": "3a8asd892asd8asd"
    }
  }'
```

> **限制**：文件大小不超过 20MB

---

### 6. 模板卡片消息

更丰富的交互卡片，支持多种布局。

**文本通知型卡片**：
```bash
curl -X POST "$WECOM_BOT_WEBHOOK" \
  -H "Content-Type: application/json" \
  -d '{
    "msgtype": "template_card",
    "template_card": {
      "card_type": "text_notice",
      "source": {
        "desc": "AI 论文助手",
        "desc_color": 0
      },
      "main_title": {
        "title": "今日论文速递",
        "desc": "2025年1月31日"
      },
      "emphasis_content": {
        "title": "12",
        "desc": "篇新论文"
      },
      "sub_title_text": "涵盖 NLP、CV、多模态等领域",
      "horizontal_content_list": [
        {"keyname": "重点领域", "value": "大语言模型"},
        {"keyname": "来源", "value": "arXiv"}
      ],
      "jump_list": [
        {
          "type": 1,
          "url": "https://example.com/papers",
          "title": "查看详情"
        }
      ],
      "card_action": {
        "type": 1,
        "url": "https://example.com/papers"
      }
    }
  }'
```

**卡片类型**：
| card_type | 说明 |
|-----------|------|
| text_notice | 文本通知型 |
| news_notice | 图文展示型 |
| button_interaction | 按钮交互型 |
| vote_interaction | 投票选择型 |

---

## 返回结果

**成功**：
```json
{
  "errcode": 0,
  "errmsg": "ok"
}
```

**常见错误码**：
| errcode | 说明 | 解决方案 |
|---------|------|---------|
| 0 | 成功 | - |
| 93000 | Webhook key 不正确 | 检查 Webhook URL 中的 key |
| 93004 | 机器人被移出群 | 重新添加机器人到群 |
| 93008 | 不合法的消息类型 | 检查 msgtype 参数 |
| 93017 | 参数错误 | 检查请求体格式 |
| 45033 | 消息发送频率超限 | 降低发送频率（上限 20 条/分钟） |

---

## 常用场景示例

### 场景一：定时发送日报
```bash
# 每天早上 9 点发送 AI 论文日报
curl -X POST "$WECOM_BOT_WEBHOOK" \
  -H "Content-Type: application/json" \
  -d '{
    "msgtype": "markdown",
    "markdown": {
      "content": "# 📚 AI 论文日报\n\n**日期**：2025-01-31\n\n## 今日精选\n\n1. **GPT-5 架构分析** - OpenAI\n   > 揭示下一代大模型的技术路线\n\n2. **多模态推理新范式** - Google\n   > 图文联合推理效果提升 30%\n\n[点击查看完整报告](https://example.com)"
    }
  }'
```

### 场景二：监控告警
```bash
# 服务异常时发送告警
curl -X POST "$WECOM_BOT_WEBHOOK" \
  -H "Content-Type: application/json" \
  -d '{
    "msgtype": "text",
    "text": {
      "content": "⚠️ 服务告警\n\n服务名：API Gateway\n状态：响应超时\n时间：2025-01-31 14:30:00\n\n请相关同事尽快处理！",
      "mentioned_list": ["@all"]
    }
  }'
```

### 场景三：竞品动态推送
```bash
curl -X POST "$WECOM_BOT_WEBHOOK" \
  -H "Content-Type: application/json" \
  -d '{
    "msgtype": "news",
    "news": {
      "articles": [
        {
          "title": "🔍 竞品动态：XX 产品发布新版本",
          "description": "新增 AI 助手功能，支持智能问答和自动摘要",
          "url": "https://example.com/competitor-update",
          "picurl": "https://example.com/competitor.png"
        }
      ]
    }
  }'
```

---

## 一键发送脚本

创建便捷脚本，快速发送消息：

```bash
cat > ~/.config/wecom-bot/send.sh << 'EOF'
#!/bin/bash
# 企业微信群机器人消息发送脚本
# 用法: send.sh "消息内容" [markdown|text]

WEBHOOK="${WECOM_BOT_WEBHOOK}"
CONTENT="$1"
TYPE="${2:-text}"

if [ -z "$WEBHOOK" ]; then
  echo "错误: 未配置 WECOM_BOT_WEBHOOK 环境变量"
  exit 1
fi

if [ -z "$CONTENT" ]; then
  echo "用法: $0 \"消息内容\" [markdown|text]"
  exit 1
fi

if [ "$TYPE" = "markdown" ]; then
  curl -s -X POST "$WEBHOOK" \
    -H "Content-Type: application/json" \
    -d "{\"msgtype\":\"markdown\",\"markdown\":{\"content\":\"$CONTENT\"}}"
else
  curl -s -X POST "$WEBHOOK" \
    -H "Content-Type: application/json" \
    -d "{\"msgtype\":\"text\",\"text\":{\"content\":\"$CONTENT\"}}"
fi
EOF

chmod +x ~/.config/wecom-bot/send.sh
```

使用方式：
```bash
# 发送文本
~/.config/wecom-bot/send.sh "Hello, 这是一条测试消息"

# 发送 Markdown
~/.config/wecom-bot/send.sh "# 标题\n内容" markdown
```

---

## 与其他工具集成

### 配合 Clawdbot 定时任务
```yaml
# 每天早上 9 点推送 AI 日报到企微群
name: ai-daily-report
schedule: "0 9 * * *"
tasks:
  - name: 抓取论文
    action: fetch_arxiv
  - name: AI 总结
    action: summarize
  - name: 推送企微
    action: wecom_notify
    webhook: $WECOM_BOT_WEBHOOK
```

### 配合乐享知识库
抓取信息 → 存入乐享 → 推送摘要到企微群，形成完整的信息流转闭环。
