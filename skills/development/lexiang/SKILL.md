---
name: lexiang
description: 腾讯乐享知识库 API 接口文档。包含通讯录管理、团队管理、知识库管理、知识节点管理、任务管理、自定义属性管理、操作日志、AI助手、单点登录、素材管理、导出任务管理等接口。
homepage: https://lexiang.tencent.com/wiki/api/
metadata: {"clawdbot":{"emoji":"📚"}}
---

# 腾讯乐享知识库 API

腾讯乐享知识库是企业级知识管理平台，提供知识库、团队协作、文档管理、AI助手等功能。

> **文档来源**：https://lexiang.tencent.com/wiki/api/

## 📊 数据模型

### 核心概念
- **Team（团队）**：顶级组织单元，一个团队下可以有多个知识库（Space）
- **Space（知识库）**：知识的容器，属于某个团队，包含多个条目（Entry），有 `root_entry_id` 作为根节点
- **Entry（条目/知识）**：知识库中的内容单元，可以是页面（page）、文件夹（folder）或文件（file），支持树形结构（parent_id）
- **File（文件）**：附件类型的条目，如 PDF、Word、图片等

### 层级关系
```
Team -> Space -> Entry（树形结构，root_entry_id 为根）
```

## 🔗 URL 规则

### 域名格式
- **{domain}**：企业专属域名，如 `csig.lexiangla.com`
- 可通过 API 返回的数据或企业配置获取

### URL 构建规则
| 资源类型 | URL 格式 | 示例 |
|---------|----------|------|
| 团队首页 | `https://{domain}/t/{team_id}/spaces` | `https://csig.lexiangla.com/t/abc123/spaces` |
| 知识库 | `https://{domain}/spaces/{space_id}` | `https://csig.lexiangla.com/spaces/xyz789` |
| 知识条目（页面） | `https://{domain}/pages/{entry_id}` | `https://csig.lexiangla.com/pages/def456` |

### ⚠️ 注意事项
- **不要使用** `https://lexiang.tencent.com/wiki/{entry_id}` 格式，这种格式无法正常访问
- **正确格式**：使用企业专属域名 + `/pages/{entry_id}` 或 `/spaces/{space_id}`
- 返回知识库链接时，优先使用 API 响应中的 `links` 字段（如果存在）
- 如果 API 未返回完整链接，请根据上述规则拼接

---

## ⚠️ 重要提示

### 请求格式说明
- **创建/更新知识节点**：使用 **JSON:API 规范格式**，通过 `relationships` 指定 `space` 和 `parent_entry`
- **获取线上文档内容**：需要添加 `?content_type=html` 参数
- **AI 接口（搜索/问答）**：需要 `x-staff-id` 头部

### 需要 x-staff-id 的接口
以下接口需要在请求头中添加 `x-staff-id`：
- 所有写操作（创建、更新、删除）
- AI 搜索 (`/v1/ai/search`)
- AI 问答 (`/v1/ai/qa`)
- 团队/知识库/节点权限设置

### 常见错误
| 错误信息 | 原因 | 解决方案 |
|----------|------|---------|
| `必须指定员工账号` | 缺少 x-staff-id | 添加 `-H "x-staff-id: $LEXIANG_STAFF_ID"` |
| `data.attributes.entry_type 不能为空` | 请求格式错误 | 使用 JSON:API 格式 |
| `content_type 不能为空` | 缺少参数 | 添加 `?content_type=html` |

## Setup 配置

### 1. 获取凭证
1. 以管理员身份登录乐享知识库企业管理后台
2. 进入【开发】→【接口凭证管理】→ 点击**添加凭证**
3. 保存 `AppKey` 和 `AppSecret`（初始化后仅显示一次，请妥善保管）

### 2. 配置权限
- 修改凭证权限，勾选当前 AppKey 允许调用的接口模块
- 设置知识授权范围（默认为公司内所有知识；若指定团队，则团队管理、知识库管理等接口的读写范围将受限）

### 3. 存储凭证

凭证配置支持以下方式（按优先级排序）：

#### 方式一：环境变量（最高优先级）
```bash
export LEXIANG_APP_KEY="your_app_key"
export LEXIANG_APP_SECRET="your_app_secret"
export LEXIANG_STAFF_ID="your_staff_id"  # 用于写操作的身份标识
```

#### 方式二：clawdbot 配置文件（推荐）
在 `~/.clawdbot/clawdbot.json` 中使用 `env` 字段配置：
```json
{
  "skills": {
    "entries": {
      "lexiang": {
        "env": {
          "LEXIANG_APP_KEY": "your_app_key",
          "LEXIANG_APP_SECRET": "your_app_secret",
          "LEXIANG_STAFF_ID": "your_staff_id"
        }
      }
    }
  }
}
```

> **关于 LEXIANG_STAFF_ID**：
> - 这是进行写操作（创建/更新/删除文档等）时必需的身份标识
> - 对应乐享通讯录中的员工账号（staff_id）
> - 该员工需要具有对应知识库/团队的编辑权限
> - 获取方式：在乐享后台【设置】→【通讯录】中查看自己的员工账号

#### 方式三：独立配置文件（兼容旧版）
```bash
mkdir -p ~/.config/lexiang
echo '{"app_key":"your_app_key","app_secret":"your_app_secret","staff_id":"your_staff_id"}' > ~/.config/lexiang/credentials
chmod 600 ~/.config/lexiang/credentials
```

> **安全提示**：凭证文件权限设为 600，仅本人可读写

### 4. 一键初始化脚本（可选）
创建初始化脚本以便快速配置：
```bash
cat > ~/.config/lexiang/init.sh << 'EOF'
#!/bin/bash
# 加载乐享凭证和获取 Token
# 优先级：环境变量 > clawdbot.json (env) > ~/.config/lexiang/credentials

# 1. 检查环境变量（clawdbot 会自动从 env 字段注入）
if [ -n "$LEXIANG_APP_KEY" ] && [ -n "$LEXIANG_APP_SECRET" ]; then
  echo "使用环境变量中的凭证"
  # 从环境变量加载 staff_id（如果有）
  if [ -z "$LEXIANG_STAFF_ID" ]; then
    # 尝试从 clawdbot.json 获取 staff_id
    if [ -f ~/.clawdbot/clawdbot.json ]; then
      STAFF_ID=$(jq -r '.skills.entries.lexiang.env.LEXIANG_STAFF_ID // empty' ~/.clawdbot/clawdbot.json 2>/dev/null)
      if [ -n "$STAFF_ID" ]; then
        export LEXIANG_STAFF_ID="$STAFF_ID"
      fi
    fi
  fi

# 2. 检查 clawdbot.json 的 env 配置
elif [ -f ~/.clawdbot/clawdbot.json ]; then
  APP_KEY=$(jq -r '.skills.entries.lexiang.env.LEXIANG_APP_KEY // empty' ~/.clawdbot/clawdbot.json 2>/dev/null)
  APP_SECRET=$(jq -r '.skills.entries.lexiang.env.LEXIANG_APP_SECRET // empty' ~/.clawdbot/clawdbot.json 2>/dev/null)
  STAFF_ID=$(jq -r '.skills.entries.lexiang.env.LEXIANG_STAFF_ID // empty' ~/.clawdbot/clawdbot.json 2>/dev/null)
  
  if [ -n "$APP_KEY" ] && [ -n "$APP_SECRET" ]; then
    export LEXIANG_APP_KEY="$APP_KEY"
    export LEXIANG_APP_SECRET="$APP_SECRET"
    if [ -n "$STAFF_ID" ]; then
      export LEXIANG_STAFF_ID="$STAFF_ID"
    fi
    echo "使用 ~/.clawdbot/clawdbot.json 中的凭证"
  fi

# 3. 检查独立配置文件
elif [ -f ~/.config/lexiang/credentials ]; then
  LEXIANG_CREDS=$(cat ~/.config/lexiang/credentials)
  export LEXIANG_APP_KEY=$(echo $LEXIANG_CREDS | jq -r '.app_key')
  export LEXIANG_APP_SECRET=$(echo $LEXIANG_CREDS | jq -r '.app_secret')
  STAFF_ID=$(echo $LEXIANG_CREDS | jq -r '.staff_id // empty')
  if [ -n "$STAFF_ID" ]; then
    export LEXIANG_STAFF_ID="$STAFF_ID"
  fi
  echo "使用 ~/.config/lexiang/credentials 中的凭证"
fi

# 检查凭证是否已加载
if [ -z "$LEXIANG_APP_KEY" ] || [ -z "$LEXIANG_APP_SECRET" ]; then
  echo "错误：未找到乐享凭证，请配置以下任一方式："
  echo "  1. 设置环境变量 LEXIANG_APP_KEY 和 LEXIANG_APP_SECRET"
  echo "  2. 在 ~/.clawdbot/clawdbot.json 中配置 skills.entries.lexiang.env"
  echo "  3. 创建 ~/.config/lexiang/credentials 文件"
  return 1 2>/dev/null || exit 1
fi

# 检查 staff_id 是否已配置（警告，非错误）
if [ -z "$LEXIANG_STAFF_ID" ]; then
  echo "警告：未配置 LEXIANG_STAFF_ID，写操作可能会失败"
  echo "  请在配置中添加 LEXIANG_STAFF_ID（员工账号）"
else
  echo "员工身份标识：$LEXIANG_STAFF_ID"
fi

# 检查是否有缓存的有效 token
if [ -f ~/.config/lexiang/token ]; then
  TOKEN_AGE=$(($(date +%s) - $(stat -f %m ~/.config/lexiang/token 2>/dev/null || stat -c %Y ~/.config/lexiang/token)))
  if [ $TOKEN_AGE -lt 7000 ]; then
    export LEXIANG_TOKEN=$(cat ~/.config/lexiang/token)
    echo "使用缓存的 Token (剩余有效期: $((7200 - TOKEN_AGE))秒)"
    return 0 2>/dev/null || exit 0
  fi
fi

# 获取新 token
export LEXIANG_TOKEN=$(curl -s -X POST "https://lxapi.lexiangla.com/cgi-bin/token" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d "{\"grant_type\":\"client_credentials\",\"app_key\":\"$LEXIANG_APP_KEY\",\"app_secret\":\"$LEXIANG_APP_SECRET\"}" \
  | jq -r '.access_token')

mkdir -p ~/.config/lexiang
echo $LEXIANG_TOKEN > ~/.config/lexiang/token
echo "已获取新 Token 并缓存"
EOF
chmod +x ~/.config/lexiang/init.sh
```

使用方式：
```bash
source ~/.config/lexiang/init.sh
# 然后即可使用 $LEXIANG_TOKEN 和 $LEXIANG_STAFF_ID 变量
```

## API Basics 基础信息

### API Key 配置
- `LEXIANG_APP_KEY` / `LEXIANG_APP_SECRET` / `LEXIANG_STAFF_ID` 环境变量
- 或在 `~/.clawdbot/clawdbot.json` 中设置 `skills.entries.lexiang.env.LEXIANG_APP_KEY` / `skills.entries.lexiang.env.LEXIANG_APP_SECRET` / `skills.entries.lexiang.env.LEXIANG_STAFF_ID`
- 或在 `~/.config/lexiang/credentials` 中存储 JSON 格式凭证

### 加载凭证变量
在所有 API 调用前，先加载凭证和获取 token：
```bash
# 方式一：使用初始化脚本（推荐，自动处理多种配置来源）
source ~/.config/lexiang/init.sh

# 方式二：手动加载（从 clawdbot.json 的 env 配置）
LEXIANG_APP_KEY=$(jq -r '.skills.entries.lexiang.env.LEXIANG_APP_KEY' ~/.clawdbot/clawdbot.json)
LEXIANG_APP_SECRET=$(jq -r '.skills.entries.lexiang.env.LEXIANG_APP_SECRET' ~/.clawdbot/clawdbot.json)
LEXIANG_STAFF_ID=$(jq -r '.skills.entries.lexiang.env.LEXIANG_STAFF_ID' ~/.clawdbot/clawdbot.json)

# 获取 access_token（有效期 2 小时）
LEXIANG_TOKEN=$(curl -s -X POST "https://lxapi.lexiangla.com/cgi-bin/token" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d "{\"grant_type\":\"client_credentials\",\"app_key\":\"$LEXIANG_APP_KEY\",\"app_secret\":\"$LEXIANG_APP_SECRET\"}" \
  | jq -r '.access_token')

echo "Token: $LEXIANG_TOKEN"
echo "Staff ID: $LEXIANG_STAFF_ID"
```

**响应示例：**
```json
{
  "token_type": "Bearer",
  "expires_in": 7200,
  "access_token": "XXXXXXXXX"
}
```

> **注意：**
> - `access_token` 有效期为 2 小时（7200秒）
> - 频率限制：20次/10分钟
> - **必须缓存 token**，避免频繁调用导致触发频率限制被拦截
> - 建议将 token 缓存到文件：`echo $LEXIANG_TOKEN > ~/.config/lexiang/token`

### 快速加载已缓存的 Token
如果已缓存 token，可直接加载：
```bash
LEXIANG_TOKEN=$(cat ~/.config/lexiang/token)
```

### 使用 access_token
所有接口调用需在 Header 中携带：
```bash
curl -X GET "https://lxapi.lexiangla.com/cgi-bin/v1/..." \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "Content-Type: application/json; charset=utf-8"
```

### 身份标识
写操作需要指定员工身份：
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/..." \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8"
```
> **重要**：
> - `x-staff-id` 是进行写操作（创建、更新、删除等）的必需参数
> - 建议使用配置的 `$LEXIANG_STAFF_ID` 环境变量
> - 该员工账号需要具有目标资源的相应权限（如知识库的编辑权限）
> - 如果需要忽略权限校验（仅在特定场景下），可使用 `x-staff-id: system-bot`

### 通用限制
- **频率限制**：大部分接口为 3000次/分钟
- **权限要求**：需在 AppKey 的授权范围内

---

## 一、通讯录管理

### 1.1 成员管理

#### 创建成员
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/contact/user/create" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "staff_id": "zhangsan",
    "name": "张三",
    "phone": "13800138000",
    "email": "zhangsan@example.com",
    "department": [1, 2],
    "position": "产品经理"
  }'
```
**参数：**
| 参数 | 必须 | 说明 |
|------|------|------|
| staff_id | 是 | 员工账号，企业内唯一 |
| name | 是 | 成员名称 |
| phone | 是 | 手机号码，企业内唯一 |
| email | 否 | 邮箱 |
| department | 否 | 部门ID列表 |
| is_leader_in_dept | 否 | 是否为部门领导（数组，对应 department） |
| direct_leader | 否 | 直属上级（staff_id 数组） |
| main_depart | 否 | 主部门ID |
| position | 否 | 职务 |
| extra_attr | 否 | 自定义字段 |

> **频率限制**：3000次/分钟

#### 更新成员
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/contact/user/update" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"staff_id": "zhangsan", "name": "张三丰"}'
```
> **注意**：成员激活后，phone 和 email 不可通过接口变更

#### 删除成员（离职）
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/contact/user/resign" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"staff_id": "zhangsan"}'
```
> 删除后无法再使用乐享或接收消息，如需恢复可调用"重新入职"接口

#### 重新入职
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/contact/user/entry" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"staff_id": "zhangsan", "department": [1]}'
```

#### 禁用成员（批量）
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/contact/user/forbidden" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"staffs": ["staff1", "staff2"]}'
```
> 禁用后无法使用乐享或接收消息，但其他成员仍可见其数据。最多100个

#### 重新启用
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/contact/user/active" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"staffs": ["staff1", "staff2"]}'
```

#### 添加管理员
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/contact/user/add-manager" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"staff_id": "zhangsan"}'
```

#### 获取成员信息
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/contact/user/get?staff_id=zhangsan" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

#### 获取部门下成员
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/contact/user/list?department_id=1&page=1&per_page=100&fetch_child=1" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```
| 参数 | 必须 | 说明 |
|------|------|------|
| department_id | 是 | 部门ID |
| page | 否 | 页码，默认1 |
| per_page | 否 | 每页数量，最大1000 |
| fetch_child | 否 | 1:包含子部门成员, 0:不包含 |

#### 获取管理员列表
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/contact/user/managers" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

#### 获取自定义字段
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/contact/user/extra-attrs" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

#### 导出所有成员信息（异步）
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/contact/export/user" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"aeskey": "your_32_char_aes_key_here_12345"}'
```
> - `aeskey` 为32位加密密钥（a-z, A-Z, 0-9），用于解密下载文件
> - 频率限制：10次/分钟
> - 需轮询查询任务状态获取下载链接

### 1.2 部门管理

#### 创建部门
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/contact/department/create" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"name": "研发部", "parent_id": 1, "order": 100}'
```
| 参数 | 必须 | 说明 |
|------|------|------|
| name | 是 | 部门名称 |
| parent_id | 是 | 父部门ID（根部门为1） |
| id | 否 | 指定部门ID（32位整型，>1） |
| order | 否 | 排序值（范围 [0, 2^32)，越大越靠前） |

**常见错误码：**
- 1003: 参数错误
- 1005: 部门名称已存在
- 1007: 部门层级超过15层

#### 更新部门
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/contact/department/edit" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"id": 2, "name": "产品研发部"}'
```

#### 删除部门
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/contact/department/delete" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"id": 2}'
```
> 不能删除根部门、含有子部门或成员的部门

#### 获取子部门
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/contact/department/index?id=1&with_descendant=1" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

#### 获取单个部门信息
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/contact/department/get?id=2" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

#### 导出所有部门信息（异步）
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/contact/export/department" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"aeskey": "your_32_char_aes_key_here_12345"}'
```

---

## 二、团队管理

### 2.1 团队接口

#### 获取团队列表
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/teams?limit=20&page_token=" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

#### 获取团队详情
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/teams/{team_id}" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

#### 设置团队成员与权限
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/teams/{team_id}/subject?downgrade=0" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "data": [
      {"type": "staff", "id": "lisi", "role": "manager"},
      {"type": "department", "id": "123", "role": "editor"}
    ]
  }'
```
| 参数 | 说明 |
|------|------|
| downgrade | 0:仅增加/不降级, 1:允许覆盖权限 |
| type | staff(成员) 或 department(部门) |
| role | manager(管理), editor(编辑), downloader(查看下载), viewer(仅查看) |

#### 移除团队成员与权限
```bash
curl -X DELETE "https://lxapi.lexiangla.com/cgi-bin/v1/kb/teams/{team_id}/subject" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"data": [{"type": "staff", "id": "lisi", "role": "editor"}]}'
```

#### 获取团队成员与权限
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/teams/{team_id}/subject?limit=20" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

---

## 三、知识库管理

### 3.1 知识库接口

#### 创建知识库
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/spaces" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "name": "技术文档库",
    "team": {
      "data": {"type": "team", "id": "team_id_here"}
    },
    "visible_type": 1,
    "subject": [
      {"type": "staff", "id": "lisi", "role": "editor"}
    ]
  }'
```
| 参数 | 必须 | 说明 |
|------|------|------|
| name | 是 | 知识库名称 |
| team.data.id | 是 | 所属团队ID |
| visible_type | 否 | 0:不可见, 1:可见, 2:跟随团队 |
| subject | 否 | 初始化权限设置 |

#### 更新知识库
```bash
curl -X PUT "https://lxapi.lexiangla.com/cgi-bin/v1/kb/spaces/{space_id}" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"name": "新知识库名称"}'
```

#### 删除知识库
```bash
curl -X DELETE "https://lxapi.lexiangla.com/cgi-bin/v1/kb/spaces/{space_id}" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID"
```

#### 获取知识库列表
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/spaces?team_id={team_id}&limit=20" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

#### 获取知识库详情
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/spaces/{space_id}" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

#### 设置知识库成员与权限
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/spaces/{space_id}/subject?downgrade=0" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "data": [
      {"type": "staff", "id": "lisi", "role": "editor"},
      {"type": "department", "id": "123", "role": "viewer"}
    ]
  }'
```

#### 移除知识库成员与权限
```bash
curl -X DELETE "https://lxapi.lexiangla.com/cgi-bin/v1/kb/spaces/{space_id}/subject" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"data": [{"type": "staff", "id": "lisi", "role": "editor"}]}'
```

#### 获取知识库成员与权限
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/spaces/{space_id}/subject?limit=20" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

### 3.2 知识节点接口

#### 创建文件夹
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/entries" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "data": {
      "type": "kb_entry",
      "attributes": {
        "entry_type": "directory",
        "name": "文件夹名称"
      },
      "relationships": {
        "space": {
          "data": {"type": "kb_space", "id": "space_id_here"}
        },
        "parent_entry": {
          "data": {"type": "kb_entry", "id": "parent_entry_id"}
        }
      }
    }
  }'
```
> **注意**：使用 JSON:API 规范格式，`space_id` 和 `parent_entry_id` 需要通过 `relationships` 指定

#### 创建在线文档
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/entries" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "data": {
      "type": "kb_entry",
      "attributes": {
        "entry_type": "page",
        "name": "文档标题"
      },
      "relationships": {
        "space": {
          "data": {"type": "kb_space", "id": "space_id_here"}
        },
        "parent_entry": {
          "data": {"type": "kb_entry", "id": "parent_entry_id"}
        }
      }
    }
  }'
```
> **注意**：使用 JSON:API 规范格式

#### 上传文件
```bash
# 需先调用"本地文件上传"接口获取 state
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/entries?state={STATE}" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "data": {
      "type": "kb_entry",
      "attributes": {
        "entry_type": "file",
        "name": "文件名.pdf"
      },
      "relationships": {
        "space": {
          "data": {"type": "kb_space", "id": "space_id_here"}
        },
        "parent_entry": {
          "data": {"type": "kb_entry", "id": "parent_entry_id"}
        }
      }
    }
  }'
```
> 支持类型：video(视频), audio(音频), file(图片/文档等)

#### 重命名知识节点
```bash
curl -X PUT "https://lxapi.lexiangla.com/cgi-bin/v1/kb/entries/{entry_id}/rename" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"name": "新名称"}'
```

#### 重新上传文件
```bash
curl -X PUT "https://lxapi.lexiangla.com/cgi-bin/v1/kb/entries/{entry_id}/upload?state={STATE}" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID"
```

#### 删除知识节点
```bash
curl -X DELETE "https://lxapi.lexiangla.com/cgi-bin/v1/kb/entries/{entry_id}" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID"
```
> **注意**：删除节点前必须保证其下没有子节点

#### 获取知识列表
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/entries?space_id={space_id}&parent_id={parent_id}&limit=20" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

#### 获取知识详情
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/entries/{entry_id}" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```
> 对于文件类型，响应包含临时 `download` 下载链接（有效60分钟）

#### 获取线上文档内容
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/entries/{entry_id}/content?content_type=html" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```
| 参数 | 必须 | 说明 |
|------|------|------|
| content_type | 是 | 内容格式，可选值：html |

> 仅支持 page 类型，返回 HTML 格式内容

#### 设置知识节点成员与权限
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/entries/{entry_id}/subject?downgrade=0" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "data": [
      {"type": "staff", "id": "lisi", "role": "editor"}
    ]
  }'
```

#### 移除知识节点成员与权限
```bash
curl -X DELETE "https://lxapi.lexiangla.com/cgi-bin/v1/kb/entries/{entry_id}/subject" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"data": [{"type": "staff", "id": "lisi", "role": "editor"}]}'
```

#### 设置知识继承权限
```bash
curl -X PUT "https://lxapi.lexiangla.com/cgi-bin/v1/kb/entries/{entry_id}/inherit" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"member_inherit_type": "inherit"}'
```

#### 获取知识节点成员与权限
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/entries/{entry_id}/subject?limit=20" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

### 3.3 在线文档块接口

#### 创建嵌套块

> **文档参考**: https://lexiang.tencent.com/wiki/api/15016.html

**接口地址**: `POST /cgi-bin/v1/kb/page/entries/{entry_id}/blocks/descendant`

**请求头**:
- `Authorization: Bearer {access_token}`
- `x-staff-id`: 成员账号（作为创建者）
- `Content-Type: application/json; charset=utf-8`

**核心参数说明**:
| 参数 | 类型 | 必须 | 说明 |
|------|------|------|------|
| parent_block_id | String | 否 | 父块 ID。**留空则插入到页面根节点**。如需指定位置，可先调用"获取子块列表"获取已有块的 parent_id |
| index | Int | 否 | 插入位置索引（从 0 开始） |
| children | Array | 嵌套块必填 | 第一级子块的临时 ID 列表 |
| descendant | Array | 是 | 所有待创建块的数组 |

> **💡 提示**：对于新建的空白文档，**不传 parent_block_id** 即可直接插入内容到页面根节点。

**descendant 数组对象结构**:
| 参数 | 说明 |
|------|------|
| block_id | 嵌套块模式必填，自定义临时 ID（字符串），用于建立父子关系 |
| block_type | 块类型：`p`(段落), `h1`-`h5`(标题), `code`(代码), `table`(表格), `table_cell`(表格单元格), `task`(任务), `callout`(高亮块), `toggle`(折叠块), `bulleted_list`(无序列表), `numbered_list`(有序列表), `divider`(分隔线), `column_list`(分栏), `column`(列), `mermaid`, `plantuml` |
| children | 该块包含的子块临时 ID 列表 |
| [内容字段] | 根据 block_type 不同使用不同字段：`text`, `heading2`, `task`, `table`, `table_cell` 等 |

**⚠️ 不支持嵌套子块的类型**:
- 标题块：`h1`, `h2`, `h3`, `h4`, `h5`
- 代码块：`code`
- 媒体块：`image`, `attachment`, `video`
- 功能块：`divider`, `mermaid`, `plantuml`

---

##### 示例 1：创建简单段落
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/page/entries/{entry_id}/blocks/descendant" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "parent_block_id": "页面根块ID",
    "index": 0,
    "descendant": [
      {
        "block_type": "p",
        "text": {
          "elements": [
            {
              "text_run": {
                "content": "这是一段普通文本"
              }
            }
          ]
        }
      }
    ]
  }'
```

##### 示例 2：创建带样式的标题
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/page/entries/{entry_id}/blocks/descendant" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "parent_block_id": "页面根块ID",
    "index": 0,
    "descendant": [
      {
        "block_type": "h2",
        "heading2": {
          "elements": [
            {
              "text_run": {
                "content": "这是一段"
              }
            },
            {
              "text_run": {
                "content": "加粗",
                "text_style": {
                  "bold": true
                }
              }
            },
            {
              "text_run": {
                "content": "和"
              }
            },
            {
              "text_run": {
                "content": "下划线",
                "text_style": {
                  "underline": true
                }
              }
            },
            {
              "text_run": {
                "content": "的H2标题"
              }
            }
          ]
        }
      }
    ]
  }'
```

**text_style 支持的样式**:
| 属性 | 类型 | 说明 |
|------|------|------|
| bold | Boolean | 加粗 |
| italic | Boolean | 斜体 |
| underline | Boolean | 下划线 |
| strikethrough | Boolean | 删除线 |
| code | Boolean | 行内代码 |
| color | String | 文字颜色 |
| background_color | String | 背景颜色 |

##### 示例 3：创建任务块
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/page/entries/{entry_id}/blocks/descendant" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "parent_block_id": "页面根块ID",
    "index": 0,
    "descendant": [
      {
        "block_type": "task",
        "task": {
          "name": "完成 API 文档更新",
          "done": false,
          "due_at": {
            "date": "2025-12-31",
            "time": "18:00:00"
          },
          "assignees": [
            {
              "staff_uuid": "员工UUID"
            }
          ]
        }
      }
    ]
  }'
```

##### 示例 4：创建代码块
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/page/entries/{entry_id}/blocks/descendant" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "parent_block_id": "页面根块ID",
    "index": 0,
    "descendant": [
      {
        "block_type": "code",
        "code": {
          "language": "python",
          "content": "def hello():\n    print(\"Hello, World!\")"
        }
      }
    ]
  }'
```

##### 示例 5：创建引用块
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/page/entries/{entry_id}/blocks/descendant" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "parent_block_id": "页面根块ID",
    "index": 0,
    "children": ["quote-1"],
    "descendant": [
      {
        "block_id": "quote-1",
        "block_type": "quote",
        "children": ["quote-text-1"]
      },
      {
        "block_id": "quote-text-1",
        "block_type": "p",
        "text": {
          "elements": [
            {
              "text_run": {
                "content": "这是一段引用文本"
              }
            }
          ]
        }
      }
    ]
  }'
```

##### 示例 6：创建高亮块 (Callout)
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/page/entries/{entry_id}/blocks/descendant" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "parent_block_id": "页面根块ID",
    "index": 0,
    "children": ["callout-1"],
    "descendant": [
      {
        "block_id": "callout-1",
        "block_type": "callout",
        "callout": {
          "background_color": "#FFF3E0",
          "icon": "⚠️"
        },
        "children": ["callout-text-1"]
      },
      {
        "block_id": "callout-text-1",
        "block_type": "p",
        "text": {
          "elements": [
            {
              "text_run": {
                "content": "这是一个警告提示"
              }
            }
          ]
        }
      }
    ]
  }'
```

##### 示例 7：创建无序列表
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/page/entries/{entry_id}/blocks/descendant" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "parent_block_id": "页面根块ID",
    "index": 0,
    "descendant": [
      {
        "block_type": "bulleted_list",
        "bulleted": {
          "elements": [{"text_run": {"content": "列表项 1"}}]
        }
      },
      {
        "block_type": "bulleted_list",
        "bulleted": {
          "elements": [{"text_run": {"content": "列表项 2"}}]
        }
      },
      {
        "block_type": "bulleted_list",
        "bulleted": {
          "elements": [{"text_run": {"content": "列表项 3"}}]
        }
      }
    ]
  }'
```

> **注意**：无序列表使用 `bulleted_list` 类型，内容字段为 `bulleted`；有序列表使用 `numbered_list` 类型，内容字段为 `numbered`

##### 示例 7b：创建有序列表
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/page/entries/{entry_id}/blocks/descendant" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "parent_block_id": "页面根块ID",
    "index": 0,
    "descendant": [
      {
        "block_type": "numbered_list",
        "numbered": {
          "elements": [{"text_run": {"content": "第一步：准备数据"}}]
        }
      },
      {
        "block_type": "numbered_list",
        "numbered": {
          "elements": [{"text_run": {"content": "第二步：训练模型"}}]
        }
      },
      {
        "block_type": "numbered_list",
        "numbered": {
          "elements": [{"text_run": {"content": "第三步：评估结果"}}]
        }
      }
    ]
  }'
```

##### 示例 8：创建表格 (嵌套块典型用法)
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/page/entries/{entry_id}/blocks/descendant" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "parent_block_id": "页面根块ID",
    "index": 0,
    "children": ["table-1"],
    "descendant": [
      {
        "block_id": "table-1",
        "block_type": "table",
        "table": {
          "row_size": 3,
          "column_size": 3,
          "column_width": [200, 200, 200],
          "header_row": true,
          "header_column": false
        },
        "children": [
          "cell-1-1", "cell-1-2", "cell-1-3",
          "cell-2-1", "cell-2-2", "cell-2-3",
          "cell-3-1", "cell-3-2", "cell-3-3"
        ]
      },
      {
        "block_id": "cell-1-1",
        "block_type": "table_cell",
        "table_cell": {"align": "center", "vertical_align": "middle"},
        "children": ["text-1-1"]
      },
      {
        "block_id": "text-1-1",
        "block_type": "p",
        "text": {
          "elements": [{"text_run": {"content": "序号", "text_style": {"bold": true}}}]
        }
      },
      {
        "block_id": "cell-1-2",
        "block_type": "table_cell",
        "table_cell": {"align": "center", "vertical_align": "middle"},
        "children": ["text-1-2"]
      },
      {
        "block_id": "text-1-2",
        "block_type": "p",
        "text": {
          "elements": [{"text_run": {"content": "名称", "text_style": {"bold": true}}}]
        }
      },
      {
        "block_id": "cell-1-3",
        "block_type": "table_cell",
        "table_cell": {"align": "center", "vertical_align": "middle"},
        "children": ["text-1-3"]
      },
      {
        "block_id": "text-1-3",
        "block_type": "p",
        "text": {
          "elements": [{"text_run": {"content": "状态", "text_style": {"bold": true}}}]
        }
      },
      {
        "block_id": "cell-2-1",
        "block_type": "table_cell",
        "table_cell": {"align": "center"},
        "children": ["text-2-1"]
      },
      {
        "block_id": "text-2-1",
        "block_type": "p",
        "text": {
          "elements": [{"text_run": {"content": "1"}}]
        }
      },
      {
        "block_id": "cell-2-2",
        "block_type": "table_cell",
        "children": ["text-2-2"]
      },
      {
        "block_id": "text-2-2",
        "block_type": "p",
        "text": {
          "elements": [{"text_run": {"content": "项目 A"}}]
        }
      },
      {
        "block_id": "cell-2-3",
        "block_type": "table_cell",
        "children": ["text-2-3"]
      },
      {
        "block_id": "text-2-3",
        "block_type": "p",
        "text": {
          "elements": [{"text_run": {"content": "✅ 完成", "text_style": {"color": "#4CAF50"}}}]
        }
      },
      {
        "block_id": "cell-3-1",
        "block_type": "table_cell",
        "table_cell": {"align": "center"},
        "children": ["text-3-1"]
      },
      {
        "block_id": "text-3-1",
        "block_type": "p",
        "text": {
          "elements": [{"text_run": {"content": "2"}}]
        }
      },
      {
        "block_id": "cell-3-2",
        "block_type": "table_cell",
        "children": ["text-3-2"]
      },
      {
        "block_id": "text-3-2",
        "block_type": "p",
        "text": {
          "elements": [{"text_run": {"content": "项目 B"}}]
        }
      },
      {
        "block_id": "cell-3-3",
        "block_type": "table_cell",
        "children": ["text-3-3"]
      },
      {
        "block_id": "text-3-3",
        "block_type": "p",
        "text": {
          "elements": [{"text_run": {"content": "⏳ 进行中", "text_style": {"color": "#FF9800"}}}]
        }
      }
    ]
  }'
```

**table 参数说明**:
| 参数 | 类型 | 说明 |
|------|------|------|
| row_size | Int | 行数 |
| column_size | Int | 列数 |
| column_width | Array | 各列宽度（像素） |
| header_row | Boolean | 是否有表头行 |
| header_column | Boolean | 是否有表头列 |

**table_cell 参数说明**:
| 参数 | 类型 | 说明 |
|------|------|------|
| align | String | 水平对齐：left/center/right |
| vertical_align | String | 垂直对齐：top/middle/bottom |

##### 示例 9：创建多个块（批量创建）
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/page/entries/{entry_id}/blocks/descendant" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "parent_block_id": "页面根块ID",
    "index": 0,
    "descendant": [
      {
        "block_type": "h1",
        "heading1": {
          "elements": [{"text_run": {"content": "文档标题"}}]
        }
      },
      {
        "block_type": "p",
        "text": {
          "elements": [{"text_run": {"content": "这是文档的第一段内容。"}}]
        }
      },
      {
        "block_type": "divider"
      },
      {
        "block_type": "h2",
        "heading2": {
          "elements": [{"text_run": {"content": "第一章节"}}]
        }
      },
      {
        "block_type": "p",
        "text": {
          "elements": [{"text_run": {"content": "章节内容..."}}]
        }
      }
    ]
  }'
```

##### 响应格式

**简单块响应**:
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "descendant": [
      {
        "block_id": "真实块ID",
        "block_type": "p",
        "parent_id": "父块ID",
        "children": [],
        "text": {...}
      }
    ],
    "block_id_relation": {}
  }
}
```

**嵌套块响应** (包含临时ID到真实ID的映射):
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "descendant": [...],
    "block_id_relation": {
      "table-1": "真实块ID-001",
      "cell-1-1": "真实块ID-002",
      "text-1-1": "真实块ID-003"
    }
  }
}
```

> **频率限制**: 3000 次/分钟
> **权限要求**: 知识库管理权限

---

#### 更新块内容
```bash
curl -X PUT "https://lxapi.lexiangla.com/cgi-bin/v1/kb/page/entries/{entry_id}/blocks/{block_id}" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "action": "update_text_elements",
    "text_elements": [{"type": "text", "text": "更新的内容"}]
  }'
```

#### 删除块
```bash
curl -X DELETE "https://lxapi.lexiangla.com/cgi-bin/v1/kb/page/entries/{entry_id}/blocks/{block_id}" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID"
```

#### 获取块详情
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/page/entries/{entry_id}/blocks/{block_id}" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

#### 获取子块列表
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/page/entries/{entry_id}/blocks/children?parent_block_id={block_id}&with_descendants=0" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

### 3.4 线上文档附件接口

#### 获取附件详情
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/files/{file_id}" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```
> 响应包含 `links.download` 附件下载链接

### 3.5 知识反馈接口

#### 获取反馈列表
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/spaces/{space_id}/feedbacks?limit=20" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```
**反馈状态 status：**
- unprocessed: 未处理
- processing: 处理中
- processed: 已处理
- not_process: 无需处理

**反馈类型 type：**
- kb_content_incomplete: 内容缺失
- kb_content_mistake: 内容有误
- kb_content_suggestion: 内容建议

---

## 四、任务管理

#### 查询成员任务列表
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/tasks?staff_id=zhangsan&type=reading&status=0&limit=20" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```
| 参数 | 必须 | 说明 |
|------|------|------|
| staff_id | 是 | 被查询者员工ID |
| type | 否 | reading(阅读任务), collaboration(协作任务) |
| status | 否 | 0:未完成, 2:已完成 |
| limit | 否 | 每页数量，默认20，最大100 |
| page_token | 否 | 分页令牌 |

#### 查询任务详情
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/tasks/{task_id}" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

#### 查询任务成员列表
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/tasks/{task_id}/members?limit=20" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

---

## 五、自定义属性管理

#### 获取自定义属性列表
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/properties?source_type=company&limit=20" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```
| 参数 | 说明 |
|------|------|
| source_type | company(公司属性), kb_space(知识库属性) |
| source_id | 当 source_type=kb_space 时必填 |

#### 获取自定义属性详情
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/properties/{property_id}" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

#### 编辑自定义属性
```bash
curl -X PUT "https://lxapi.lexiangla.com/cgi-bin/v1/kb/properties/{property_id}/schema" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "name": "属性名称",
    "description": "属性描述",
    "multiple": false,
    "options": [
      {"key": "opt1", "value": "选项1"},
      {"key": "opt2", "value": "选项2"}
    ]
  }'
```

#### 获取知识自定义属性
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/entries/{entry_id}/properties/values" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

#### 修改知识自定义属性
```bash
curl -X PUT "https://lxapi.lexiangla.com/cgi-bin/v1/kb/entries/{entry_id}/properties/values" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "properties": [
      {"id": "property_id", "value": ["opt1"]}
    ]
  }'
```

---

## 六、操作日志

#### 获取知识操作日志列表
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/operation-logs?log_type=kb&started_at=2025-01-01&ended_at=2025-12-31" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```
> 返回知识库内的文档操作（新建、添加权限等），包含操作人、IP、地理位置及具体操作内容

#### 获取管理操作日志列表
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/operation-logs?log_type=manager&started_at=2025-01-01&ended_at=2025-12-31" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```
> 返回管理层面的操作（平台管理员添加、通讯录管理、团队管理、知识库管理等）

---

## 七、AI 助手

### 7.1 FAQ 管理

#### 获取 FAQ 列表
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/ai-faqs?limit=20&page=1" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```
> 返回问题、答案、分类、相似问法等信息

### 7.2 AI 搜索

#### AI 搜索
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/ai/search" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"query": "搜索关键词"}'
```
> **注意**：此接口需要 `x-staff-id` 头部

### 7.3 AI 问答

#### AI 问答
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/ai/qa" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "query": "问题内容",
    "research": false
  }'
```
> **注意**：此接口需要 `x-staff-id` 头部
> `research=true` 使用专业研究模式

### 7.4 知识解析

#### 获取知识解析结果
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/entries/{entry_id}/parsed-content" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

#### 获取知识解析切片
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/entries/{entry_id}/chunked-content" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```
> 用于 Embedding 和检索。返回切片文件下载地址（有效期60分钟，格式为 jsonl）

#### 获取附件解析结果
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/files/{file_id}/parsed-content" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

#### 获取附件解析切片
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/kb/files/{file_id}/chunked-content" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```

---

## 八、素材管理

> **文档参考**: https://lexiang.tencent.com/wiki/api/12004.html

#### 本地文件上传（三步流程）

**步骤一：获取上传凭证**
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/files/upload-params" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"name": "文件名.md", "media_type": "file"}'
```

| 参数 | 必须 | 说明 |
|------|------|------|
| name | 是 | 文件名（需带扩展名，如 "测试.xlsx"、"文档.md"） |
| media_type | 是 | 媒体类型：`file`(文件), `video`(视频), `audio`(音频) |

**响应示例：**
```json
{
  "options": {
    "Bucket": "lexiang-10029162",
    "Region": "ap-shanghai"
  },
  "object": {
    "key": "company_xxx/kb/entries/2026/01/xxx.md",
    "state": "xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx",
    "auth": {
      "Authorization": "q-sign-algorithm=sha1&q-ak=...",
      "XCosSecurityToken": "..."
    }
  }
}
```

**步骤二：上传文件到腾讯云 COS**
```bash
curl -X PUT "https://{Bucket}.cos.{Region}.myqcloud.com/{key}" \
  -H "Authorization: {object.auth.Authorization}" \
  -H "x-cos-security-token: {object.auth.XCosSecurityToken}" \
  -H "Content-Type: application/octet-stream" \
  --data-binary @/path/to/file.md
```
> 上传成功返回 HTTP 200，响应头包含 `ETag`

**步骤三：使用 state 创建知识节点**
```bash
curl -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/entries?state={state}&space_id={space_id}" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "data": {
      "type": "kb_entry",
      "attributes": {
        "entry_type": "file",
        "name": "文件名.md"
      },
      "relationships": {
        "parent_entry": {
          "data": {"type": "kb_entry", "id": "parent_entry_id"}
        }
      }
    }
  }'
```

> **entry_type 支持的类型：**
> - `file`: 图片、文档（doc/docx/xls/xlsx/ppt/pptx/pdf/txt/**md** 等）
> - `video`: 视频（mp4/mov/avi/wmv 等）
> - `audio`: 音频（mp3/m4a/wav 等）

---

#### 完整示例：上传 Markdown 文件

```bash
#!/bin/bash

LEXIANG_TOKEN=$(cat ~/.config/lexiang/token)
LEXIANG_STAFF_ID="your_staff_id"
FILE_PATH="/path/to/document.md"
FILE_NAME="文档名称.md"
SPACE_ID="your_space_id"
PARENT_ID="parent_entry_id"  # 可选，不传则放到知识库根目录

# 步骤1: 获取上传凭证
UPLOAD_PARAMS=$(curl -s -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/files/upload-params" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d "{\"name\": \"$FILE_NAME\", \"media_type\": \"file\"}")

BUCKET=$(echo "$UPLOAD_PARAMS" | jq -r '.options.Bucket')
REGION=$(echo "$UPLOAD_PARAMS" | jq -r '.options.Region')
KEY=$(echo "$UPLOAD_PARAMS" | jq -r '.object.key')
STATE=$(echo "$UPLOAD_PARAMS" | jq -r '.object.state')
AUTH=$(echo "$UPLOAD_PARAMS" | jq -r '.object.auth.Authorization')
TOKEN=$(echo "$UPLOAD_PARAMS" | jq -r '.object.auth.XCosSecurityToken')

# 步骤2: 上传到腾讯云COS
curl -s -X PUT "https://${BUCKET}.cos.${REGION}.myqcloud.com/${KEY}" \
  -H "Authorization: $AUTH" \
  -H "x-cos-security-token: $TOKEN" \
  -H "Content-Type: application/octet-stream" \
  --data-binary "@$FILE_PATH"

# 步骤3: 创建知识节点
curl -s -X POST "https://lxapi.lexiangla.com/cgi-bin/v1/kb/entries?state=$STATE&space_id=$SPACE_ID" \
  -H "Authorization: Bearer $LEXIANG_TOKEN" \
  -H "x-staff-id: $LEXIANG_STAFF_ID" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d "{
    \"data\": {
      \"type\": \"kb_entry\",
      \"attributes\": {
        \"entry_type\": \"file\",
        \"name\": \"$FILE_NAME\"
      },
      \"relationships\": {
        \"parent_entry\": {
          \"data\": {\"type\": \"kb_entry\", \"id\": \"$PARENT_ID\"}
        }
      }
    }
  }"
```

> **💡 提示**：乐享支持 Markdown 文件预览，上传后可直接在线查看渲染效果

---

## 九、导出任务管理

#### 查询任务
```bash
curl "https://lxapi.lexiangla.com/cgi-bin/v1/jobs/{job_id}" \
  -H "Authorization: Bearer $LEXIANG_TOKEN"
```
> - 只能查询72小时内创建的任务
> - `download_url` 有效期约 5 小时

**响应示例：**
```json
{
  "code": 0,
  "msg": "ok",
  "status": 0,
  "data": {
    "download_url": "https://xxxxx"
  }
}
```
> status: 0(等待中), 1(进行中), 2(已完成)

---

## 十、单点登录（SSO）

### 流程概述
乐享知识库支持 OAuth 授权码模式的单点登录，分为 SP 发起和 IdP 发起两种流程。

### SP 发起 SSO 流程
1. 用户访问乐享链接
2. 乐享重定向到企业授权页面
3. 企业验证后跳转回乐享

### IdP 发起 SSO 流程
用户从企业平台携带凭证跳转至乐享

### 可信 IP
乐享服务器的请求来源 IP（用于判断请求是否合法）：
- `111.230.70.44`
- `111.230.156.88`

---

## 错误码说明

| HTTP状态码 | 说明 |
|-----------|------|
| 200 OK | 请求成功 |
| 201 Created | 创建成功 |
| 204 No Content | 删除成功 |
| 400 Bad Request | 请求参数错误 |
| 401 Unauthorized | Token无效或过期 |
| 403 Forbidden | 无权限访问 |
| 404 Not Found | 资源不存在 |
| 429 Too Many Requests | 超出频率限制 |

### 通讯录常见错误码
| 错误码 | 说明 |
|--------|------|
| 1001 | 用户不存在 |
| 1002 | 部门不存在 |
| 1003 | 参数错误 |
| 1004 | 部门下存在子部门 |
| 1005 | 部门下存在用户 / 部门名称已存在 |
| 1006 | 该部门是根部门 |
| 1007 | 部门层级超过15层 |
| 1010 | 父部门不能为该部门的子部门 |

---

## 权限角色说明

| 角色 | 说明 |
|------|------|
| manager | 管理权限 |
| editor | 编辑权限 |
| downloader | 查看/下载权限 |
| viewer | 仅查看权限 |

---

## 十一、调试经验与最佳实践

### 📝 写入文档内容的两种方式对比

| 方式 | 优点 | 缺点 | 推荐场景 |
|------|------|------|---------|
| **块接口 (page + blocks)** | 精确控制格式、可实时编辑 | 结构复杂，列表/表格等需要嵌套块 | 需要程序化编辑文档内容 |
| **上传 Markdown 文件** | 简单高效、格式完整保留 | 需要三步流程，更新需重新上传 | **推荐！** 批量创建文档、Markdown 内容发布 |

### 🎯 推荐：上传 Markdown 文件方式

对于需要发布 Markdown 格式文档的场景（如技术文档、论文摘要等），**强烈推荐使用文件上传方式**：

1. **Markdown 格式完整保留**：标题、列表、表格、代码块等均可正确渲染
2. **流程简单**：只需三步（获取凭证 → 上传 COS → 创建节点）
3. **支持在线预览**：乐享原生支持 `.md` 文件的在线渲染

### ⚠️ 使用块接口的注意事项

如果必须使用块接口（如需要实时编辑文档内容），请注意以下要点：

#### 1. 新建文档无需获取 parent_block_id

```bash
# ✅ 正确：新建空白文档后，直接插入内容（不传 parent_block_id）
curl -X POST ".../blocks/descendant" \
  -d '{"descendant": [...]}'

# ❌ 错误：新建文档后还要先获取 root_block_id
# 这是多此一举，直接不传 parent_block_id 即可
```

#### 2. 列表块的字段名与类型名不同

```bash
# 无序列表：block_type 是 bulleted_list，但内容字段是 bulleted
{
  "block_type": "bulleted_list",
  "bulleted": {
    "elements": [{"text_run": {"content": "列表项内容"}}]
  }
}

# 有序列表：block_type 是 numbered_list，但内容字段是 numbered
{
  "block_type": "numbered_list",
  "numbered": {
    "elements": [{"text_run": {"content": "列表项内容"}}]
  }
}
```

#### 3. 嵌套块必须使用 children 和 block_id

表格、引用块、高亮块等嵌套结构必须：
- 父块声明 `children` 数组（临时 ID 列表）
- 子块声明 `block_id`（与父块 children 中的 ID 对应）

```bash
{
  "children": ["table-1"],  # 声明第一级子块
  "descendant": [
    {
      "block_id": "table-1",  # 临时 ID
      "block_type": "table",
      "children": ["cell-1", "cell-2"]  # 表格的子块
    },
    {
      "block_id": "cell-1",
      "block_type": "table_cell",
      ...
    }
  ]
}
```

#### 4. 标题块的字段名需要匹配

```bash
# h1 使用 heading1，h2 使用 heading2，以此类推
{
  "block_type": "h2",
  "heading2": {  # 注意：不是 "text"！
    "elements": [{"text_run": {"content": "标题内容"}}]
  }
}
```

#### 5. 不支持嵌套子块的类型

以下块类型**不能**包含子块：
- 标题块：`h1`, `h2`, `h3`, `h4`, `h5`
- 代码块：`code`
- 媒体块：`image`, `attachment`, `video`
- 功能块：`divider`, `mermaid`, `plantuml`

### 🔧 常见错误排查

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| 列表内容显示为空 | 使用了错误的字段名（如 `text` 而非 `bulleted`） | 无序列表用 `bulleted`，有序列表用 `numbered` |
| 嵌套块创建失败 | 缺少 `children` 或 `block_id` | 确保父子块通过临时 ID 关联 |
| 标题块内容丢失 | 使用了 `text` 而非 `heading1/2/3...` | 字段名必须与块类型对应 |
| 引用块创建失败 (code 51) | `quote` 块需要嵌套结构 | 使用 `children` + 子块方式 |
| 上传文件接口 404 | 使用了旧版接口路径 | 使用 `/v1/kb/files/upload-params` |

### 💡 最佳实践总结

1. **优先使用 Markdown 文件上传**：对于完整文档发布，这是最简单可靠的方式
2. **块接口用于动态编辑**：只在需要程序化修改文档内容时使用
3. **批量创建使用单次请求**：将多个块放在同一个 `descendant` 数组中
4. **缓存 Token**：避免频繁调用 token 接口（限制 20次/10分钟）
5. **测试新文档先不传 parent_block_id**：新建空白文档直接插入即可

---

## 参考链接

- **官方文档**：https://lexiang.tencent.com/wiki/api/
- **更新日志**：https://lexiang.tencent.com/wiki/api/10005.html
- **可信IP说明**：https://lexiang.tencent.com/wiki/api/10003.html
