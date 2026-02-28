---
name: vercel-secure-deploy
description: 将 Google AI Studio 项目部署到 Vercel。当用户提到"部署"、"上线"、"发布网站"、"Vercel"、"保护 API 密钥"时触发。
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
---

# Vercel 安全部署助手

帮助用户将 Google AI Studio 项目安全地部署到互联网上，全程交互式引导，无需技术背景。

## 执行流程

### 第一步：欢迎并收集基本信息

首先向用户打招呼并解释将要做什么：

"你好！我来帮你把这个 AI 项目发布到网上，让任何人都能访问。我会一步步引导你完成。在开始之前，我需要了解一些信息。"

使用 AskUserQuestion 工具询问：

**问题 1：你有 Vercel 账号吗？**
- 有，已经注册过了
- 没有，需要注册一个

如果没有，引导用户：
"请先注册一个 Vercel 账号：
1. 打开 https://vercel.com
2. 点击右上角 'Sign Up'
3. 推荐使用 GitHub 账号登录（如果有的话）
4. 注册完成后告诉我"

**问题 2：你有 Gemini API 密钥吗？**
- 有，我已经有了
- 没有，需要获取一个

如果没有，引导用户：
"请获取一个免费的 Gemini API 密钥：
1. 打开 https://aistudio.google.com/apikey
2. 用 Google 账号登录
3. 点击 'Create API Key'
4. 复制生成的密钥（一长串字母数字）
5. 把密钥发给我"

**问题 3：请把你的 API 密钥发给我**
（等待用户提供密钥，格式类似：AIzaSy...）

收到密钥后确认：
"收到！我会把这个密钥安全地配置到服务器端，不会暴露在网页代码中。"

**问题 4：你想要什么域名前缀？**

使用 AskUserQuestion 工具询问：
- 使用默认（项目名称）
- 自定义域名前缀

如果选择自定义，让用户在"其他"选项中输入想要的前缀。

说明：
"你的网站地址将会是：**[前缀].vercel.app**

域名前缀规则：
- 只能包含小写字母、数字和连字符（-）
- 不能以连字符开头或结尾
- 例如：my-cool-app、memegen2024"

记录用户选择的域名前缀，后续部署时使用。

### 第二步：分析项目（重要）

自动执行，无需用户操作：

1. 读取 `package.json` 确定项目类型
2. **读取原项目的 AI 服务代码，提取模型名称**（非常重要！）
3. 查找 API 调用代码的具体实现方式
4. 检查现有配置

**关键：必须从原项目代码中提取以下信息：**
- 使用的模型名称（如 `gemini-3-pro-image-preview`）
- API 调用的参数结构（如 `imageConfig`、`inputImage` 等）
- 响应数据的处理方式

向用户报告：
"正在分析你的项目...

✅ 项目类型：[React/Vue/其他]
✅ AI 服务：[Gemini/OpenAI/其他]
✅ 使用的模型：[从代码中提取的模型名称]
✅ 需要保护的 API 密钥：已找到

接下来我会自动完成代码改造，请稍等..."

### 第三步：自动改造代码

执行以下操作（自动完成，向用户展示进度）：

1. **创建后端接口** (`api/` 目录)
   - 告诉用户："正在创建安全的后端接口..."
   - **重要：必须使用原项目中的模型名称和 API 调用方式**
   - 参考 `templates/` 目录下的模板，但要根据原项目调整

2. **修改前端代码**
   - 告诉用户："正在修改前端代码，移除暴露的密钥..."
   - 更新 services 文件，改为调用后端接口
   - 保持原有的函数签名和返回值格式

3. **创建配置文件**
   - 告诉用户："正在创建部署配置..."
   - 创建 `vercel.json`
   - 更新 `vite.config.ts`（移除密钥注入）
   - 更新 `package.json`（添加依赖和 engines 字段）

4. **创建环境变量文件**
   - 创建 `.env.local` 写入用户提供的 API 密钥
   - 确保 `.gitignore` 包含 `.env*` 或 `*.local`

完成后告诉用户：
"✅ 代码改造完成！

改动说明：
- 创建了安全的后端接口，API 密钥只存在服务器端
- 网页代码中不再包含任何密钥
- 已配置好部署设置

接下来进入部署环节..."

### 第四步：预部署检查（重要）

**此步骤确保 Vercel 构建一次成功，必须执行！**

1. **检查并配置 Node.js 版本**

   读取 `package.json`，确保包含 `engines` 字段：

   ```json
   {
     "engines": {
       "node": ">=18"
     }
   }
   ```

   注意：使用 `">=18"` 而不是 `">=18.x"`，避免版本警告。

2. **安装依赖**

   ```bash
   npm install
   ```

3. **本地构建测试**

   运行本地构建命令验证项目可以正常构建：

   ```bash
   npm run build
   ```

   如果构建失败，分析错误并修复后重试。常见问题：
   - TypeScript 类型错误：检查并修复类型定义
   - 依赖缺失：运行 `npm install` 安装
   - 环境变量问题：检查 vite.config.ts 配置

4. **检查必要文件存在**

   确认以下文件存在且配置正确：
   - `vercel.json` - Vercel 部署配置
   - `api/` 目录下的 API 路由文件
   - `package.json` 中的 build 脚本

告诉用户："正在进行预部署检查，确保一次部署成功..."

检查通过后告诉用户："✅ 预部署检查通过，项目可以正常构建。"

### 第五步：检查并安装 Vercel CLI

**自动执行**，使用 Bash 工具检查 Vercel CLI 是否已安装：

```bash
which vercel || npm install -g vercel
```

告诉用户："正在检查 Vercel CLI..."

如果未安装，自动安装。向用户报告安装结果。

### 第六步：检查 Vercel 登录状态

**自动执行**，使用 Bash 工具检查登录状态：

```bash
vercel whoami 2>&1
```

如果已登录，告诉用户："已检测到 Vercel 登录状态，继续部署..."

如果未登录，**自动执行登录流程**：

1. 告诉用户："检测到未登录 Vercel，正在为你启动登录流程..."

2. 使用 Bash 工具运行登录命令（自动发送回车以跳过交互提示）：

```bash
printf '\n' | vercel login 2>&1
```

3. 此命令会在后台运行，读取输出文件获取验证链接（通常包含 `vercel.com/oauth/device?user_code=` 格式的 URL）

4. 将链接展示给用户：
"请打开以下链接完成 Vercel 登录：

**[提取到的验证链接]**

在浏览器中完成授权后，登录会自动完成。"

5. 等待后台任务完成，检查输出是否包含 "Congratulations" 确认登录成功

6. 运行 `vercel whoami` 验证登录成功

**备选方案**：如果自动登录失败，引导用户使用 token 方式登录：
- 让用户访问 https://vercel.com/account/tokens 创建 token
- 使用 `vercel login --token <token>` 完成登录

### 第七步：首次部署并链接项目

**自动执行**，使用 Bash 工具运行非交互式部署来创建项目：

```bash
vercel --yes 2>&1
```

**注意：`--name` 参数已废弃，不要使用！** 项目名称会自动使用目录名。

告诉用户："正在创建 Vercel 项目..."

这一步会：
- 创建 Vercel 项目
- 生成 `.vercel` 目录
- 链接本地项目到 Vercel

### 第八步：配置环境变量

**自动执行**，使用 Bash 工具配置环境变量（使用用户之前提供的 API 密钥）：

```bash
echo "API_KEY_VALUE" | vercel env add GEMINI_API_KEY production 2>&1
echo "API_KEY_VALUE" | vercel env add GEMINI_API_KEY preview 2>&1
echo "API_KEY_VALUE" | vercel env add GEMINI_API_KEY development 2>&1
```

注意：将 `API_KEY_VALUE` 替换为用户在第一步提供的实际密钥。

告诉用户："正在配置 API 密钥到 Vercel 环境变量..."

### 第九步：本地预构建并部署到生产环境（推荐方式）

**重要：使用本地预构建方式部署，比远程构建更稳定！**

1. **拉取项目设置**

```bash
vercel pull --yes 2>&1
```

2. **本地构建生产版本**

```bash
vercel build --prod 2>&1
```

3. **部署预构建产物到生产环境**

```bash
vercel deploy --prebuilt --prod --yes 2>&1
```

告诉用户："正在进行正式部署..."

**为什么使用本地预构建？**
- Vercel 远程构建有时会因为 npm install 超时等问题失败
- 本地预构建可以提前发现和修复问题
- 部署预构建产物更快更稳定

部署完成后，从输出中提取生产网址（通常是 `xxx.vercel.app`）。

### 完成确认

向用户展示最终结果：

"部署完成！

**你的网站地址：[从部署输出中提取的生产网址]**

现在任何人都可以通过这个网址访问你的应用了。

✅ 已完成的工作：
- 创建了安全的后端接口
- API 密钥已安全配置到 Vercel 环境变量
- 项目已部署到生产环境

一些小贴士：
- 这个网址是永久有效的
- 如果以后要更新代码，运行以下命令重新部署：
  ```bash
  vercel build --prod && vercel deploy --prebuilt --prod --yes
  ```
- API 密钥安全地存储在 Vercel 服务器上，不会泄露

还有什么问题吗？"

## 错误处理指南

### 常见问题及自动修复

**npm install 在 Vercel 构建时失败**
- 原因：通常是 Node.js 版本不兼容或 npm 超时
- 修复：使用本地预构建方式（`vercel build --prod` + `vercel deploy --prebuilt --prod`）

**engines 字段警告**
- 使用 `"node": ">=18"` 而不是 `">=18.x"`
- 避免使用会自动升级的版本格式

**本地 npm install 失败**
- 检查是否安装了 Node.js
- 引导用户安装：https://nodejs.org

**vercel login 失败**
- 检查网络连接
- 如果无法获取验证链接，使用 token 方式登录：
  1. 让用户访问 https://vercel.com/account/tokens
  2. 点击 "Create Token" 创建新 token
  3. 运行 `vercel login --token <用户的token>`
- 如果浏览器无法打开，手动复制输出中的验证链接到浏览器

**Vercel 构建失败**
- 先在本地运行 `npm run build` 复现问题
- 检查 TypeScript 错误并修复
- 确保所有依赖都在 package.json 中声明

**部署后 API 调用报错 "model not found"**
- **这是最常见的问题！**
- 原因：API 路由中使用的模型名称与原项目不一致
- 修复：必须从原项目的服务文件中提取正确的模型名称
- 不要使用模板中的默认模型名称，每个项目可能使用不同的模型

**部署后网站报错**
- 检查环境变量是否正确配置：`vercel env ls`
- 查看 Vercel 仪表板的 Function Logs

**API 调用失败**
- 确认 API 密钥正确
- 确认 Gemini API 已启用
- 检查 `/api/gemini` 路由是否正确部署
- 确认模型名称正确

**预构建环境不匹配错误**
- 如果看到 "prebuilt output was built with target environment preview"
- 需要使用 `vercel build --prod` 重新构建，再用 `vercel deploy --prebuilt --prod`

## 技术操作参考

### 创建 API 路由的正确方式

1. **首先读取原项目的服务文件**，提取：
   - 模型名称（如 `gemini-3-pro-image-preview`）
   - API 调用参数结构
   - 响应处理逻辑

2. **根据原项目创建 API 路由**，而不是直接使用模板

3. **模板仅作参考**，`templates/` 目录下的文件：
   - `gemini-api-route.ts` - Gemini API 后端路由参考
   - `openai-api-route.ts` - OpenAI API 后端路由参考
   - `frontend-service.ts` - 前端服务调用方式参考

### 部署命令速查

```bash
# 首次部署（创建项目）
vercel --yes

# 配置环境变量
echo "your-api-key" | vercel env add GEMINI_API_KEY production

# 拉取项目设置
vercel pull --yes

# 本地预构建（推荐）
vercel build --prod

# 部署预构建产物到生产环境（推荐）
vercel deploy --prebuilt --prod --yes

# 更新部署的完整流程
vercel build --prod && vercel deploy --prebuilt --prod --yes
```

## 预部署检查清单

部署前确保以下项目全部通过：

- [ ] `package.json` 包含 `"engines": { "node": ">=18" }`
- [ ] `npm run build` 本地构建成功
- [ ] `vercel.json` 配置正确
- [ ] `api/` 目录存在且包含 API 路由
- [ ] API 路由中的模型名称与原项目一致
- [ ] `.gitignore` 包含 `.env*` 或 `*.local`
- [ ] 已确认用户的域名前缀选择（默认或自定义）

## 更换 API 密钥

如果用户需要更换 API 密钥：

```bash
# 删除旧密钥
vercel env rm GEMINI_API_KEY production --yes
vercel env rm GEMINI_API_KEY preview --yes
vercel env rm GEMINI_API_KEY development --yes

# 添加新密钥
echo "new-api-key" | vercel env add GEMINI_API_KEY production
echo "new-api-key" | vercel env add GEMINI_API_KEY preview
echo "new-api-key" | vercel env add GEMINI_API_KEY development

# 重新部署使新密钥生效
vercel build --prod && vercel deploy --prebuilt --prod --yes
```
