# 状態管理パターン

**影響度: MEDIUM**

合成されたコンポーネント間で state をリフトアップし、共有 Context を管理するためのパターン。

---

## 1. 状態管理を UI から分離する

**影響度: MEDIUM（UI を変更せずに state の実装を差し替え可能にする）**

Provider コンポーネントだけが状態管理の方法を知っているべきである。UI コンポーネントは Context のインターフェースを消費するだけで、state が useState から来るのか、Zustand から来るのか、サーバー同期から来るのかを知らない。

### 悪い例: UI が state の実装に結合

```tsx
function ChannelComposer({ channelId }: { channelId: string }) {
  // UI コンポーネントがグローバル state の実装を知っている
  const state = useGlobalChannelState(channelId)
  const { submit, updateInput } = useChannelSync(channelId)

  return (
    <Composer.Frame>
      <Composer.Input
        value={state.input}
        onChange={(text) => sync.updateInput(text)}
      />
      <Composer.Submit onPress={() => sync.submit()} />
    </Composer.Frame>
  )
}
```

### 良い例: 状態管理を Provider に隔離

```tsx
// Provider がすべての状態管理の詳細を処理
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

// UI コンポーネントは Context のインターフェースだけを知っている
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

// 使用例
function Channel({ channelId }: { channelId: string }) {
  return (
    <ChannelProvider channelId={channelId}>
      <ChannelComposer />
    </ChannelProvider>
  )
}
```

### 異なる Provider、同じ UI

```tsx
// 一時的なフォーム用のローカル state
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

// チャンネル用のグローバル同期 state
function ChannelProvider({ channelId, children }) {
  const { state, update, submit } = useGlobalChannel(channelId)

  return (
    <Composer.Provider state={state} actions={{ update, submit }}>
      {children}
    </Composer.Provider>
  )
}
```

同じ `Composer.Input` コンポーネントがどちらの Provider でも動作する。それは実装ではなく、Context のインターフェースにのみ依存しているからである。

---

## 2. 依存性注入のための汎用 Context インターフェースを定義する

**影響度: HIGH（ユースケースをまたいだ依存性注入可能な state を実現）**

コンポーネントの Context に `state`、`actions`、`meta` の3つのパートからなる**汎用インターフェース**を定義する。このインターフェースはどの Provider でも実装できる契約であり、同じ UI コンポーネントをまったく異なる state 実装で動作させることができる。

**基本原則:** state をリフトアップし、内部構造を合成し、state を依存性注入可能にする。

### 悪い例: UI が特定の state 実装に結合

```tsx
function ComposerInput() {
  // 特定のフックに密結合
  const { input, setInput } = useChannelComposerState()
  return <TextInput value={input} onChangeText={setInput} />
}
```

### 良い例: 汎用インターフェースで依存性注入を実現

```tsx
// どの Provider でも実装できる汎用インターフェースを定義
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

### UI コンポーネントは実装ではなくインターフェースを消費する

```tsx
function ComposerInput() {
  const {
    state,
    actions: { update },
    meta,
  } = use(ComposerContext)

  // このコンポーネントはインターフェースを実装するどの Provider でも動作する
  return (
    <TextInput
      ref={meta.inputRef}
      value={state.input}
      onChangeText={(text) => update((s) => ({ ...s, input: text }))}
    />
  )
}
```

### 異なる Provider が同じインターフェースを実装する

```tsx
// Provider A: 一時的なフォーム用のローカル state
function ForwardMessageProvider({ children }: { children: React.ReactNode }) {
  const [state, setState] = useState(initialState)
  const inputRef = useRef(null)
  const submit = useForwardMessage()

  return (
    <ComposerContext
      value={{
        state,
        actions: { update: setState, submit },
        meta: { inputRef },
      }}
    >
      {children}
    </ComposerContext>
  )
}

// Provider B: チャンネル用のグローバル同期 state
function ChannelProvider({ channelId, children }: Props) {
  const { state, update, submit } = useGlobalChannel(channelId)
  const inputRef = useRef(null)

  return (
    <ComposerContext
      value={{
        state,
        actions: { update, submit },
        meta: { inputRef },
      }}
    >
      {children}
    </ComposerContext>
  )
}
```

---

## 3. State を Provider コンポーネントにリフトアップする

**影響度: HIGH（コンポーネント境界を超えた state 共有を実現）**

状態管理を専用の Provider コンポーネントに移す。これにより、メイン UI の外にある兄弟コンポーネントが、props のバケツリレーや不自然な ref を使わずに state にアクセス・変更できるようになる。

### 悪い例: state がコンポーネント内に閉じ込められている

```tsx
function ForwardMessageComposer() {
  const [state, setState] = useState(initialState)
  const forwardMessage = useForwardMessage()

  return (
    <Composer.Frame>
      <Composer.Input />
      <Composer.Footer />
    </Composer.Frame>
  )
}

// 問題: このボタンはどうやって Composer の state にアクセスする？
function ForwardMessageDialog() {
  return (
    <Dialog>
      <ForwardMessageComposer />
      <MessagePreview /> {/* Composer の state が必要 */}
      <DialogActions>
        <CancelButton />
        <ForwardButton /> {/* submit を呼ぶ必要がある */}
      </DialogActions>
    </Dialog>
  )
}
```

### よくあるアンチパターン

**useEffect で state を上に同期:**

```tsx
function ForwardMessageComposer({ onInputChange }) {
  const [state, setState] = useState(initialState)
  useEffect(() => {
    onInputChange(state.input) // 変更のたびに同期
  }, [state.input])
}
```

**submit 時に ref から state を読み取る:**

```tsx
function ForwardMessageDialog() {
  const stateRef = useRef(null)
  return (
    <Dialog>
      <ForwardMessageComposer stateRef={stateRef} />
      <ForwardButton onPress={() => submit(stateRef.current)} />
    </Dialog>
  )
}
```

### 良い例: state を Provider にリフトアップ

```tsx
function ForwardMessageProvider({ children }: { children: React.ReactNode }) {
  const [state, setState] = useState(initialState)
  const forwardMessage = useForwardMessage()
  const inputRef = useRef(null)

  return (
    <Composer.Provider
      state={state}
      actions={{ update: setState, submit: forwardMessage }}
      meta={{ inputRef }}
    >
      {children}
    </Composer.Provider>
  )
}

function ForwardMessageDialog() {
  return (
    <ForwardMessageProvider>
      <Dialog>
        <ForwardMessageComposer />
        <MessagePreview />
        <DialogActions>
          <CancelButton />
          <ForwardButton />
        </DialogActions>
      </Dialog>
    </ForwardMessageProvider>
  )
}

function ForwardButton() {
  const { actions } = use(Composer.Context)
  return <Button onPress={actions.submit}>Forward</Button>
}
```

**重要なポイント:** 共有 state が必要なコンポーネントは、視覚的に入れ子になっている必要はない。同じ Provider の中にいればよい。
