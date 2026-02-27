---
name: yakit-hotpatch-skill
description: Use when user asks to write Yakit Web Fuzzer hotpatch code, including custom fuzztag functions, beforeRequest, afterRequest, mirrorHTTPFlow, retryHandler, customFailureChecker, or mockHTTPRequest hooks. Also use when dealing with CSRF token bypass, encrypted payload brute-force, or custom data extraction in Web Fuzzer.
---

# Yakit Web Fuzzer 热加载编写指南

## 概述

热加载是 Yakit Web Fuzzer 中的高级功能，允许用户编写 Yak 代码片段，在 Fuzzer 运行时动态执行。本指南提供编写热加载代码所需的全部 API 参考和模式。

## 适用场景

- 需要自定义 fuzztag 逻辑（加密、编码、动态生成）
- 需要在发包前/收包后修改请求或响应
- 需要从响应中提取数据（数据镜像）
- 需要实现重试逻辑、失败判断、Mock 响应
- 典型安全测试场景：CSRF token 绕过、加密参数爆破、签名计算

## 热加载函数类型速查表

| 函数名 | 触发时机 | 签名 | 返回值 |
|--------|---------|------|--------|
| **自定义函数** | `{{yak(name\|param)}}` 渲染时 | `func(param) return string/list` | 字符串或数组 |
| **beforeRequest** | 每次发包前 | `func(req)` 或 `func(https, originReq, req)` | 修改后的请求 `[]byte` |
| **afterRequest** | 每次收包后 | `func(rsp)` 或 `func(https, originReq, req, originRsp, rsp)` | 修改后的响应 `[]byte` |
| **mirrorHTTPFlow** | 响应处理完成 | `func(req, rsp)` 或 `func(req, rsp, params)` | `map[string]string` 提取数据 |
| **retryHandler** | 需要重试时 | `func(https, retryCount, req, rsp, retryFunc)` | void |
| **customFailureChecker** | 响应检查时 | `func(https, req, rsp, failFunc)` | void |
| **mockHTTPRequest** | 请求拦截时 | `func(https, url, req, mockResponseFunc)` | void |

## 一、自定义 Fuzztag 函数

### 基本格式

在 Web Fuzzer 请求中通过 `{{yak(函数名)}}` 或 `{{yak(函数名|参数)}}` 调用热加载函数。

参数中可以嵌套 fuzztag：`{{yak(handle|{{x(pass_top25)}})}}`

### 函数定义

```yak
// 基础：接收字符串参数，返回字符串
handle = func(param) {
    return param.Upper()
}

// 返回数组：每个元素生成一个独立请求
handle = func(param) {
    return ["payload1", "payload2", "payload3"]
}

// 箭头函数简写
handle = result => result.Upper()

// 使用 x 前缀字符串展开 fuzztag
handle = result => x"{{int(1-10)}}"
```

### yield 模式（流式返回）

当需要动态生成大量 payload 时，使用 yield 回调：

```yak
handle = func(param, yield) {
    for i in 10 {
        yield(string(i))
    }
}
```

### 动态热加载（dyn 标签）

`{{yak:dyn::标签组(handle)}}` 配合分组标签使用，每次标签组渲染时重新调用：

```
请求体示例：{{int::1(1-10)}}{{yak:dyn::1(handle)}}
```

### 多参数传递

`|` 分隔符会将所有参数合并为一个字符串传入：

```yak
// 调用：{{yak(handle|a|b)}}
// param 的值为 "a|b"
handle = func(param) {
    assert param == "a|b"
    return "ok"
}
```

## 二、魔术方法 - beforeRequest

在每次 HTTP 请求发送前调用，用于修改请求内容。不需要 `{{yak()}}` 标签即可自动执行。

### 传统签名（1 参数）

```yak
beforeRequest = func(req) {
    // req: []byte 类型，当前请求的原始数据
    // 返回修改后的请求
    return req
}
```

### 完整签名（3 参数，推荐）

```yak
beforeRequest = func(https, originReq, req) {
    // https: bool，是否为 HTTPS 请求
    // originReq: []byte，用户原始模板请求（未经 fuzztag 渲染）
    // req: []byte，当前已渲染的请求
    return req
}
```

### 实战：CSRF Token 绕过

```yak
beforeRequest = func(req) {
    // 1. 发送 GET 请求获取包含 token 的页面
    rsp, _, err = poc.HTTP(`GET /login HTTP/1.1
Host: target.com
`)
    if err != nil { return req }

    // 2. 从响应中提取 Set-Cookie
    cookie = poc.GetHTTPPacketHeader(rsp, "Set-Cookie")

    // 3. 用 xpath 提取 CSRF token
    node, err = xpath.LoadHTMLDocument(rsp)
    if err != nil { return req }
    tokenNode = xpath.FindOne(node, "//input[@name='token']")
    if tokenNode == nil { return req }
    token = xpath.SelectAttr(tokenNode, "value")

    // 4. 替换请求中的占位符并设置 Cookie
    req = req.ReplaceAll("__TOKEN__", token)
    req = poc.AppendHTTPPacketHeader(req, "Cookie", cookie)
    return req
}
```

### 实战：添加时间戳

```yak
beforeRequest = func(req) {
    now = str.TrimRight(sprint(time.Now().Unix()), "\n")
    return req.ReplaceAll("__TIMESTAMP__", now)
}
```

### 实战：请求签名

```yak
beforeRequest = func(req) {
    body = poc.GetHTTPPacketBody(req)
    sign = codec.Md5(string(body) + "secret_key")
    return poc.ReplaceHTTPPacketHeader(req, "X-Sign", sign)
}
```

## 三、魔术方法 - afterRequest

在每次收到 HTTP 响应后调用，用于修改响应内容。

### 传统签名（1 参数）

```yak
afterRequest = func(rsp) {
    // rsp: []byte 类型，响应的原始数据
    return rsp
}
```

### 完整签名（5 参数，推荐）

```yak
afterRequest = func(https, originReq, req, originRsp, rsp) {
    // https: bool
    // originReq: []byte，用户原始模板请求
    // req: []byte，实际发送的请求
    // originRsp: []byte，原始响应
    // rsp: []byte，当前响应
    return rsp
}
```

### 实战：替换响应体

```yak
afterRequest = func(rsp) {
    return poc.ReplaceHTTPPacketBody(rsp, "modified body")
}
```

## 四、mirrorHTTPFlow - 数据提取

从请求/响应中提取键值对，结果显示在 Fuzzer 结果的提取器列中。

### 签名

```yak
// 基础签名（2 参数）
mirrorHTTPFlow = func(req, rsp) {
    return {"key": "value"}
}

// 带变量签名（3 参数）：可访问 Fuzzer 中定义的变量
mirrorHTTPFlow = func(req, rsp, params) {
    // params: map[string]string，Fuzzer 配置的参数
    return {"extracted": "data"}
}
```

### 实战：提取响应中的关键信息

```yak
mirrorHTTPFlow = func(req, rsp) {
    body = poc.GetHTTPPacketBody(rsp)
    // 提取 JSON 响应中的 token
    token = json.Find(body, "$.token")
    status = poc.GetHTTPPacketHeader(rsp, "Status")
    return {
        "token": token,
        "status": sprint(status),
        "body_length": sprint(len(body)),
    }
}
```

## 五、retryHandler - 重试处理

当需要根据响应内容决定是否重试时使用。

### 签名

```yak
// 完整签名（5 参数，推荐）
retryHandler = func(https, retryCount, req, rsp, retry) {
    // retryCount: int，当前重试次数
    // retry: func(...[]byte)，调用此函数触发重试，可传入新请求
    if rsp.Contains("rate limit") {
        sleep(1)
        retry()
    }
}

// 传统签名（4 参数）
retryHandler = func(retryCount, req, rsp, retry) {
    if rsp.Contains("no ready") { retry() }
}

// 最小签名（3 参数）
retryHandler = func(req, rsp, retry) {
    if rsp.Contains("error") { retry() }
}
```

## 六、customFailureChecker - 自定义失败判断

自定义判断响应是否为失败状态。

### 签名

```yak
// 完整签名（4 参数）
customFailureChecker = func(https, req, rsp, fail) {
    // fail: func(string)，调用此函数标记为失败，参数为失败原因
    if rsp.Contains("forbidden") {
        fail("access denied")
    }
}

// 传统签名（3 参数）
customFailureChecker = func(req, rsp, fail) {
    if !rsp.Contains("success") { fail("no success flag") }
}

// 最小签名（2 参数）
customFailureChecker = func(rsp, fail) {
    if rsp.Contains("error") { fail("error in response") }
}
```

## 七、mockHTTPRequest - Mock 响应

拦截请求并返回自定义响应，不实际发送网络请求。

### 签名

```yak
// 完整签名（4 参数）
mockHTTPRequest = func(https, url, req, mockResponse) {
    // mockResponse: func(rsp interface{})
    // rsp 可以是 string 或 []byte，内容为完整的 HTTP 响应
    mockResponse("HTTP/1.1 200 OK\r\n\r\nmocked")
}

// 传统签名（3 参数）
mockHTTPRequest = func(url, req, mockResponse) {
    mockResponse("HTTP/1.1 200 OK\r\n\r\nmocked body")
}
```

## 八、综合实战示例

### AES CBC 加密爆破

```yak
handle = func(p) {
    key = codec.DecodeHex("31323334313233343132333431323334")~
    iv = codec.DecodeHex("03395d68979ed8632646813f4c0bbdb3")~
    usernameDict = ["admin"]
    passwordDict = ["admin", "123456", "admin123", "88888888", "666666"]
    resultList = []
    for username in usernameDict {
        for password in passwordDict {
            m = {"username": username, "password": password}
            jsonInput = json.dumps(m)
            result = codec.AESCBCEncryptWithPKCS7Padding(key, jsonInput, iv)~
            base64Result = codec.EncodeBase64(result)
            resultList.Append(base64Result)
        }
    }
    return resultList
}
```

调用方式：将请求中加密参数设置为 `{{yak(handle)}}`

### 条件重试 + 数据提取

```yak
handle = result => x"{{int(1)}}"

mirrorHTTPFlow = func(req, rsp) {
    if string(rsp).Contains("success") {
        return {"result": "found"}
    }
    return {"result": "not found"}
}

retryHandler = func(https, retryCount, req, rsp, retry) {
    if rsp.Contains("no ready") { retry() }
}
```

### 完整的签名计算 + 提取模板

```yak
// 自定义 payload 生成
genPayload = func(param) {
    return ["test1", "test2", "test3"]
}

// 发包前计算签名
beforeRequest = func(https, originReq, req) {
    body = poc.GetHTTPPacketBody(req)
    ts = sprint(time.Now().Unix())
    sign = codec.Md5(string(body) + ts + "secret")
    req = poc.ReplaceHTTPPacketHeader(req, "X-Timestamp", ts)
    req = poc.ReplaceHTTPPacketHeader(req, "X-Sign", sign)
    return req
}

// 提取响应关键字段
mirrorHTTPFlow = func(req, rsp) {
    body = poc.GetHTTPPacketBody(rsp)
    return {
        "status_code": sprint(poc.GetStatusCodeFromResponse(rsp)),
        "body_length": sprint(len(body)),
    }
}
```

## 常用 Yak 标准库函数

### 编解码 (codec)

| 函数 | 说明 |
|------|------|
| `codec.EncodeBase64(data)` | Base64 编码 |
| `codec.DecodeBase64(str)~` | Base64 解码 |
| `codec.EncodeBase64Url(data)` | URL-safe Base64 |
| `codec.Md5(data)` | MD5 哈希 |
| `codec.Sha256(data)` | SHA256 哈希 |
| `codec.DecodeHex(hex)~` | 十六进制解码 |
| `codec.EncodeToHex(data)` | 十六进制编码 |
| `codec.AESCBCEncryptWithPKCS7Padding(key, data, iv)~` | AES CBC 加密 |
| `codec.AESCBCDecryptWithPKCS7Padding(key, data, iv)~` | AES CBC 解密 |
| `codec.RSAEncryptWithPKCS1v15(pemBytes, data)~` | RSA 加密 |
| `codec.Sm2EncryptC1C3C2(pubKey, data)~` | SM2 加密 |

### HTTP 操作 (poc)

| 函数 | 说明 |
|------|------|
| `poc.HTTP(rawRequest)` | 发送原始 HTTP 请求，返回 `(rsp, req, err)` |
| `poc.GetHTTPPacketBody(packet)` | 获取 HTTP 包体 |
| `poc.GetHTTPPacketHeader(packet, key)` | 获取指定 Header |
| `poc.ReplaceHTTPPacketBody(packet, body)` | 替换包体 |
| `poc.ReplaceHTTPPacketHeader(packet, key, value)` | 替换 Header |
| `poc.AppendHTTPPacketHeader(packet, key, value)` | 添加 Header |
| `poc.GetStatusCodeFromResponse(rsp)` | 获取状态码 |

### 其他常用

| 函数 | 说明 |
|------|------|
| `json.dumps(obj)` | 对象转 JSON 字符串 |
| `json.Find(data, jsonpath)` | JSONPath 提取 |
| `xpath.LoadHTMLDocument(html)` | 加载 HTML 文档 |
| `xpath.FindOne(node, expr)` | XPath 查找单个节点 |
| `xpath.SelectAttr(node, attr)` | 获取节点属性值 |
| `re.FindAll(data, pattern)` | 正则匹配所有 |
| `str.TrimSpace(s)` | 去除首尾空白 |
| `sprint(v)` / `sprintf(fmt, v...)` | 格式化字符串 |
| `sleep(seconds)` | 等待指定秒数 |

## Yak 语法要点

- **Wavy Call (`~`)**: `codec.DecodeHex(data)~` 自动处理错误，等价于 `result, err = ...; die(err)`
- **x 前缀字符串**: `x"{{int(1-10)}}"` 在字符串中展开 fuzztag，返回数组
- **Map 字面量**: `{"key": "value"}` 直接创建字典
- **箭头函数**: `f = (a, b) => { return a + b }` 或单表达式 `f = a => a.Upper()`
- **for-range**: `for k, v in list { ... }` 或 `for item in list { ... }`
- **字符串方法**: `"abc".Upper()`、`"abc".Contains("b")`、`"abc".ReplaceAll("a", "x")`

## 常见错误

| 错误 | 原因 | 修复 |
|------|------|------|
| `function XXX not found` | 热加载中未定义该函数名 | 检查函数名拼写是否与 `{{yak(XXX)}}` 一致 |
| beforeRequest 不执行 | 函数签名错误或语法错误 | 先在 Yak Runner 中测试代码逻辑 |
| 返回值未生效 | 忘记 return 或返回类型不对 | beforeRequest/afterRequest 必须返回 `[]byte` 或 `string` |
| mirrorHTTPFlow 无输出 | 返回值不是 `map[string]string` | 确保返回字典，值必须是字符串 |
| 并发问题 | 热加载函数内部有共享状态 | 热加载引擎已内置互斥锁，单个函数是串行执行的 |
| `~` 导致中断 | wavy call 遇到错误会终止执行 | 对可能失败的调用改用 `result, err = ...` 手动处理 |
