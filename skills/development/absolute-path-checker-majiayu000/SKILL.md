---
name: absolute-path-checker
description: 実装のテスト、リファクタリング、ポータビリティ、ディレクトリ階層について議論する際に、絶対パスが実装コードに存在しないことを確認する
allowed-tools: [Grep, Read, Edit, Bash]
---

# 絶対パスチェッカー

## 目的

実装コード内に絶対パス（特にHOMEディレクトリを含むパス）が存在しないことを担保し、ポータビリティを確保する。

## トリガー条件

以下のトピックについて議論・作業する際に自動的に実行:

- テストの実装・修正
- リファクタリング
- ポータビリティの改善
- ディレクトリ構造・階層の変更

## 実施手順

### 1. 絶対パスの検索

実装ファイル（.ts, .js）内で以下のパターンを検索:

```bash
# HOME ディレクトリベースの絶対パス
grep -r "/Users/" --include="*.ts" --include="*.js" .
grep -r "/home/" --include="*.ts" --include="*.js" .

# ルートからの絶対パス（設定ファイル以外）
grep -r '"/[a-z]' --include="*.ts" --include="*.js" .
```

### 2. 検出結果の判断

検出されたパスが以下のどれに該当するか判断:

| 種類 | 対応 |
|------|------|
| **実装コード内のリテラル** | 相対化が必要 |
| **テスト結果・ログ出力** | 許容（ただし、テストの期待値には含めない） |
| **設定ファイルのデフォルト値** | 環境変数や相対パスに置換 |
| **ドキュメント・コメント内の例** | 許容 |

### 3. 相対パスへの変換

#### 優先順位

1. **既存の変数を使用**: プロジェクトで定義済みの変数があれば使う
   - 例: `baseDir`, `projectRoot`, `Deno.cwd()` など

2. **import.meta を使用** (Deno/ESM の場合):
   ```typescript
   const __dirname = new URL(".", import.meta.url).pathname;
   const configPath = join(__dirname, "../config.json");
   ```

3. **相対パスリテラル**:
   ```typescript
   // Before
   const path = "/Users/dev/project/data/file.txt";

   // After
   const path = "./data/file.txt";
   ```

### 4. テストの実行

修正後は必ずテストを実行して動作確認:

```bash
deno task test
```

## 注意事項

- `$HOME` や `~` を含むパスも絶対パスと同様に扱う
- 環境変数から取得したパス（`Deno.env.get("HOME")` など）は許容
- パスの結合には `join()` や `resolve()` を使用し、文字列連結は避ける
