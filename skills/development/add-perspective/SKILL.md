---
name: add-perspective
description: 振り返り観点を追加するガイド。ユーザー指摘から学習し、類似問題を将来検出できるようにする。観点、perspective、チェック追加時に使用。
---

# 振り返り観点追加ガイド

ユーザーからの指摘や問題発見時に、類似問題を将来の振り返りで検出できるよう観点を追加する手順。

## 使用タイミング

- `[ACTION_REQUIRED: /add-perspective]` が表示されたとき
- ユーザーから「動いてる？」「正常？」等の指摘を受けたとき
- 振り返りで新しいチェック観点が必要と判断したとき

## 手順

### 1. 問題の分析

まず、指摘された問題の根本原因を特定する。

```markdown
| 項目 | 内容 |
|------|------|
| 問題の概要 | 何が起きたか |
| 根本原因 | なぜ発生したか |
| 検出方法 | どうすれば事前に気づけたか |
```

### 2. 既存観点の確認

新しい観点が本当に必要か確認する。

```bash
# 既存の観点を確認（PERSPECTIVES配列全体を表示）
grep -A 100 "PERSPECTIVES = \[" .claude/hooks/reflection_self_check.py | head -150
```

既存観点で検出可能な場合は追加不要。キーワードの拡充で対応できる場合はキーワード追加のみ。

### 3. 観点の定義

新しい観点が必要な場合、以下を定義する。

| フィールド | 説明 | 例 |
|-----------|------|-----|
| `id` | 一意の識別子（snake_case） | `ci_failure_analysis` |
| `name` | 日本語の表示名 | `CI失敗分析` |
| `description` | 確認すべき内容 | `CI失敗時に根本原因を分析したか` |
| `keywords` | 検出用キーワード（正規表現） | `[r"CI.*失敗", r"根本原因"]` |

### 4. reflection_self_check.py への追加

`.claude/hooks/reflection_self_check.py` の `PERSPECTIVES` 配列に追加:

```python
# Issue #XXXX: [問題の説明]
{
    "id": "new_perspective_id",
    "name": "観点の表示名",
    "description": "確認すべき内容の説明",
    "keywords": [
        r"キーワード1",
        r"キーワード2",
        r"複合.*パターン",
    ],
},
```

### 5. reflect SKILL.md への追加

`reflection_self_check.py`のPERSPECTIVESに追加した新観点を、`.claude/skills/reflect/SKILL.md`のセクション6「改善点の洗い出し」の観点チェックテーブルにも反映する:

```markdown
| N | 新観点の名前 | 確認すべき内容 | #XXXX |
```

### 6. テストの追加

`.claude/hooks/tests/test_reflection_self_check.py` にテストを追加:

```python
def test_detects_new_perspective(self):
    """新観点が正しく検出される."""
    transcript = "キーワード1を含むテキスト"
    missing = get_missing_perspectives(transcript)
    perspective_ids = [p["id"] for p in missing]
    assert "new_perspective_id" not in perspective_ids
```

### 7. Issueの作成（任意）

大きな変更の場合はIssueを作成してからworktreeで作業する。

## キーワード設計のベストプラクティス

| 項目 | 推奨 |
|------|------|
| **複合パターン** | 単一キーワードより `r"CI.*失敗"` のような複合が誤検知を減らす |
| **正規表現** | `r"(Pre|Post|Stop)"` でOR条件も可能 |
| **網羅性** | 同じ意味の異なる表現を含める（例: 失敗、エラー、問題） |
| **テスト** | 実際のtranscriptでキーワードが検出されることを確認 |

## 追加しない方が良いケース

| ケース | 理由 |
|--------|------|
| 一回限りの特殊な問題 | 再発可能性が低い |
| 既存観点のキーワード拡充で対応可能 | 観点の重複を避ける |
| 主観的な評価基準 | キーワードで検出困難 |

## 実例

### Issue #2289: 「対応済み」判断の検証

**問題**: 「既に対応済み」と判断したが、実際には仕組みが有効に機能していなかった。

**追加した観点**:

```python
{
    "id": "already_handled_check",
    "name": "「対応済み」判断の検証",
    "description": "「対応済み」と判断した場合、その仕組みの実行タイミング（Pre/Post/Stop）を確認し、実際に有効か検証したか",
    "keywords": [
        r"対応済み.*検証",
        r"実行タイミング",
        r"(Pre|Post|Stop)",
        r"フック.*確認",
        r"仕組み.*有効",
        r"対応済み.*なし",
    ],
},
```

## チェックリスト

- [ ] 既存観点で対応できないか確認した
- [ ] `id`, `name`, `description`, `keywords` を定義した
- [ ] `reflection_self_check.py` の PERSPECTIVES に追加した
- [ ] `execute.md` のセクション8に追加した
- [ ] テストを追加した
- [ ] Pythonの構文エラーがないことを確認した
