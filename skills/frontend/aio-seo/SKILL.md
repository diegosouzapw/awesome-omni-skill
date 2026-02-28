---
name: aio-seo
description: "AI Search Engine Optimization。JSON-LD構造化データ、Schema.org、AIクローラー対応。ランディングページ、製品ページ作成時に自動適用。"
---

# AIO/SEO - AI Search Engine Optimization

AIクローラー（ChatGPT、Claude、Perplexity等）向けの構造化データ実装スキル。

## 参照

glasswerks LPの実装例:
```
/Users/kimny/Dropbox/_DevProjects/_LandingPage/glasswerks-lp/index.html
```

## 基本原則

### 1. JSON-LD形式で埋め込み

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@graph": [...]
}
</script>
```

### 2. 日英バイリンガル対応

```json
"description": "日本語説明 | English description"
```

### 3. 関連エンティティのリンク

```json
"parentOrganization": {
  "@id": "https://www.glasswerks.jp/#organization"
}
```

## MUED用スキーマ

### Organization

| プロパティ | 値 |
|-----------|-----|
| name | MUED |
| alternateName | ミュード, MUED LMS, MUEDnote |
| slogan | Making creativity visible. |
| parentOrganization | glasswerks inc. |

### SoftwareApplication（MUEDnote）

| プロパティ | 値 |
|-----------|-----|
| name | MUEDnote |
| applicationCategory | ProductivityApplication |
| operatingSystem | iOS |
| offers.price | 0 |

### knowsAbout（キーワード）

```
AI時代の創作支援, AI-era Creative Support
音楽制作, Music Production
判断の記録, Decision Recording
クリエイティブワークフロー, Creative Workflow
音声文字起こし, Speech Transcription
練習ログ, Practice Logging
セルフコーチング, Self-Coaching
メタ認知, Metacognition
```

## Next.js実装

```tsx
// lib/seo/structured-data.ts
export const structuredData = {
  "@context": "https://schema.org",
  "@graph": [
    // Organization
    // SoftwareApplication
    // WebSite
  ]
};

// app/layout.tsx
<script
  type="application/ld+json"
  dangerouslySetInnerHTML={{
    __html: JSON.stringify(structuredData)
  }}
/>
```

## チェックリスト

- [ ] Organization スキーマ実装
- [ ] SoftwareApplication スキーマ実装
- [ ] WebSite スキーマ実装
- [ ] 日英バイリンガル description
- [ ] glasswerks との parentOrganization リンク
- [ ] Google Rich Results Test で検証
  - https://search.google.com/test/rich-results
- [ ] Schema.org Validator で検証
  - https://validator.schema.org/
