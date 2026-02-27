---
name: image-publisher
description: Upload local images to image hosting services and get accessible URLs. Use this skill when the user wants to upload a local image file to an image host and get the public URL. Supports GitHub and S3-compatible storage (Qiniu, Aliyun OSS, Tencent COS, etc.).
license: MIT
---

# Image Publisher

一键将本地图片上传到图床，获取可访问的图片链接。

## 功能

- 上传本地图片到图床
- 返回可直接访问的图片 URL
- 支持 GitHub 图床
- 支持 S3 兼容存储（七牛云、阿里云 OSS、腾讯云 COS 等）
- 支持按年/月/日路径存储
- 自动检测文件名冲突，避免覆盖

## 使用前配置

### 配置方式：使用 .env 文件

在技能目录创建 `.env` 文件：

```bash
cd ~/.claude/skills/image-publisher
cp .env.example .env
```

编辑 `.env` 文件：

```bash
# GitHub 配置
GITHUB_TOKEN=your_github_personal_access_token
GITHUB_USER=your_username
GITHUB_REPO=your_repo
GITHUB_BRANCH=master
GITHUB_IMAGES_DIR=images

# 路径模式（可选）
# flat: 平铺模式，images/xxx.png
# date: 按日期，images/2026/01/12/xxx.png
GITHUB_PATH_MODE=flat
```

### S3 图床配置（七牛云等）

编辑 `.env` 文件：

```bash
# 选择 S3 图床
PROVIDER_TYPE=s3

# S3 配置
S3_ACCESS_KEY_ID=your_access_key_id
S3_SECRET_ACCESS_KEY=your_secret_access_key
S3_BUCKET=your_bucket
S3_ENDPOINT=s3.cn-south-1.qiniucs.com
S3_REGION=cn-south-1
S3_IMAGES_DIR=images
S3_PATH_MODE=flat

# 可选：自定义域名（CDN 加速）
# S3_DOMAIN=https://cdn.example.com
```

**七牛云 S3 配置示例：**

| 配置项 | 示例值 |
|--------|--------|
| `S3_ENDPOINT` | `s3.cn-south-1.qiniucs.com` |
| `S3_REGION` | `cn-south-1` |

**安装依赖：**

```bash
pip install boto3
```

### 获取 GitHub Token

1. 访问 https://github.com/settings/tokens
2. 生成新 Token，勾选 `repo` 权限

## 使用方式

```
我: 上传图片 ~/Desktop/screenshot.png
Claude: [调用技能上传图片]
```

## 支持的图片格式

png, jpg, jpeg, gif, webp, svg, bmp, ico

## 输出结果

发布成功后会返回：

```json
{
  "filename": "screenshot.png",
  "path": "images/2026/01/12/screenshot.png",
  "raw_url": "https://raw.githubusercontent.com/user/rebo/master/images/2026/01/12/screenshot.png",
  "cdn_url": "https://cdn.jsdelivr.net/gh/user/rebo@master/images/2026/01/12/screenshot.png"
}
```

当 `GITHUB_PATH_MODE=flat` 时路径为 `images/screenshot.png`，当 `GITHUB_PATH_MODE=date` 时路径为 `images/2026/01/12/screenshot.png`。

## 不做什么

- 不编辑图片（裁剪、压缩等）
- 不提供批量上传
- 不提供相册管理功能

## 资源

- **Scripts**: `scripts/upload.py` - 上传脚本
- **References**: `references/config.md` - 配置参考
