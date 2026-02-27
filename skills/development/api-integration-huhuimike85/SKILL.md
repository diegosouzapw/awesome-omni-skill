---
name: api-integration
description: API 集成和设计最佳实践。用于设计、实现和优化 RESTful API、GraphQL API 或其他 API 集成。包括错误处理、认证、限流、版本控制等。
allowed-tools: Read, Grep, Glob, Edit, Bash
---

# API 集成最佳实践

## API 设计原则

### RESTful API 设计

#### 资源命名
```
✅ 好的命名
GET    /api/v1/users
GET    /api/v1/users/{id}
POST   /api/v1/users
PUT    /api/v1/users/{id}
DELETE /api/v1/users/{id}

❌ 不好的命名
GET /api/v1/getUsers
POST /api/v1/createUser
```

#### HTTP 方法使用
- **GET** - 获取资源（幂等、安全）
- **POST** - 创建资源
- **PUT** - 完整更新资源（幂等）
- **PATCH** - 部分更新资源
- **DELETE** - 删除资源（幂等）

#### 状态码规范
```
2xx - 成功
  200 OK - 成功返回数据
  201 Created - 资源创建成功
  204 No Content - 成功但无返回内容

4xx - 客户端错误
  400 Bad Request - 请求参数错误
  401 Unauthorized - 未认证
  403 Forbidden - 无权限
  404 Not Found - 资源不存在
  422 Unprocessable Entity - 验证失败
  429 Too Many Requests - 限流

5xx - 服务器错误
  500 Internal Server Error - 服务器错误
  502 Bad Gateway - 网关错误
  503 Service Unavailable - 服务不可用
```

### API 响应格式

#### 统一响应结构
```typescript
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: any;
  };
  meta?: {
    timestamp: string;
    requestId: string;
  };
}
```

#### 成功响应
```json
{
  "success": true,
  "data": {
    "id": "123",
    "name": "John Doe"
  },
  "meta": {
    "timestamp": "2024-01-14T10:00:00Z",
    "requestId": "req_abc123"
  }
}
```

#### 错误响应
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email format",
    "details": {
      "field": "email",
      "value": "invalid-email"
    }
  },
  "meta": {
    "timestamp": "2024-01-14T10:00:00Z",
    "requestId": "req_abc123"
  }
}
```

### 分页

#### Offset-based 分页
```
GET /api/v1/users?page=1&limit=20

Response:
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "totalPages": 5
  }
}
```

#### Cursor-based 分页（推荐用于大数据集）
```
GET /api/v1/users?cursor=abc123&limit=20

Response:
{
  "data": [...],
  "pagination": {
    "nextCursor": "def456",
    "hasMore": true
  }
}
```

## 认证和授权

### JWT 认证
```typescript
// 请求头
Authorization: Bearer <jwt_token>

// Token 结构
{
  "sub": "user_id",
  "exp": 1234567890,
  "iat": 1234567890,
  "roles": ["user", "admin"]
}
```

### API Key 认证
```typescript
// 请求头
X-API-Key: <api_key>

// 或查询参数（不推荐）
GET /api/v1/users?api_key=<api_key>
```

### OAuth 2.0
```typescript
// Authorization Code Flow
1. GET /oauth/authorize?client_id=...&redirect_uri=...
2. POST /oauth/token
   {
     "grant_type": "authorization_code",
     "code": "...",
     "client_id": "...",
     "client_secret": "..."
   }
```

## 错误处理

### 错误类型定义
```typescript
enum ErrorCode {
  VALIDATION_ERROR = 'VALIDATION_ERROR',
  AUTHENTICATION_ERROR = 'AUTHENTICATION_ERROR',
  AUTHORIZATION_ERROR = 'AUTHORIZATION_ERROR',
  NOT_FOUND = 'NOT_FOUND',
  CONFLICT = 'CONFLICT',
  RATE_LIMIT_EXCEEDED = 'RATE_LIMIT_EXCEEDED',
  INTERNAL_ERROR = 'INTERNAL_ERROR'
}
```

### 错误处理中间件
```typescript
app.use((err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  const errorCode = err.code || 'INTERNAL_ERROR';

  res.status(statusCode).json({
    success: false,
    error: {
      code: errorCode,
      message: err.message,
      details: err.details
    },
    meta: {
      timestamp: new Date().toISOString(),
      requestId: req.id
    }
  });
});
```

## 限流和节流

### 限流策略
```typescript
// Token Bucket
const rateLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 分钟
  max: 100, // 最多 100 个请求
  message: 'Too many requests'
});

// 响应头
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1234567890
```

### 不同端点的限流
```typescript
// 严格限流（登录）
POST /api/v1/auth/login - 5 requests/15min

// 普通限流（读取）
GET /api/v1/users - 100 requests/15min

// 宽松限流（静态资源）
GET /api/v1/public/* - 1000 requests/15min
```

## API 版本控制

### URL 版本控制（推荐）
```
/api/v1/users
/api/v2/users
```

### Header 版本控制
```
Accept: application/vnd.myapi.v1+json
```

### 版本迁移策略
```typescript
// 支持多版本
app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);

// 版本废弃通知
res.setHeader('X-API-Deprecated', 'true');
res.setHeader('X-API-Sunset', '2024-12-31');
```

## 请求验证

### 输入验证
```typescript
import { z } from 'zod';

const createUserSchema = z.object({
  name: z.string().min(2).max(50),
  email: z.string().email(),
  age: z.number().int().min(18).max(120)
});

app.post('/api/v1/users', async (req, res) => {
  try {
    const data = createUserSchema.parse(req.body);
    // 处理请求
  } catch (error) {
    res.status(400).json({
      success: false,
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Invalid input',
        details: error.errors
      }
    });
  }
});
```

## 缓存策略

### HTTP 缓存头
```typescript
// 强缓存
Cache-Control: public, max-age=3600

// 协商缓存
ETag: "abc123"
Last-Modified: Wed, 21 Oct 2024 07:28:00 GMT

// 不缓存
Cache-Control: no-store, no-cache, must-revalidate
```

### Redis 缓存
```typescript
async function getUser(id: string) {
  // 尝试从缓存获取
  const cached = await redis.get(`user:${id}`);
  if (cached) {
    return JSON.parse(cached);
  }

  // 从数据库获取
  const user = await db.users.findById(id);

  // 写入缓存
  await redis.setex(`user:${id}`, 3600, JSON.stringify(user));

  return user;
}
```

## API 文档

### OpenAPI/Swagger
```yaml
openapi: 3.0.0
info:
  title: My API
  version: 1.0.0
paths:
  /users:
    get:
      summary: Get all users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserList'
```

## 监控和日志

### 请求日志
```typescript
app.use((req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;
    logger.info({
      method: req.method,
      path: req.path,
      statusCode: res.statusCode,
      duration,
      requestId: req.id
    });
  });

  next();
});
```

### 性能监控
```typescript
// 响应时间
X-Response-Time: 123ms

// 追踪 ID
X-Trace-Id: abc123
X-Span-Id: def456
```

## 安全最佳实践

### CORS 配置
```typescript
app.use(cors({
  origin: ['https://example.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

### 安全头
```typescript
app.use(helmet({
  contentSecurityPolicy: true,
  xssFilter: true,
  noSniff: true,
  hsts: true
}));
```

### 输入清理
```typescript
// 防止 SQL 注入
const query = 'SELECT * FROM users WHERE id = ?';
db.query(query, [userId]);

// 防止 XSS
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(userInput);
```

## 测试

### API 测试
```typescript
describe('GET /api/v1/users', () => {
  it('should return users list', async () => {
    const response = await request(app)
      .get('/api/v1/users')
      .expect(200);

    expect(response.body.success).toBe(true);
    expect(response.body.data).toBeInstanceOf(Array);
  });

  it('should handle pagination', async () => {
    const response = await request(app)
      .get('/api/v1/users?page=1&limit=10')
      .expect(200);

    expect(response.body.pagination.page).toBe(1);
    expect(response.body.pagination.limit).toBe(10);
  });
});
```

## 检查清单

设计或审查 API 时检查：

- [ ] RESTful 命名规范
- [ ] HTTP 方法使用正确
- [ ] 状态码使用恰当
- [ ] 统一的响应格式
- [ ] 完善的错误处理
- [ ] 认证和授权机制
- [ ] 限流保护
- [ ] 输入验证
- [ ] API 版本控制
- [ ] 缓存策略
- [ ] 完整的 API 文档
- [ ] 日志和监控
- [ ] 安全防护
- [ ] 完整的测试覆盖
