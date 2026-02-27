---
name: feature-engineering
description: モデルの性能を向上させるために、既存のデータから新しい特徴量を作成する。
---

# Feature Engineering

## 概要
モデルの性能を向上させるために、既存のデータから新しい特徴量を作成する。

## 手順
1.  **ドメイン知識の適用**: ビジネスロジックに基づいた特徴量作成。
2.  **変換**: 対数変換、Box-Cox変換、標準化/正規化。
3.  **エンコーディング**: One-Hot Encoding, Label Encoding, Target Encoding.
4.  **相互作用**: 変数間の掛け合わせ、割り算。
5.  **集約**: グループごとの統計量（平均、最大、最小、分散など）の算出。
6.  **時系列特徴量**: ラグ特徴量、移動平均など（時系列データの場合）。

## 使用ツール・ライブラリ
- pandas, numpy, scikit-learn

## 成果物の保存場所
- 特徴エンジニアリング、学習、検証用ノートブック: `project/src/03_main.ipynb`
- 特徴量エンジニアリング用コード: `project/src/modules/feature_engineering.py`
- 特徴量生成後のデータセット: `project/data/[元データ名]-feature_engineered.csv`
- 実験結果ログ: `project/reports/experiment/[番号]-[アプローチの概要].md`
