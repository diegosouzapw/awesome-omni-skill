---
name: moon-proto
description: Moonrepo (Task Runner) と proto (Toolchain Manager) の統合利用ガイド。環境構築、タスク管理、ワークスペース設定の概要を提供する。詳細はサブファイルを参照。
globs:
  - "**/.moon/**"
  - "**/moon.yml"
  - "**/.prototools"
---

# Moonrepo and proto: Modern Infrastructure Guide

## 概要

このスキルは、**Moonrepo**（タスクランナー/モノレポ管理）と **proto**（ツールチェーン管理）を組み合わせて使用するためのガイドです。
これらを併用することで、開発環境の構築からCIでの実行までを高速かつ一貫性のあるものにできます。

詳細な情報は以下のファイルに分割されています。必要に応じて参照してください。

### 📚 詳細ガイド

1.  **[Toolchain Management](./guides/proto-management.md)**
    *   `proto` と `.prototools` の使い方
    *   言語・ツールのバージョン管理

2.  **[Tasks and Projects](./guides/moon-tasks.md)**
    *   `moon.yml` でのタスク定義
    *   `moon run` コマンドとキャッシュ

3.  **[Workspace Configuration](./guides/workspace-config.md)**
    *   `.moon/workspace.yml` の設定
    *   プロジェクト構成とCI/CD連携

## 参考リソース

- [Moonrepo Documentation](https://moonrepo.dev/docs)
- [proto Documentation](https://moonrepo.dev/proto)
- [moonrepo Discord](https://discord.gg/moonrepo)
