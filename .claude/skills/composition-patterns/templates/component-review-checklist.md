# コンポーネント設計レビュー チェックリスト

コンポーネントの新規作成やリファクタリング時に確認する項目。

---

## コンポーネント設計 (HIGH)

- [ ] boolean props（`isX`, `showX`）が3つ以上ないか？ → Composition に切り替える
- [ ] 条件分岐が props に基づいて分岐していないか？ → 明示的なバリアントを作成
- [ ] render props（`renderX`）を使っていないか？ → children による Composition を検討
- [ ] Compound Components パターンで構造化されているか？

## 状態管理 (MEDIUM)

- [ ] UI コンポーネントが特定の state 実装（useGlobalXxx 等）に直接依存していないか？
- [ ] state、actions、meta の3パート構成で Context インターフェースが定義されているか？
- [ ] state は Provider コンポーネントにリフトアップされているか？
- [ ] Provider を差し替えても UI が動作するか？

## 実装パターン (MEDIUM)

- [ ] 各バリアントは使用する Provider / state が明示的か？
- [ ] 各バリアントに含まれる UI 要素が明確か？
- [ ] 不可能な状態の組み合わせが存在しないか？

## React 19 API

- [ ] `forwardRef` の代わりに ref を通常の props として受け取っているか？
- [ ] `useContext()` の代わりに `use()` を使用しているか？
- [ ] `<Context.Provider>` の代わりに `<Context>` を使用しているか？

---

## 判定基準

| 問題 | 対処法 |
|------|--------|
| boolean props が3つ以上 | → `references/architecture-patterns.md` の「Boolean Props の増殖を避ける」 |
| props のバケツリレー | → `references/architecture-patterns.md` の「Compound Components を使う」 |
| UI が state 実装に結合 | → `references/state-management-patterns.md` の「状態管理を UI から分離する」 |
| 兄弟コンポーネントで state 共有 | → `references/state-management-patterns.md` の「State を Provider にリフトアップ」 |
| render props の乱用 | → `references/implementation-patterns.md` の「Children による Composition を優先」 |
| forwardRef / useContext 使用 | → `references/react19-api-changes.md` |
