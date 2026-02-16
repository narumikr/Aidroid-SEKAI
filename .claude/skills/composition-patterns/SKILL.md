---
name: composition-patterns
description:
  このスキルは、ユーザーが「コンポーネントのリファクタリング」、「Booleanプロパティ増殖の修正」、「Compound Componentsの構築」、「コンポーネントAPIの設計」、「コンポーネント・アーキテクチャのレビュー」を依頼する場合、またはReactのコンポジションパターン、コンテキストプロバイダー、コンポーネントライブラリを扱う際に使用してください。React 19のAPI変更（use() フック、ref プロパティの仕様変更など）にも対応しています。
license: MIT
metadata:
  author: vercel
  version: "2.0.0"
---

# React Composition パターン

柔軟でメンテナンスしやすい React コンポーネントを構築するための Composition パターン集。
boolean props の増殖を避け、Compound Components、state のリフトアップ、内部構造の合成を活用する。

## 適用タイミング

以下の場面でこのガイドラインを参照すること:

- boolean props が多いコンポーネントのリファクタリング時
- 再利用可能なコンポーネントライブラリの構築時
- 柔軟なコンポーネント API の設計時
- コンポーネントアーキテクチャのレビュー時
- Compound Components や Context Provider を扱う時

## ルールカテゴリ（優先度順）

| 優先度 | カテゴリ           | 影響度 | 詳細リファレンス                          |
| ------ | ------------------ | ------ | ----------------------------------------- |
| 1      | コンポーネント設計 | HIGH   | `references/architecture-patterns.md`     |
| 2      | 状態管理           | MEDIUM | `references/state-management-patterns.md` |
| 3      | 実装パターン       | MEDIUM | `references/implementation-patterns.md`   |
| 4      | React 19 API       | MEDIUM | `references/react19-api-changes.md`       |

## クイックリファレンス

### 1. コンポーネント設計 (HIGH)

- **Boolean Props の増殖を避ける** — boolean props で振る舞いをカスタマイズせず、Composition を使う
- **Compound Components を使う** — 共有 Context を使って複雑なコンポーネントを構造化する

### 2. 状態管理 (MEDIUM)

- **状態管理を UI から分離する** — Provider だけが状態管理の方法を知っている状態にする
- **汎用 Context インターフェース** — state, actions, meta による汎用インターフェースを定義し、依存性注入を可能にする
- **State を Provider にリフトアップ** — 兄弟コンポーネントからアクセスできるよう、state を Provider に引き上げる

### 3. 実装パターン (MEDIUM)

- **明示的なバリアント** — boolean モードの代わりに、明示的なバリアントコンポーネントを作成する
- **Children over Render Props** — renderX props ではなく children で Composition する

### 4. React 19 API (MEDIUM)

> **React 19+ 専用。** React 18 以前を使用している場合はスキップすること。

- **forwardRef 不要** — `ref` を通常の props として受け取る
- **use() を使用** — `useContext()` の代わりに `use()` を使う

## 核心原則（判断に迷った時の指針）

1. **Boolean props が3つ以上** → Composition に切り替える
2. **UI が state 実装を知っている** → Provider に分離する
3. **兄弟コンポーネントで state 共有** → Provider にリフトアップする
4. **render props を使っている** → children で代替できないか検討する

## 詳細リファレンス

パターンの詳細な説明とコード例:

- **`references/architecture-patterns.md`** — Boolean props 回避と Compound Components の詳細パターン
- **`references/state-management-patterns.md`** — 状態分離、Context インターフェース設計、state リフトアップの詳細
- **`references/implementation-patterns.md`** — 明示的バリアントと children Composition の実装テクニック
- **`references/react19-api-changes.md`** — React 19 の ref / use() / Context 構文の変更点

## 実装例

完全な実装例:

- **`examples/compound-component-example.md`** — Composer を Compound Components で実装する完全な例（型定義・Context・サブコンポーネント・エクスポートまで）
- **`examples/provider-pattern-example.md`** — 同じ UI を異なる Provider で使い回す実装例（ローカル state / グローバル同期 state / 兄弟コンポーネントからのアクセス）

## テンプレート

- **`templates/component-review-checklist.md`** — コンポーネント設計レビュー用チェックリスト（設計・状態管理・実装パターン・React 19 API）

## 参考リンク

1. [React 公式ドキュメント](https://react.dev)
2. [Context によるデータの受け渡し](https://react.dev/learn/passing-data-deeply-with-context)
3. [use() API リファレンス](https://react.dev/reference/react/use)
