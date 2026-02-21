---
title: UIからstate管理を分離する
impact: MEDIUM
impactDescription: UIを変更せずにstate実装の交換を可能にする
tags: composition, state, architecture
---

## UIからstate管理を分離する

プロバイダーコンポーネントだけがstateの管理方法を知っている唯一の場所であるべきだ。UIコンポーネントはコンテキストインターフェースを利用し、stateがuseStateから来るのか、Zustandから来るのか、サーバー同期から来るのかを知らない。

**誤り（UIがstate実装に密結合）：**

```tsx
function ChannelComposer({ channelId }: { channelId: string }) {
  // UI component knows about global state implementation
  const state = useGlobalChannelState(channelId)
  const { submit, updateInput } = useChannelSync(channelId)

  return (
    <Composer.Frame>
      <Composer.Input
        value={state.input}
        onChange={(text) => updateInput(text)}
      />
      <Composer.Submit onPress={() => submit()} />
    </Composer.Frame>
  )
}
```

**正しい（state管理をプロバイダーに分離）：**

```tsx
// Provider handles all state management details
function ChannelProvider({
  channelId,
  children,
}: {
  channelId: string
  children: React.ReactNode
}) {
  const { state, update, submit } = useGlobalChannel(channelId)
  const inputRef = useRef(null)

  return (
    <Composer.Provider
      state={state}
      actions={{ update, submit }}
      meta={{ inputRef }}
    >
      {children}
    </Composer.Provider>
  )
}

// UI component only knows about the context interface
function ChannelComposer() {
  return (
    <Composer.Frame>
      <Composer.Header />
      <Composer.Input />
      <Composer.Footer>
        <Composer.Submit />
      </Composer.Footer>
    </Composer.Frame>
  )
}

// Usage
function Channel({ channelId }: { channelId: string }) {
  return (
    <ChannelProvider channelId={channelId}>
      <ChannelComposer />
    </ChannelProvider>
  )
}
```

**異なるプロバイダー、同じUI：**

```tsx
// Local state for ephemeral forms
function ForwardMessageProvider({ children }) {
  const [state, setState] = useState(initialState)
  const forwardMessage = useForwardMessage()

  return (
    <Composer.Provider
      state={state}
      actions={{ update: setState, submit: forwardMessage }}
    >
      {children}
    </Composer.Provider>
  )
}

// Global synced state for channels
function ChannelProvider({ channelId, children }) {
  const { state, update, submit } = useGlobalChannel(channelId)

  return (
    <Composer.Provider state={state} actions={{ update, submit }}>
      {children}
    </Composer.Provider>
  )
}
```

同じ`Composer.Input`コンポーネントが両方のプロバイダーで動作する。実装ではなくコンテキストインターフェースのみに依存しているからだ。
