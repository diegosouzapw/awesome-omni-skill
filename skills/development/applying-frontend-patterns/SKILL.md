---
name: applying-frontend-patterns
description: Framework-agnostic frontend component design patterns.
allowed-tools: [Read, Grep, Glob, Task]
user-invocable: false
---

# フロントエンドパターン

## コアパターン

| パターン                 | 使用タイミング               |
| ------------------------ | ---------------------------- |
| Container/Presentational | データ取得 + 表示            |
| カスタムフック           | 共有する振る舞い             |
| コンポジション           | 柔軟なコンポーネント         |
| 状態管理                 | ローカル → 共有 → グローバル |

## Container/Presentational

| Container（ロジック） | Presentational（UI）       |
| --------------------- | -------------------------- |
| データを取得          | propsでデータを受け取る    |
| 状態を管理            | ステートレス（理想）       |
| イベントを処理        | コールバックpropsを呼ぶ    |
| スタイリングなし      | すべてのスタイリングがここ |

## 状態管理

| スコープ   | ツール        | 例                     |
| ---------- | ------------- | ---------------------- |
| ローカル   | useState      | フォーム入力、トグル   |
| 共有       | Context       | テーマ、認証状態       |
| グローバル | Zustand/Redux | アプリ全体のキャッシュ |

## 使わないとき

シンプルな一回限りのコンポーネント、プロトタイプ（YAGNI）、再利用が期待されない。

## 参考

| トピック                 | ファイル                                 |
| ------------------------ | ---------------------------------------- |
| Container/Presentational | `references/container-presentational.md` |
