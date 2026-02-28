---
name: openclaw-codex-oauth-proxy
description: OpenClaw 用 openai-codex（ChatGPT OAuth）跑 GPT-5.x Codex/Spark 的配置与排障（网关进程、代理、模型、thinking、JSON 输出）。
---

# openclaw-codex-oauth-proxy

## 0) 先确认你走的是 `openai-codex`（不是 API Key 的 `openai`）

```bash
openclaw models status --plain
openclaw models status --probe --probe-provider openai-codex --json
```

看 `status=ok`，并且 provider 是 `openai-codex`。

## 1) “能登录 OAuth，但模型不可用”的常见根因：`api.openai.com` 网络不通

ChatGPT OAuth 的登录主要走 `auth.openai.com`，但模型调用通常仍需要连 `api.openai.com`。

用 **同一台机器**验证（期望返回 `401`，不是超时）：

```bash
curl -m 8 -I https://api.openai.com/v1/models
```

如果直连超时，但加代理能通：

```bash
curl -m 8 --proxy http://127.0.0.1:7897 -I https://api.openai.com/v1/models
```

那就要让 **Gateway 常驻进程**也走代理（见下一节）。

## 2) 关键点：Gateway 是常驻服务，不会继承你终端里的代理环境变量

你在终端里能 `curl --proxy ...` 不代表 `openclaw-gateway` 进程也能出网。

把代理写进 `~/.openclaw/openclaw.json` 的 `env.vars`，然后重启网关：

```bash
openclaw config set env.vars.HTTPS_PROXY "http://127.0.0.1:7897"
openclaw config set env.vars.HTTP_PROXY "http://127.0.0.1:7897"
openclaw config set env.vars.ALL_PROXY "http://127.0.0.1:7897"
openclaw config set env.vars.NO_PROXY "127.0.0.1,localhost"

openclaw gateway restart
openclaw gateway status
```

如果没装服务：

```bash
openclaw gateway install
openclaw gateway restart
```

端口冲突/旧进程残留时，用：

```bash
openclaw gateway --force
```

## 3) 设置默认模型 + thinking（例如默认 `medium`）

```bash
openclaw models set openai-codex/gpt-5.3-codex
openclaw config set agents.defaults.thinkingDefault medium
```

切 Spark（如果 models list 里能看到）：

```bash
openclaw models set openai-codex/gpt-5.3-codex-spark
```

验证实际生效的模型（看输出里的 `agentMeta.model`）：

```bash
openclaw agent --agent main -m "ping" --thinking minimal --json
```

## 4) CLI `--json` 输出前面夹杂插件日志，别直接管道给 `jq`

有些命令会先打印类似 `[plugins] ...`，导致：

- `jq` 解析失败
- 甚至触发 Node CLI `EPIPE`（broken pipe）

稳妥写法：先落盘，再从第一个 `{` 截 JSON。

```bash
openclaw sessions --json > /tmp/sessions.json
python3 - <<'PY'
import json
raw=open("/tmp/sessions.json","r",encoding="utf-8",errors="replace").read()
raw=raw[raw.find("{"):]
print(json.loads(raw).keys())
PY
```

