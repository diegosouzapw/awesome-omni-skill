---
name: security-auditor-skill
description: 扫描指定目录或文件中的安全漏洞（特别是针对 macOS/Linux 的恶意安装脚本、Base64 混淆、反向 Shell）。当用户要求“检查安全”、“扫描漏洞”、“审计代码”或怀疑某个 Skill/Repo 有毒时触发。
---

# Security Auditor Skill

## 核心能力
本 Skill 用于对不明来源的代码仓库（特别是 Agent Skills）进行静态安全审计。
它专注于发现“投毒”痕迹，如：
- `curl | bash` 模式的隐蔽执行
- Base64 编码的恶意载荷
- 针对 macOS (Quarantine removal) 或 Linux 的系统篡改命令
- 伪装成文本的可执行二进制文件

## 使用方法 (Usage)
1.  **扫描特定目录**: "扫描这个目录: `/path/to/skill`"
2.  **默认扫描**: "扫描当前所有 Skills" (默认路径需在 Prompt 中确认, 通常为 `~/.openclaw/skills`)
3.  **处理结果**: 扫描完成后，Skill 会输出一份报告。用户可以告知 "将 xxxx 加入白名单" 以消除误报。

## 路由逻辑 (Routing)

### 1. 执行扫描
当用户请求扫描时，执行 `scripts/security_scanner.py`。

**Args**:
- `target_path`: 用户指定的绝对路径。若未指定，询问用户或使用当前工作区。
- `whitelist_path`: 默认为 `data/whitelist.json`。

**Script Command**:
```bash
python scripts/security_scanner.py --target "TARGET_PATH" --whitelist "data/whitelist.json"
```

### 2. 解读报告 (Interpretation)
脚本将输出 JSON 格式的结果。你必须将其转换为易读的 Markdown 表格：
- 🚨 **CRITICAL**: 必须高亮警告，建议用户**绝对不要执行**该文件。
- ⚠️ **HIGH/WARNING**: 提示风险，需用户人工复核代码。
- ✅ **SAFE**: 未发现已知特征。

### 3. 白名单管理
如果用户确认某个 `CRITICAL/HIGH` 实际上是误报（例如正常的模型下载），
**不要** 修改 Python 脚本。
而是**指导用户**（或由 Agent 代劳）将该文件的 Hash 或特征名称更新到 `data/whitelist.json` 中。
