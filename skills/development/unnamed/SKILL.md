---
name: 代码审查
description: 按照 Sentry 工程实践进行代码审查。用于审查拉取请求、检查代码变更或提供代码质量反馈。涵盖安全性、性能、测试和设计审查。
---

# Sentry 代码审查

审查 Sentry 项目代码时请遵循以下指南。

## 审查清单

### 识别问题

在代码变更中寻找以下问题：

- **运行时错误**：潜在的异常、空指针问题、越界访问
- **性能**：无界的 O(n²) 操作、N+1 查询、不必要的内存分配
- **副作用**：影响其他组件的意外行为变更
- **向后兼容性**：没有迁移路径的破坏性 API 变更
- **ORM 查询**：复杂的 Django ORM 可能导致意外的查询性能
- **安全漏洞**：注入、XSS、访问控制缺陷、密钥泄露

### 设计评估

- 组件交互是否逻辑合理？
- 变更是否与现有项目架构一致？
- 是否与当前需求或目标存在冲突？

### 测试覆盖

每个 PR 都应有适当的测试覆盖：

- 业务逻辑的功能测试
- 组件交互的集成测试
- 关键用户路径的端到端测试

验证测试是否覆盖了实际需求和边界情况。避免在测试代码中使用过多的分支或循环。

### 长期影响

当变更涉及以下内容时，标记为需要高级工程师审查：

- 数据库 Schema 修改
- API 契约变更
- 新框架或库的引入
- 性能关键代码路径
- 安全敏感功能

## 反馈指南

### 语气

- 保持礼貌和同理心
- 提供可操作的建议，而非模糊的批评
- 不确定时用疑问句表达："你是否考虑过...？"

### 批准

- 仅剩轻微问题时即可批准
- 不要因为代码风格偏好而阻止 PR
- 记住：目标是降低风险，而非追求完美代码

## 需要标记的常见模式

### Python/Django

```python
# 错误示例：N+1 查询
for user in users:
    print(user.profile.name)  # 每个用户单独查询

# 正确示例：预取关联数据
users = User.objects.prefetch_related('profile')
```

### TypeScript/React

```typescript
// 错误示例：useEffect 中缺少依赖
useEffect(() => {
  fetchData(userId);
}, []);  // userId 不在依赖中

// 正确示例：包含所有依赖
useEffect(() => {
  fetchData(userId);
}, [userId]);
```

### 安全

```python
# 错误示例：SQL 注入风险
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")

# 正确示例：参数化查询
cursor.execute("SELECT * FROM users WHERE id = %s", [user_id])
```

## 参考资料

- [Sentry 代码审查指南](https://develop.sentry.dev/engineering-practices/code-review/)
