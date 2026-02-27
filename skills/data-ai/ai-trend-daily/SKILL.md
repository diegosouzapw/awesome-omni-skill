---
name: ai-trend-daily
description: "AIトレンドネタ収集"
---

# AIトレンドネタ収集

AI分野に特化したトレンド情報を収集し、`ideas/daily/YYYYMMDD-ai-trend.md` に保存する。

## 実行手順

### 0. 対象分野の理解

以下のAI関連領域を重点的にカバーする：

**コアモデル**
- **LLM/基盤モデル**: GPT, Claude, Gemini, Llama, Mistral, DeepSeek, Qwen, Grok, Command R 等
- **画像生成AI**: Midjourney, Stable Diffusion, DALL-E, FLUX, Adobe Firefly, Ideogram 等
- **動画生成AI**: Sora, Runway, Kling, Pika, Veo, Luma Dream Machine, Hailuo 等
- **音声/音楽生成AI**: ElevenLabs, Suno, Udio, Stable Audio, NotebookLM Audio 等
- **3D生成AI**: Meshy, Tripo, Stable Point Aware 3D 等
- **マルチモーダルAI**: 画像+テキスト+音声+動画の統合モデル

**応用・インフラ**
- **AIエージェント**: Claude Code, GitHub Copilot, Cursor, Devin, MCP, Computer Use, マルチエージェント 等
- **AI研究**: 新アーキテクチャ、学習手法、ベンチマーク、安全性研究 等
- **AI応用/実践**: RAG, ファインチューニング, プロンプトエンジニアリング 等
- **AIインフラ/チップ**: NVIDIA GPU, AMD, Cerebras, Groq, AWS Trainium, Google TPU 等
- **ロボティクス/身体性AI**: Figure AI, Tesla Optimus, Physical Intelligence, VLAモデル 等
- **AI for Science**: AlphaFold, 創薬AI, 気候モデル, 材料科学 等

**ビジネス・社会**
- **AI企業動向**: 資金調達、提携、M&A、人事、決算
- **AI倫理/規制/安全性**: EU AI Act, 著作権問題, 安全性ガイドライン
- **日本のAI動向**: 国内AI企業、政府方針、日本語モデル

### 1. トレンド情報の収集

以下のソースから**並列で**最新のAIトレンド情報を取得する。

---

#### A. Hugging Face Daily Papers（注目AI論文）

- https://huggingface.co/papers
- コミュニティが選んだ注目AI論文の日次キュレーション
- 各論文の**タイトル、論文ページURL（`https://huggingface.co/papers/XXXX.XXXXX`形式）、Upvote数**を取得
- **タイトルは日本語に翻訳して出力**

#### B. はてなブックマーク AI・機械学習

- https://b.hatena.ne.jp/hotentry/it/AI%E3%83%BB%E6%A9%9F%E6%A2%B0%E5%AD%A6%E7%BF%92
- AI・機械学習カテゴリに絞って取得
- 各エントリーの**タイトル、元記事URL、ブックマーク数**を必ず取得
- はてブのエントリーページURLではなく、リンク先の元記事URLを抽出

#### C. Hacker News（AI関連をフィルタ）

- https://news.ycombinator.com/
- フロントページ全記事を取得し、AI/ML/LLM関連のものを選別
- 各記事の**タイトル、HNコメントページURL（`https://news.ycombinator.com/item?id=XXXXX`形式）、ポイント数**を取得
- **タイトルは日本語に翻訳して出力**
- AI関連でないものも話題性が高ければ（200pt以上）「その他注目」として含める

#### D. AI企業公式ブログ（最新記事チェック）

以下の公式ブログから最新1〜3記事をチェックし、新着があれば取得。
新着がないブログはスキップしてよい。

**LLM/基盤モデル企業**

| 企業 | URL | 注目ポイント |
|------|-----|-------------|
| **OpenAI** | https://openai.com/blog | GPT, Sora, DALL-E, o1/o3, API更新 |
| **Anthropic** | https://www.anthropic.com/news | Claude, 安全性研究, MCP |
| **Google DeepMind** | https://deepmind.google/discover/blog/ | Gemini, Veo, AlphaFold |
| **Meta AI** | https://ai.meta.com/blog/ | Llama, SAM, オープンソース |
| **xAI** | https://x.ai/blog | Grok, Colossus |
| **Mistral AI** | https://mistral.ai/news/ | Mistral/Mixtral, 欧州AI |
| **Cohere** | https://cohere.com/blog | Command R, 企業向けLLM, RAG |

**AI開発プラットフォーム**

| 企業 | URL | 注目ポイント |
|------|-----|-------------|
| **Hugging Face** | https://huggingface.co/blog | オープンソースモデル, ツール |
| **Microsoft AI** | https://blogs.microsoft.com/ai/ | Copilot, Azure AI, Phi |
| **Amazon AWS AI** | https://aws.amazon.com/blogs/machine-learning/ | Bedrock, Trainium, Nova |
| **NVIDIA** | https://blogs.nvidia.com/blog/category/deep-learning/ | GPU, 推論最適化, NIM |

**画像/動画/音声生成**

| 企業 | URL | 注目ポイント |
|------|-----|-------------|
| **Stability AI** | https://stability.ai/news | Stable Diffusion, Stable Audio, Stable Video |
| **Runway** | https://runwayml.com/blog | Gen-4, プロ向け動画生成 |
| **ElevenLabs** | https://elevenlabs.io/blog | 音声合成, 音声クローン |

**ロボティクス/AI for Science**

| 企業 | URL | 注目ポイント |
|------|-----|-------------|
| **Figure AI** | https://www.figure.ai/news | ヒューマノイドロボット |
| **Physical Intelligence** | https://www.physicalintelligence.company/blog | 汎用ロボットAI, pi0 |
| **Sakana AI** | https://sakana.ai/blog/ | 日本発AI研究, AI Scientist |

各記事の**タイトル、URL、公開日**を取得。

#### E. Reddit AI系サブレッド（14サブレッド）

- **重要**: WebFetchツールはreddit.comをブロックするため、**Bashツールでcurlコマンドを使用**すること
- 各サブレッドから `/hot.json?t=day&limit=10` で上位10件を取得
- **old.reddit.com**を使用（www.reddit.comではない）
- User-Agentヘッダーを設定: `"User-Agent: ai-trend-collector/1.0 (trend analysis tool)"`
- 各記事の**タイトル、Redditコメントページの完全URL、投票数（ups）、コメント数**を取得
- **タイトルは日本語に翻訳して出力**

取得例（Bashツールで実行）:
```bash
curl -s -H "User-Agent: ai-trend-collector/1.0 (trend analysis tool)" \
  "https://old.reddit.com/r/OpenAI/hot.json?t=day&limit=10" | \
  jq -r '.data.children[] | "\(.data.title)|\(.data.ups)|\(.data.num_comments)|https://www.reddit.com\(.data.permalink)"'
```

データ構造:
- `data.children[].data.title`: タイトル
- `data.children[].data.ups`: 投票数
- `data.children[].data.num_comments`: コメント数
- `data.children[].data.permalink`: パス（`https://www.reddit.com` + permalink で完全URL）

**LLM/チャットAI系（5サブレッド）**:
- r/OpenAI - OpenAI製品・GPT関連
- r/LocalLLaMA - ローカルLLM・オープンソースモデル
- r/ClaudeAI - Claude関連
- r/ChatGPT - ChatGPT使い方・活用
- r/Singularity - AGI・技術的特異点・AI未来予測

**AIコーディング/エージェント系（2サブレッド）**:
- r/ClaudeCode - Claude Code関連
- r/CursorAI - Cursor AI関連

**画像/動画/音声生成AI系（3サブレッド）**:
- r/StableDiffusion - Stable Diffusion・画像生成
- r/midjourney - Midjourney
- r/aivideo - AI動画生成全般

**AI全般/研究（2サブレッド）**:
- r/MachineLearning - 機械学習研究
- r/artificial - AI全般ニュース

**中国AI/新興（2サブレッド）**:
- r/DeepSeek - DeepSeek関連
- r/AIAgents - AIエージェント全般

#### F. 日本語AI専門メディア

以下のサイトから最新AI記事を取得：

| メディア | URL | 注目ポイント |
|---------|-----|-------------|
| **ITmedia AI+** | https://www.itmedia.co.jp/aiplus/ | AI技術ニュース、企業導入事例 |
| **テクノエッジ** | https://www.techno-edge.net/ | 生成AIウィークリー、深掘り記事 |
| **Ledge.ai** | https://ledge.ai/ | 日本最大級AI特化ニュース |

各記事の**タイトル、URL**を取得。AI関連記事のみ抽出。

#### G. AI個人ブログ/ニュースレター

以下の影響力のあるAIブログから最新記事をチェック：

| ブログ | URL | 特徴 |
|--------|-----|------|
| **Simon Willison** | https://simonwillison.net/ | LLM実践活用、ツール分析、更新頻度高 |
| **Latent Space** | https://www.latent.space/ | LLM・AIインフラの深い技術分析 |

最新1〜3記事をチェックし、注目記事があれば取得。

#### H. GitHub Trending（ML/AI）

- https://github.com/trending?since=daily
- 日次トレンドからAI/ML関連のリポジトリを抽出
- 各リポジトリの**名前、URL、説明、本日のスター数**を取得
- 特にモデル実装、AIツール、フレームワークに注目

#### I. 中国AI動向

中国AIの動向は英語キュレーションソースで追跡：

| ソース | URL | 特徴 |
|--------|-----|------|
| **Interconnected (Kevin Xu)** | https://interconnected.blog/ | 中国テック・AI動向の英語解説 |

DeepSeek, Qwen, Kimi, GLM等の中国AIモデルの動向にも注目。

---

### 2. 分析

収集した情報を以下の観点で分析：

**カテゴリ分類**
各記事を以下のカテゴリに分類：
- 🧠 **LLM/基盤モデル**: 新モデルリリース、性能比較、API更新
- 🎨 **画像生成AI**: 新モデル、技法、ツール
- 🎬 **動画生成AI**: 新サービス、品質比較
- 🎵 **音声/音楽AI**: 音声合成、音楽生成、Audio系
- 🤖 **AIエージェント/コーディング**: 開発ツール、自動化、MCP
- 📄 **AI研究/論文**: 新手法、アーキテクチャ、ベンチマーク
- 🏢 **AI企業動向**: 資金調達、提携、規制、人事
- 🔧 **AI応用/実践**: RAG、プロンプト、ファインチューニング、活用事例
- ⚖️ **AI倫理/規制/安全性**: ガイドライン、法規制、安全性研究
- 🦾 **ロボティクス/身体性AI**: ヒューマノイド、VLA、自律システム
- 💻 **AIインフラ/チップ**: GPU、推論最適化、MLOps
- 🔬 **AI for Science**: 創薬、タンパク質構造、気候、材料
- 🌏 **中国AI**: DeepSeek, Qwen, 中国AI政策

**重要度評価**
- 🔥🔥🔥: 業界を変えるレベル（新モデルリリース、重大発表、画期的論文）
- 🔥🔥: 注目すべき動き（機能アップデート、重要な研究成果、業界トレンド）
- 🔥: 参考情報（チュートリアル、活用事例、マイナーアップデート）

**クロスソース分析**
- 複数ソースで同じ話題が取り上げられていれば「ホットトピック」としてハイライト
- ソース横断でトレンドの方向性を要約

### 3. 出力

**まず「AI トレンド収集完了。」というメッセージを返してから、結果を `ideas/daily/YYYYMMDD-ai-trend.md` に保存。**

以下のフォーマットで出力：

```markdown
# AI トレンド: YYYY-MM-DD

## 本日のホットトピック

> 複数ソースで話題になっているトピックを1〜3個、簡潔に要約

## 注目ニュース TOP10

| # | タイトル | カテゴリ | 重要度 | ソース | メモ |
|---|---------|---------|--------|--------|------|
| 1 | [タイトル](URL) | 🧠 LLM | 🔥🔥🔥 | OpenAI Blog | ポイント |
| 2 | ... | | | | |

---

## AI論文（Hugging Face Daily Papers）

| タイトル | Upvotes | カテゴリ | 概要 |
|---------|---------|---------|------|
| [日本語タイトル](HF論文URL) | XX | 🧠 LLM | 一行要約 |

## はてブ AI・機械学習（日本市場）

### 注目トピック

| タイトル | ブクマ数 | カテゴリ | メモ |
|---------|---------|---------|------|
| [タイトル](元記事URL) | XXX users | 🧠 LLM等 | ポイント |

### 全エントリー

1. [タイトル](元記事URL) (XXX users) - 概要
2. ...

## Hacker News（AI関連）

### 注目トピック

| タイトル | ポイント | カテゴリ | メモ |
|---------|---------|---------|------|
| [日本語タイトル](HNコメントページURL) | XXXpt | 🤖 Agent等 | ポイント |

### 全エントリー

1. [日本語タイトル](HNコメントページURL) (XXXpt) - 概要
2. ...

## AI企業公式ブログ 新着

| 企業 | タイトル | 公開日 | カテゴリ | 概要 |
|------|---------|--------|---------|------|
| OpenAI | [タイトル](URL) | MM/DD | 🧠 LLM | 概要 |

（新着がない企業は省略。全社新着なしの場合「本日の新着なし」と記載）

## Reddit AI系（14サブレッド）

### 注目トピック

| タイトル | 投票数 | コメント数 | カテゴリ | サブレッド | メモ |
|---------|--------|-----------|---------|-----------|------|
| [日本語タイトル](RedditコメントページURL) | XXX ups | XXX | 🧠 LLM等 | r/OpenAI | ポイント |

### カテゴリ別エントリー

#### LLM/チャットAI系
1. [日本語タイトル](URL) (XXX ups, XXX comments) - r/OpenAI - 概要
2. ...

#### AIコーディング/エージェント系
1. [日本語タイトル](URL) (XXX ups, XXX comments) - r/ClaudeCode - 概要
2. ...

#### 画像/動画/音声生成AI系
1. [日本語タイトル](URL) (XXX ups, XXX comments) - r/StableDiffusion - 概要
2. ...

#### AI全般/研究
1. [日本語タイトル](URL) (XXX ups, XXX comments) - r/MachineLearning - 概要
2. ...

#### 中国AI/新興
1. [日本語タイトル](URL) (XXX ups, XXX comments) - r/DeepSeek - 概要
2. ...

## GitHub Trending（AI/ML）

| リポジトリ | 本日★ | 概要 |
|-----------|-------|------|
| [owner/repo](URL) | +XXX | 概要 |

## 日本語AI専門メディア

| メディア | タイトル | カテゴリ | 概要 |
|---------|---------|---------|------|
| ITmedia AI+ | [タイトル](URL) | 🏢 企業 | 概要 |
| テクノエッジ | [タイトル](URL) | 🧠 LLM | 概要 |
| Ledge.ai | [タイトル](URL) | 🤖 Agent | 概要 |

## AI個人ブログ/ニュースレター 注目記事

| ブログ | タイトル | カテゴリ | 概要 |
|--------|---------|---------|------|
| Simon Willison | [タイトル](URL) | 🔧 実践 | 概要 |
| Latent Space | [タイトル](URL) | 🧠 LLM | 概要 |

---

## 今日のAIトレンド要約

- **LLM**: （一行要約）
- **画像生成**: （一行要約）
- **動画生成**: （一行要約）
- **音声/音楽**: （一行要約）
- **エージェント**: （一行要約）
- **ロボティクス**: （一行要約）
- **AIインフラ**: （一行要約）
- **中国AI**: （一行要約）
- **企業動向**: （一行要約）

（動きがないカテゴリは省略可）
```

## 注意事項

- WebFetchツールを使用して情報を取得（Reddit以外）
- **Redditは必ずBashツールのcurlで取得**（WebFetchはブロックされるため）
- **すべての記事にURLリンクを必ず含める（リンクなしは不可）**
- **はてブは元記事のURLを必ず取得**（はてブページURLではなく）
- **Hacker NewsはHNコメントページURL（`item?id=`形式）を使用**（元記事URLではなく）
- **英語・中国語タイトルは日本語に翻訳して出力**
- Reddit APIレート制限に注意（1分あたり60リクエスト程度）
- 投票数（ups）/コメント数/ポイント数/ブックマーク数が高い記事を優先
- 出力ファイルのYYYYMMDDは実行日の日付を使用
- AI企業ブログは新着がなければスキップ可
- **収集は可能な限り並列で実行し、速度を最大化すること**
- **Papers With Code は2025年にシャットダウン済みなので使用しないこと**
