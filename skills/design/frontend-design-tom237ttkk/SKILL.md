---
name: frontend-design
description: 「Clarity / Deference / Depth」と、1プロダクト=1焦点の構成・大きいタイポ・余白・最小CTAを抽象化して、Web UI設計に適用する。
---

# Apple-like Web Design Skill

## Goal

Apple.com 的な **情報設計・タイポ・余白・CTA・モーションの節度**を再現可能なルールに落とし込む。

## Inputs (ask if missing)

1. 画面目的（何を達成させたいか）
2. 主役（1 つに絞る）：プロダクト/機能/価値/行動
3. ユーザーの最上位タスク（上から 3 つ）
4. 必須コンテンツ（法務表記/価格/比較など）
5. ブランド制約（色・トーン・禁止表現）
6. 対象デバイス比率（Mobile/Tablet/Desktop）
7. 参照：既存画面 or Figma or 競合 URL

## Core Principles (abstracted)

### 1) Clarity

- 1 画面=1 メッセージ。視線の“最短経路”を作る
- 文字は大きく、短く、誤読しない（曖昧な装飾より階層で語る）

### 2) Deference

- UI は目立たない。コンテンツ（価値・製品・データ）を主役にする
- 装飾は“意味があるときだけ”（影や線は情報の分離のため）

### 3) Depth

- 階層は「余白」「重なり」「動き」で伝える（色で誤魔化さない）
- スクロールはストーリー。章立てで理解を前進させる

## Layout System (rules)

- **Fold（最初の 1 スクリーン）に置くのは最大 3 要素**
  1. 見出し（主張）
  2. サブ見出し（補足/ベネフィット）
  3. CTA（最大 2 つ）
- 1 セクション=1 論点。セクション間は十分に離す（“呼吸”を作る）
- 密度は「画面を埋めない」方向に倒す。迷ったら削る
- 比較/仕様は“後半”に寄せ、上流は価値訴求に集中

## Typography (rules)

- 見出しは大きく短く（1 行〜2 行）。句読点は最小限
- サブ見出しは「何が良いか」を 1 文で言い切る
- 本文は読みやすい行長（長文化するなら段落を割る）
- Web 実装はまず **system-ui スタック**（SF Pro の Web 配布は前提にしない）
  - 例: `system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif`

## Color & Material (rules)

- 基本はニュートラル（白/黒/グレー）＋アクセント 1 色(0f83fd)
- アクセントは“行動”か“状態”にのみ使う（装飾に使わない）
- 罫線や影は最小限。必要なら「意味（区切り/階層）」を明確化

## CTA Pattern (Apple-like)

- 主要 CTA: 1 つ（例：購入/開始/相談）
- 副次 CTA: 1 つ（例：詳しく見る）
- CTA 文言は短く、動詞で。迷ったら「Learn more / Buy」型に寄せる

## Imagery & Content

- 画像は“説明”ではなく“確信”を作る（主役を大きく、背景は静かに）
- 追加要素（アイコン/バッジ/装飾）は、主役の邪魔なら削除
- 機能列挙は避け、ストーリー順に提示（Why → What → Proof）

## Motion (optional)

- スクロールで段階的に出す（progressive disclosure）
- 動きは「理解を助ける」ためだけに使う（見せるために動かさない）
- `prefers-reduced-motion` 対応を必須にする

## Output format (what to produce)

### A) 3 案の情報設計

- Variant 1: 最小（Apple 寄り）—要素を削り切る
- Variant 2: バランス —価値 → 証拠 → 比較 →FAQ
- Variant 3: データ重視 —比較/仕様を早めに出す（ただし Fold は守る）

### B) セクション構成（章立て）

例:

1. Hero（主張 + サブ + CTA）
2. Proof（1 つの強い根拠：数値/実例/デモ）
3. Benefits（3 点まで）
4. How it works（3 ステップ）
5. Compare / Specs（必要なら）
6. FAQ / Trust（不安の解消）
7. Final CTA

### C) Design tokens（初期値）

```css
:root {
  --container-max: 1120px;
  --gutter: 24px;
  --radius: 16px;
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 24px;
  --space-6: 32px;
  --space-7: 48px;
  --space-8: 64px;

  --text-hero: clamp(40px, 5vw, 72px);
  --text-h1: clamp(28px, 3vw, 48px);
  --text-body: 16px;
  --line-body: 1.6;
}
```
