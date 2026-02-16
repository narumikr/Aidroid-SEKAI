# Provider パターン 実装例

同じ UI コンポーネントを異なる Provider で使い回す完全な例。

---

## ローカル State の Provider

一時的なフォーム（転送ダイアログなど）向け。

```tsx
function ForwardMessageProvider({ children }: { children: React.ReactNode }) {
  const [state, setState] = useState<ComposerState>({
    input: '',
    attachments: [],
    isSubmitting: false,
  })
  const inputRef = useRef<TextInput>(null)
  const forwardMessage = useForwardMessage()

  const submit = async () => {
    setState((s) => ({ ...s, isSubmitting: true }))
    await forwardMessage(state)
    setState((s) => ({ ...s, isSubmitting: false }))
  }

  return (
    <Composer.Provider
      state={state}
      actions={{ update: setState, submit }}
      meta={{ inputRef }}
    >
      {children}
    </Composer.Provider>
  )
}
```

## グローバル同期 State の Provider

チャンネルメッセージなど、サーバーと同期する state 向け。

```tsx
function ChannelProvider({
  channelId,
  children,
}: {
  channelId: string
  children: React.ReactNode
}) {
  const { state, update, submit } = useGlobalChannel(channelId)
  const inputRef = useRef<TextInput>(null)

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
```

## Provider 外の兄弟コンポーネントからの state アクセス

Provider の境界内であれば、Composer.Frame の外からでも state と actions にアクセスできる。

```tsx
function ForwardMessageDialog() {
  return (
    <ForwardMessageProvider>
      <Dialog>
        {/* Composer の UI */}
        <Composer.Frame>
          <Composer.Input placeholder="Add a message, if you'd like." />
          <Composer.Footer>
            <Composer.Formatting />
            <Composer.Emojis />
          </Composer.Footer>
        </Composer.Frame>

        {/* Composer の外だが、Provider の中にあるカスタム UI */}
        <MessagePreview />

        {/* ダイアログ下部のアクション */}
        <DialogActions>
          <CancelButton />
          <ForwardButton />
        </DialogActions>
      </Dialog>
    </ForwardMessageProvider>
  )
}

// Provider 内であれば、どこからでも Context にアクセス可能
function ForwardButton() {
  const { actions: { submit } } = use(Composer.Context)!
  return <Button onPress={submit}>Forward</Button>
}

function MessagePreview() {
  const { state } = use(Composer.Context)!
  return <Preview message={state.input} attachments={state.attachments} />
}
```

## 同じ UI を異なる Provider で使い回す

```tsx
// ForwardMessageProvider（ローカル state）で動作
<ForwardMessageProvider>
  <Composer.Frame>
    <Composer.Input />
    <Composer.Submit />
  </Composer.Frame>
</ForwardMessageProvider>

// ChannelProvider（グローバル同期 state）で動作
<ChannelProvider channelId="abc">
  <Composer.Frame>
    <Composer.Input />
    <Composer.Submit />
  </Composer.Frame>
</ChannelProvider>
```

重要なのは Provider の境界であり、視覚的なネストではない。UI は合成する再利用可能なパーツ。state は Provider から依存性注入される。
