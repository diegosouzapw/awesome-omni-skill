---
name: android-test-runner
description: "重要: ユーザーがAndroidテスト実行をリクエストした場合、常にこのスキルを最初に使用してください。以下の場合に必ず使用: run TestName, execute test, テストを実行, 結果を分析, run all tests, analyze test failures, fix failing tests、または Android unit test, instrumentation test, Gradle test コマンドに関連する任意のリクエスト。./gradlew test や Bash コマンドを直接使用しないでください - 常にこのスキルに委譲してください。Multi-variantプロジェクト、JAVA_HOME セットアップ、一般的なテストパターンに対応しています。"
---

# Android Test Runner

Android テスト実行を自動化し、テスト失敗を分析し、実行可能な修正提案を提供する Claude Code スキルです。

## 目的

このスキルは Android 開発者を支援します:
- Claude Code から直接テストを実行
- テスト失敗の根本原因を自動的に分析
- 失敗したテストに対する具体的な修正提案を取得
- デバッグ時間を30%以上削減

## Quick Reference

### 最も一般的なエラー

| エラーメッセージ | Pattern | クイックフィックス |
|---------------|---------|-----------| | `Unable to locate a Java Runtime` | 6, 12 | `export JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home"` |
| `Unknown command-line option '--tests'` | 5, 11 | `./gradlew :module:testVariantDebugUnitTest` |
| `No answer found for:` (MockK) | 2 | `every { mock.method() } returns value` |
| `No value passed for parameter` | 7, 13 | 不足しているコンストラクタパラメータを追加 |
| `Test not found` (Compose UI) | 17 | `androidTest/` に移動 |
| `NoSuchElementException` (Flow) | 19 | Turbine の `.test {}` を使用 |
| `expected: X but was: Y` | 8, 14 | Mock データを確認 |
| CI失敗、ローカル成功 | 16, 18 | すべてのバリアントをテスト `./gradlew test` |

詳細は `knowledge/test-failure-patterns.md` を参照

## 使い方

### テスト実行

```
User: すべてのunit testを実行して
Assistant: [./gradlew test を実行し結果を分析]

User: UserRepositoryのテストを実行して
Assistant: [./gradlew test --tests "*UserRepositoryTest" を実行し結果を分析]
```

### 失敗分析

テストが失敗すると、このスキルは自動的に:
1. テスト出力ログを解析
2. 失敗パターンを特定（NullPointerException, Mock設定問題等）
3. コード例付きの具体的な修正提案を提供

### 修正提案の取得

```
User: LoginViewModelTestが失敗した理由は?
Assistant: [テスト失敗ログを分析し以下を提供:
- 根本原因の説明
- 関連するコードスニペット
- コード例付きの具体的な修正]
```

## System Instructions

ユーザーが Android テストの実行やテスト失敗の分析をリクエストした場合、以下の手順に従ってください:

### 0. 環境セットアップ & プロジェクト検出（必須 - 常に最初に実行）

テストを実行する前に、環境を確認・設定してください:

#### ステップ1: JAVA_HOME 確認

```bash
# JAVA_HOMEが設定されているか確認
if [ -z "$JAVA_HOME" ]; then
    # macOS: Android Studio JBR を自動検出
    export JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home"
fi

# 確認
$JAVA_HOME/bin/java -version
```

**エラー検出**:
- "Unable to locate a Java Runtime" が表示された場合 → JAVA_HOMEを設定して再試行

#### ステップ2: プロジェクト構造検出

```bash
# ビルドバリアントを検出（存在する場合）
./gradlew tasks --all | grep "test.*UnitTest"

# 出力例（multi-variantプロジェクト）:
# :app:testDevelopDebugUnitTest
# :app:testDevelopReleaseUnitTest
# :app:testProductionDebugUnitTest
```

**ビルドバリアント検出**:
- 複数の `testXxxDebugUnitTest` タスクが存在 → プロジェクトにビルドバリアントあり
- `Unknown command-line option '--tests'` エラー → バリアント固有タスクを使用する必要あり

**BuildType認識**:
本番プロジェクトには複数のBuild Typeが存在することが多い:
- `Debug`: 通常のテスト実行（最も一般的）
- `Release`: リリースビルド
- `CiRelease`: CI専用リリースビルド
- `Minified`: 難読化テストビルド

**総バリアント数** = Product Flavors × Build Types
- 例: 4 flavors × 3 build types = 12 variants
- 日常開発: `Debug` ビルドタイプのみテスト
- コミット前: すべてのflavorsを `Debug` でテスト（`./gradlew :module:testDebug`）
- リリース前: すべてのバリアントをテスト（`./gradlew :module:test`）

#### ステップ3: テストタスク決定

**ビルドバリアントありのプロジェクト**:
```bash
# 単一バリアント（最速、クイックチェック用）
./gradlew :module:testVariantDebugUnitTest

# すべてのバリアント（CI相当、コミット前推奨）
./gradlew :module:test
```

**ビルドバリアントなしのプロジェクト**:
```bash
# 標準コマンドが動作
./gradlew test
./gradlew test --tests "*TestClass"
```

### 1. テスト実行（拡張版）

Bashツールを使用してGradleテストコマンドを実行してください。**重要**: 常に最初にJAVA_HOMEを設定してください（ステップ0から）。

#### パターンA: 標準プロジェクト（ビルドバリアントなし）

```bash
# すべてのunit test
./gradlew test

# 特定のテストクラス
./gradlew test --tests "*UserRepositoryTest"

# 特定のテストメソッド
./gradlew test --tests "*UserRepositoryTest.testGetUser"

# 詳細出力付き
./gradlew test --info
```

#### パターンB: Multi-Variantプロジェクト（本番環境で一般的）

```bash
# JAVA_HOME設定 + 単一バリアントテスト
export JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home" && \
./gradlew :module:testVariantDebugUnitTest

# すべてのバリアント（CI相当、コミット前推奨）
export JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home" && \
./gradlew :module:test

# 特定モジュールの特定バリアント
export JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home" && \
./gradlew :data:testProductionDebugUnitTest
```

#### エラーハンドリング & 自動再試行

**エラー1: `Unknown command-line option '--tests'`**
- **原因**: プロジェクトに複数のビルドバリアントあり
- **修正**: バリアント固有タスクに切り替え
```bash
# バリアントを検出
./gradlew tasks --all | grep "test.*UnitTest"

# バリアントタスクを使用
./gradlew :module:testVariantDebugUnitTest
```

**エラー2: `Unable to locate a Java Runtime`**
- **原因**: JAVA_HOME未設定
- **修正**: JAVA_HOMEを設定して再試行
```bash
export JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home"
./gradlew :module:testVariantDebugUnitTest
```

**重要**:
- 常にプロジェクトルートディレクトリから実行
- 常にAndroidプロジェクトでJAVA_HOMEを設定
- テストコマンドを選択する前にビルドバリアントを検出
- 標準出力と標準エラー出力の両方をキャプチャ
- コミット前にすべてのバリアントのテストを推奨（CIパリティ）

### 2. テスト結果の解析

テスト実行後、出力を分析:

**成功インジケータ**:
- "BUILD SUCCESSFUL"
- "X tests completed, X passed"

**失敗インジケータ**:
- "BUILD FAILED"
- "X tests completed, X failed"
- 出力中のスタックトレース

抽出:
- 失敗したテストクラス名
- 失敗したテストメソッド名
- 例外タイプとメッセージ
- スタックトレースの場所

### 3. 失敗パターン分析

失敗を一般的なパターンと照合（完全なリストは `knowledge/test-failure-patterns.md` を参照）:

**一般的なパターン**:
- Pattern 1: NullPointerException
- Pattern 2: MockK: No Answer Found
- Pattern 3-10: 標準的なテスト失敗
- Pattern 11-16: Multi-variantプロジェクト固有の問題
- Pattern 17-19: 高度なパターン（Compose UI, BuildType, Flow）

### 4. 修正提案の提供

レスポンスを以下の形式でフォーマット:

```markdown
## テスト失敗分析

**失敗したテスト**: `UserRepositoryTest.testGetUser()`

**根本原因**: MockK mock が `userApi.getUser()` に対して設定されていない

**修正方法**:

`UserRepositoryTest.kt:45` で、mock設定を追加:

```kotlin
@Before
fun setup() {
    every { userApi.getUser() } returns Result.success(mockUser)
}
```

**説明**:
テストが `userApi.getUser()` を呼び出していますが、MockKは何を返すべきか分かりません。
`every { ... } returns ...` を使用してmockの動作を定義する必要があります。
```

### 5. ナレッジ参照

一般的なパターンについては、以下のナレッジファイルを参照:

- `knowledge/test-failure-patterns.md` - 一般的なAndroidテスト失敗パターン（19パターン）
- `knowledge/project-template.md` - プロジェクト固有のカスタマイズ用テンプレート
- `templates/fix-suggestions.md` - 修正提案用テンプレート

## 重要事項

- 常にプロジェクトルートから実行
- ログを注意深く解析して正確な失敗場所を特定
- ファイルパスと行番号を提供（例: `UserRepository.kt:25`）
- 修正提案時に実際のテストファイルからコードスニペットを含める
- テスト修正は最小限で焦点を絞ったものに
- プロジェクト固有のパターンについては、プロジェクト固有のナレッジファイルを参照

## カスタマイズ

### プロジェクト固有版の作成

最大の精度（80% → 98%）を得るために、プロジェクト固有版を作成:

1. このスキルをプロジェクトにコピー:
   ```bash
   cp -r android-test-runner your-project/.claude/skills/
   ```

2. `knowledge/project-specific.md` を以下の内容で作成:
   - ビルドバリアントとビルドタイプ
   - プロジェクト標準のテストライブラリ
   - アーキテクチャパターン（Result型、Flow型等）
   - CI/CD設定
   - プロジェクト固有の一般的な失敗パターン

開発ワークフローについては `~/.config/claude-code/AI_DEVELOPMENT_GUIDELINES.md` を参照。

## 成功基準

- ✅ Gradle経由でテストを実行可能
- ✅ 失敗の根本原因を正確に特定（70%以上の精度）
- ✅ 実行可能な修正提案を提供
- ✅ デバッグ時間を30%以上削減

## バージョン

### v1.0 - 汎用版リリース (2025-11-20)
- ✅ Pattern 1-19 を実装
- ✅ Multi-variantプロジェクトサポート
- ✅ JAVA_HOME 自動設定
- ✅ Quick Reference 追加
- ✅ 包括的なドキュメント
- ✅ すべての説明を日本語で記述（視認性・メンテナンス性向上）

android-test-runner v0.2（プロジェクト固有版、98%精度）をベースにしています

