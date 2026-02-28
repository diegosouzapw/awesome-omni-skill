---
name: claude-config-management
description: "Claude Code設定(リポジトリルート)の構成管理ガイド。ファイルレベルsymlinkによる設定管理、管理対象の追加・削除、Taskfileタスクの実行方法を提供する。「設定ファイルを追加して」「新しいスキルを追加して」「symlinkの状態を確認して」「Claude設定を変更して」のようにClaude Code設定の構成変更を行うときに使用する。"
---

# Claude Code 構成管理

## アーキテクチャ

`~/.claude` は実ディレクトリ。リポジトリの `dotclaude/` 配下の管理対象ファイルだけを個別に symlink する。
ランタイムファイル(cache, debug, history 等)はリポジトリに含まれない。

```
~/.claude/                       (実ディレクトリ)
├── CLAUDE.md               -> <repo>/dotclaude/CLAUDE-global.md
├── settings.json           -> <repo>/dotclaude/settings.json
├── env.sh                  -> <repo>/dotclaude/env.sh
├── hooks/                  -> <repo>/dotclaude/hooks/
├── skills/
│   ├── <skill-name>/       -> <repo>/dotclaude/skills/<skill-name>/  (symlink)
│   └── <plugin-skills>/       (実ディレクトリ、管理外)
├── cache/                     (ランタイム、管理外)
├── projects/                  (ランタイム、管理外)
└── ...
```

**管理方針**: `dotclaude/` 配下のファイル・ディレクトリが symlink 対象｡リポジトリルートの他のファイルは管理外｡

## タスク

```sh
task setup    # symlink セットアップ (マイグレーション含む)
task status   # 状態確認
task clean    # symlink 削除
```

タスク定義: `Taskfile.yml` (リポジトリルート)

## 管理対象の追加手順

### トップレベルファイルを追加

1. `dotclaude/` に `<filename>` を配置
2. `Taskfile.yml` の `setup` タスクの `for file in dotclaude/settings.json dotclaude/env.sh` に追加
3. `task setup` を実行

### トップレベルディレクトリを追加

1. `dotclaude/` に `<dirname>/` を配置
2. `Taskfile.yml` の `setup` タスクの `for dir in dotclaude/hooks` に追加
3. `task setup` を実行

### グローバルスキルを追加 (git管理対象)

1. `dotclaude/skills/<skill-name>/SKILL.md` を作成
2. `task setup` を実行 (`dotclaude/skills/*/` を自動検出、Taskfile変更不要)

### プロジェクトスコープのスキルを追加

`.claude/skills/<skill-name>/SKILL.md` を作成。symlink 不要。

## worktree 環境チェック

worktree 環境では symlink 先がメインリポジトリを指すため、`~/.claude/` 配下の symlink を直接編集してはいけない。
worktree 判定を行い、`$(pwd)/<file>` を使用すること。

```bash
GIT_DIR=$(git rev-parse --git-dir)
GIT_COMMON=$(git rev-parse --git-common-dir)
# GIT_DIR != GIT_COMMON → worktree 環境
# GIT_DIR == GIT_COMMON → 通常リポジトリ
```

詳細は CLAUDE.md の「worktree 環境でのファイルパス解決」を参照。

## 注意事項

- `ln -sfn` でディレクトリ先に既存ディレクトリがあるとネスト symlink が発生する。
  `~/.claude` が実ディレクトリであることを前提としている
- `task setup` はべき等 (何度実行しても安全)
