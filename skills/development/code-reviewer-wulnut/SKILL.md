---
name: code-reviewer
description: 综合代码审查 skill，支持 TypeScript、JavaScript、Python、Swift、Kotlin、Go。包括自动代码分析、最佳实践检查、安全扫描和审查清单生成。当审查 Pull Request、提供代码反馈、识别问题或确保代码质量标准时使用此 skill。
---

# Code Reviewer

完整的代码审查工具包，包含现代工具和最佳实践。

## 核心能力

### 1. PR 分析器

自动化 PR 分析工具。

**使用**：
```bash
python scripts/pr_analyzer.py <project-path> [options]
```

### 2. 代码质量检查器

综合分析和优化工具。

**使用**：
```bash
python scripts/code_quality_checker.py <target-path> [--verbose]
```

### 3. 审查报告生成器

专业任务的高级工具。

**使用**：
```bash
python scripts/review_report_generator.py [arguments] [options]
```

## 参考文档

- **代码审查清单**: `references/code_review_checklist.md`
- **编码标准**: `references/coding_standards.md`
- **常见反模式**: `references/common_antipatterns.md`

## 技术栈

**语言**: TypeScript, JavaScript, Python, Go, Swift, Kotlin
**前端**: React, Next.js, React Native, Flutter
**后端**: Node.js, Express, GraphQL, REST APIs
**数据库**: PostgreSQL, Prisma, NeonDB, Supabase
**DevOps**: Docker, Kubernetes, Terraform, GitHub Actions, CircleCI
**云**: AWS, GCP, Azure

## 最佳实践总结

### 代码质量

- 遵循既定模式
- 编写全面测试
- 记录决策
- 定期审查

### 性能

- 优化前先测量
- 使用适当的缓存
- 优化关键路径
- 在生产环境监控

### 安全

- 验证所有输入
- 使用参数化查询
- 实施正确的身份验证
- 保持依赖更新

### 可维护性

- 编写清晰代码
- 使用一致命名
- 添加有帮助的注释
- 保持简单
