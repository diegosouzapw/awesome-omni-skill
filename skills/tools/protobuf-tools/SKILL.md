---
name: protobuf-tools
description: Protocol Buffers (protobuf) の使用ガイド。概要と各ガイド（Style Guide, Best Practices, Tools）へのリンクを提供する。詳細はサブファイルを参照。
globs:
  - "**/*.proto"
  - "**/gen/**"
  - "**/*protobuf*/**"
  - "**/*grpc*/**"
---

# Protocol Buffers (protobuf) Tools & Guide

## 概要

Protocol Buffers (protobuf) は、Google が開発した言語中立・プラットフォーム中立な拡張可能な構造化データのシリアライズメカニズムです。

詳細な情報は以下のファイルに分割されています。必要に応じて参照してください。

### 📚 詳細ガイド

1.  **[Style Guide](./guides/style-guide.md)**
    *   File structure
    *   Naming conventions
    *   Basic examples

2.  **[Best Practices](./guides/best-practices.md)**
    *   Versioning
    *   Field management
    *   Common patterns (Timestamp, Empty)

3.  **[Tools](./guides/tools.md)**
    *   `protoc` basic usage
    *   `buf` CLI summary

## Reference

*   [Protocol Buffers Documentation](https://protobuf.dev/): 公式ドキュメント。
*   [Language Guide (proto3)](https://protobuf.dev/programming-guides/proto3/): proto3 の構文ガイド。
*   [Style Guide](https://protobuf.dev/programming-guides/style/): 公式スタイルガイド。
*   [Buf Documentation](https://buf.build/docs/): Buf ツールのドキュメント。
*   [Buf Lint Rules](https://buf.build/docs/lint/rules/): Buf が提供するスタイル・lint ルールの一覧。
