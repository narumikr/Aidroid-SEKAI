# Compound Component 完全実装例

Composer コンポーネントを Compound Components パターンで実装する完全な例。

---

## Context と型定義

```tsx
import { createContext, use, useRef, useState } from 'react'

// 汎用インターフェース定義
interface ComposerState {
  input: string
  attachments: Attachment[]
  isSubmitting: boolean
}

interface ComposerActions {
  update: (updater: (state: ComposerState) => ComposerState) => void
  submit: () => void
}

interface ComposerMeta {
  inputRef: React.RefObject<TextInput>
}

interface ComposerContextValue {
  state: ComposerState
  actions: ComposerActions
  meta: ComposerMeta
}

const ComposerContext = createContext<ComposerContextValue | null>(null)
```

## サブコンポーネント定義

```tsx
function ComposerProvider({ children, state, actions, meta }: {
  children: React.ReactNode
  state: ComposerState
  actions: ComposerActions
  meta: ComposerMeta
}) {
  return (
    <ComposerContext value={{ state, actions, meta }}>
      {children}
    </ComposerContext>
  )
}

function ComposerFrame({ children }: { children: React.ReactNode }) {
  return <form>{children}</form>
}

function ComposerInput({ placeholder }: { placeholder?: string }) {
  const {
    state,
    actions: { update },
    meta: { inputRef },
  } = use(ComposerContext)!
  return (
    <TextInput
      ref={inputRef}
      value={state.input}
      placeholder={placeholder}
      onChangeText={(text) => update((s) => ({ ...s, input: text }))}
    />
  )
}

function ComposerSubmit({ label = 'Send' }: { label?: string }) {
  const {
    state: { isSubmitting },
    actions: { submit },
  } = use(ComposerContext)!
  return (
    <Button onPress={submit} disabled={isSubmitting}>
      {label}
    </Button>
  )
}

function ComposerHeader() {
  return <header>Compose</header>
}

function ComposerFooter({ children }: { children: React.ReactNode }) {
  return <footer className="flex gap-2">{children}</footer>
}

function ComposerFormatting() {
  return <button aria-label="Formatting">B</button>
}

function ComposerEmojis() {
  return <button aria-label="Emoji">:)</button>
}

function ComposerAttachments() {
  return <button aria-label="Attach">+</button>
}
```

## Compound Component としてエクスポート

```tsx
export const Composer = {
  Provider: ComposerProvider,
  Context: ComposerContext,
  Frame: ComposerFrame,
  Input: ComposerInput,
  Submit: ComposerSubmit,
  Header: ComposerHeader,
  Footer: ComposerFooter,
  Formatting: ComposerFormatting,
  Emojis: ComposerEmojis,
  Attachments: ComposerAttachments,
}
```

## 使用例

```tsx
function ChannelComposer() {
  return (
    <Composer.Frame>
      <Composer.Header />
      <Composer.Input />
      <Composer.Footer>
        <Composer.Attachments />
        <Composer.Formatting />
        <Composer.Emojis />
        <Composer.Submit />
      </Composer.Footer>
    </Composer.Frame>
  )
}
```
