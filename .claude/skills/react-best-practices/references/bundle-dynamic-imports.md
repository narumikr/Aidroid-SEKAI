---
title: 重いコンポーネントには動的インポートを使用する
impact: CRITICAL
impactDescription: directly affects TTI and LCP
tags: bundle, dynamic-import, code-splitting, next-dynamic
---

## 重いコンポーネントには動的インポートを使用する

初期レンダリングに不要な大きなコンポーネントを遅延読み込みするために`next/dynamic`を使用します。

**誤り（Monacoがメインチャンクとともにバンドルされてしまう〜300KB）：**

```tsx
import { MonacoEditor } from './monaco-editor'

function CodePanel({ code }: { code: string }) {
  return <MonacoEditor value={code} />
}
```

**正しい（Monacoをオンデマンドで読み込む）：**

```tsx
import dynamic from 'next/dynamic'

const MonacoEditor = dynamic(
  () => import('./monaco-editor').then(m => m.MonacoEditor),
  { ssr: false }
)

function CodePanel({ code }: { code: string }) {
  return <MonacoEditor value={code} />
}
```
