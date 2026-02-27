---
name: web-artifacts-builder
description: Suite of tools for creating elaborate, multi-component Codex CLI HTML artifacts using modern frontend web technologies (React, Tailwind CSS, shadcn/ui). Use for complex artifacts requiring state management, routing, or shadcn/ui components - not for simple single-file HTML/JSX artifacts.
license: Complete terms in LICENSE.txt
---

# Web Artifacts Builder

To build powerful frontend Codex artifacts, follow these steps:
1. Initialize the frontend repo using `scripts/init-artifact.sh`
2. Develop your artifact by editing the generated code
3. Bundle all code into a single HTML file using `scripts/bundle-artifact.sh`
4. Display artifact to user
5. (Optional) Test the artifact

**Stack**: React 18 + TypeScript + Vite + Parcel (bundling) + Tailwind CSS + shadcn/ui

## Design & Style Guidelines

VERY IMPORTANT: To avoid what is often referred to as "AI slop", avoid using excessive centered layouts, purple gradients, uniform rounded corners, and Inter font.

## Quick Start

### Step 1: Initialize Project

Run the initialization script to create a new React project:
```bash
bash scripts/init-artifact.sh <project-name>
cd <project-name>
```

This creates a fully configured project with:
- ✅ React + TypeScript (via Vite)
- ✅ Tailwind CSS 3.4.1 with shadcn/ui theming system
- ✅ Path aliases (`@/`) configured
- ✅ 40+ shadcn/ui components pre-installed
- ✅ All Radix UI dependencies included
- ✅ Parcel configured for bundling (via .parcelrc)
- ✅ Node 18+ compatibility (auto-detects and pins Vite version)

### Step 2: Develop Your Artifact

To build the artifact, edit the generated files. See **Common Development Tasks** below for guidance.

### Step 3: Bundle to Single HTML File

ReactコンポーネントをCodexで共有できる単一HTMLにまとめるには、プロジェクト直下で次を実行します（例: `./scripts/bundle-artifact.sh`）:
```bash
bash scripts/bundle-artifact.sh
```

これで `bundle.html` が生成され、JS/CSS/アセットがすべてインライン化された1ファイルになります。`bundle.html` を Codex チャットに添付すれば、そのまま実行可能なアーティファクトとして共有できます。

**Requirements**: プロジェクト直下に `index.html` があること。

**What the script does**:
- bundling依存 (parcel, @parcel/config-default, parcel-resolver-tspaths, html-inline) を導入
- `.parcelrc` を生成してパスエイリアス対応
- Parcelでビルド（source mapなし）
- `html-inline` で dist/index.html を1ファイル化 (`bundle.html` 出力)

### Step 4: Share Artifact with User

生成された `bundle.html` を Codex で提示・添付すれば、追加のビルドや依存インストールなしでそのまま開けます。

### Step 5: Testing/Visualizing the Artifact (Optional)

Note: This is a completely optional step. Only perform if necessary or requested.

To test/visualize the artifact, use available tools (including other Skills or built-in tools like Playwright or Puppeteer). In general, avoid testing the artifact upfront as it adds latency between the request and when the finished artifact can be seen. Test later, after presenting the artifact, if requested or if issues arise.

## Reference

- **shadcn/ui components**: https://ui.shadcn.com/docs/components
