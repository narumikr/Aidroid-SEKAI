# React 19 API 変更

**影響度: MEDIUM**

> **React 19+ 専用。** React 18 以前を使用している場合はこのドキュメントをスキップすること。

React 19 では `ref` が通常の props になり（`forwardRef` ラッパーが不要に）、`use()` が `useContext()` を置き換える。

---

## ref を通常の props として扱う

### 悪い例: React 19 で forwardRef を使用

```tsx
const ComposerInput = forwardRef<TextInput, Props>((props, ref) => {
  return <TextInput ref={ref} {...props} />
})
```

### 良い例: ref を通常の props として扱う

```tsx
function ComposerInput({ ref, ...props }: Props & { ref?: React.Ref<TextInput> }) {
  return <TextInput ref={ref} {...props} />
}
```

---

## useContext の代わりに use を使用する

### 悪い例: React 19 で useContext を使用

```tsx
const value = useContext(MyContext)
```

### 良い例: useContext の代わりに use を使用

```tsx
const value = use(MyContext)
```

`use()` は `useContext()` と異なり、条件分岐内でも呼び出せる。

---

## Context Provider の新しい構文

React 19 では `<Context.Provider>` ではなく `<Context>` を直接使える。

### 悪い例

```tsx
<ComposerContext.Provider value={{ state, actions, meta }}>
  {children}
</ComposerContext.Provider>
```

### 良い例

```tsx
<ComposerContext value={{ state, actions, meta }}>
  {children}
</ComposerContext>
```

---

## 参考リンク

- [React 19 公式ドキュメント](https://react.dev/blog/2024/12/05/react-19)
- [use() API リファレンス](https://react.dev/reference/react/use)
