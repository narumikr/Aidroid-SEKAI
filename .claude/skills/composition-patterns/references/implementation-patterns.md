# 実装パターン

**影響度: MEDIUM**

Compound Components と Context Provider を実装するための具体的なテクニック。

---

## 1. 明示的なコンポーネントバリアントを作成する

**影響度: MEDIUM（自己文書化されたコード、隠れた条件分岐なし）**

boolean props を大量に持つ1つのコンポーネントではなく、明示的なバリアントコンポーネントを作成する。各バリアントは必要なパーツを組み合わせる。コード自体がドキュメントになる。

### 悪い例: 1つのコンポーネントに多数のモード

```tsx
// このコンポーネントが実際に何をレンダリングするのか？
<Composer
  isThread
  isEditing={false}
  channelId='abc'
  showAttachments
  showFormatting={false}
/>
```

### 良い例: 明示的なバリアント

```tsx
// 何がレンダリングされるか一目瞭然
<ThreadComposer channelId="abc" />

// または
<EditMessageComposer messageId="xyz" />

// または
<ForwardMessageComposer messageId="123" />
```

各実装は固有で、明示的で、自己完結している。それでいて共有パーツを使い回せる。

### 実装例

```tsx
function ThreadComposer({ channelId }: { channelId: string }) {
  return (
    <ThreadProvider channelId={channelId}>
      <Composer.Frame>
        <Composer.Input />
        <AlsoSendToChannelField channelId={channelId} />
        <Composer.Footer>
          <Composer.Formatting />
          <Composer.Emojis />
          <Composer.Submit />
        </Composer.Footer>
      </Composer.Frame>
    </ThreadProvider>
  )
}

function EditMessageComposer({ messageId }: { messageId: string }) {
  return (
    <EditMessageProvider messageId={messageId}>
      <Composer.Frame>
        <Composer.Input />
        <Composer.Footer>
          <Composer.Formatting />
          <Composer.Emojis />
          <Composer.CancelEdit />
          <Composer.SaveEdit />
        </Composer.Footer>
      </Composer.Frame>
    </EditMessageProvider>
  )
}

function ForwardMessageComposer({ messageId }: { messageId: string }) {
  return (
    <ForwardMessageProvider messageId={messageId}>
      <Composer.Frame>
        <Composer.Input placeholder="Add a message, if you'd like." />
        <Composer.Footer>
          <Composer.Formatting />
          <Composer.Emojis />
          <Composer.Mentions />
        </Composer.Footer>
      </Composer.Frame>
    </ForwardMessageProvider>
  )
}
```

各バリアントは以下が明示的:

- 使用する Provider / state
- 含まれる UI 要素
- 利用可能なアクション

boolean props の組み合わせを考える必要がない。不可能な状態も存在しない。

---

## 2. Render Props より Children による Composition を優先する

**影響度: MEDIUM（よりクリーンな Composition、優れた可読性）**

`renderX` props ではなく `children` で Composition する。children の方が読みやすく、自然に組み合わせられ、コールバックのシグネチャを理解する必要がない。

### 悪い例: render props

```tsx
function Composer({
  renderHeader,
  renderFooter,
  renderActions,
}: {
  renderHeader?: () => React.ReactNode
  renderFooter?: () => React.ReactNode
  renderActions?: () => React.ReactNode
}) {
  return (
    <form>
      {renderHeader?.()}
      <Input />
      {renderFooter ? renderFooter() : <DefaultFooter />}
      {renderActions?.()}
    </form>
  )
}

// 使い方が不自然で柔軟性に欠ける
return (
  <Composer
    renderHeader={() => <CustomHeader />}
    renderFooter={() => (
      <>
        <Formatting />
        <Emojis />
      </>
    )}
    renderActions={() => <SubmitButton />}
  />
)
```

### 良い例: children を使った Compound Components

```tsx
function ComposerFrame({ children }: { children: React.ReactNode }) {
  return <form>{children}</form>
}

function ComposerFooter({ children }: { children: React.ReactNode }) {
  return <footer className='flex'>{children}</footer>
}

// 柔軟な使い方
return (
  <Composer.Frame>
    <CustomHeader />
    <Composer.Input />
    <Composer.Footer>
      <Composer.Formatting />
      <Composer.Emojis />
      <SubmitButton />
    </Composer.Footer>
  </Composer.Frame>
)
```

### render props が適切な場合

```tsx
// 親からデータを渡す必要がある場合は render props が有効
<List
  data={items}
  renderItem={({ item, index }) => <Item item={item} index={index} />}
/>
```

親から子にデータや state を渡す必要がある場合は render props を使う。静的な構造の組み立てには children を使う。
