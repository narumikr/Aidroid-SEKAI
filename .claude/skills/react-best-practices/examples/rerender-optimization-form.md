# Example: Re-render Optimized Form Component

A form component applying multiple re-render optimization rules together.

## Before (5 issues)

```tsx
'use client'

import { useState, useEffect, useCallback, useMemo } from 'react'

function ProductForm({ categories }: { categories: Category[] }) {
  const [formData, setFormData] = useState({
    name: '',
    price: 0,
    category: '',
    description: '',
  })
  const [windowWidth, setWindowWidth] = useState(window.innerWidth) // not lazy (rerender-lazy-state-init)
  const [isMobile, setIsMobile] = useState(false)

  // Derived state via effect (rerender-derived-state-no-effect)
  useEffect(() => {
    setIsMobile(windowWidth < 768)
  }, [windowWidth])

  // Subscribes to continuous value (rerender-derived-state)
  useEffect(() => {
    const handler = () => setWindowWidth(window.innerWidth)
    window.addEventListener('resize', handler)
    return () => window.removeEventListener('resize', handler)
  }, [])

  // Non-functional setState creates unstable callback (rerender-functional-setstate)
  const handleChange = useCallback((field: string, value: string) => {
    setFormData({ ...formData, [field]: value })
  }, [formData]) // dependency on formData causes re-creation every render

  // Effect for interaction logic (rerender-move-effect-to-event)
  useEffect(() => {
    if (formData.name.length > 100) {
      alert('Name too long!')
    }
  }, [formData.name])

  const sortedCategories = useMemo(
    () => categories.sort((a, b) => a.name.localeCompare(b.name)), // mutates original array
    [categories]
  )

  return (
    <form>
      <input
        value={formData.name}
        onChange={(e) => handleChange('name', e.target.value)}
      />
      {/* ... */}
    </form>
  )
}
```

## After (all 5 issues fixed)

```tsx
'use client'

import { useState, useCallback } from 'react'

function ProductForm({ categories }: { categories: Category[] }) {
  // Lazy initialization for expensive computation (rerender-lazy-state-init)
  const [formData, setFormData] = useState(() => ({
    name: '',
    price: 0,
    category: '',
    description: '',
  }))

  // Subscribe to derived boolean directly (rerender-derived-state)
  const isMobile = useMediaQuery('(max-width: 767px)')

  // Functional setState - no dependency on formData (rerender-functional-setstate)
  const handleChange = useCallback((field: string, value: string) => {
    setFormData(prev => ({ ...prev, [field]: value }))
  }, []) // stable: no dependencies needed

  // Validation in event handler, not effect (rerender-move-effect-to-event)
  const handleNameChange = useCallback((e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value
    if (value.length > 100) {
      alert('Name too long!')
      return
    }
    setFormData(prev => ({ ...prev, name: value }))
  }, [])

  // Immutable sort (js-tosorted-immutable) + derive during render (rerender-derived-state-no-effect)
  const sortedCategories = categories.toSorted((a, b) =>
    a.name.localeCompare(b.name)
  )

  return (
    <form>
      <input
        value={formData.name}
        onChange={handleNameChange}
      />
      {/* ... */}
    </form>
  )
}
```

## Applied Rules

| Rule | Impact | What Changed |
|------|--------|-------------|
| `rerender-derived-state` | MEDIUM | windowWidth subscription → useMediaQuery boolean |
| `rerender-derived-state-no-effect` | MEDIUM | Effect-based derived state → computed during render |
| `rerender-functional-setstate` | MEDIUM | Spread setState → functional updater (stable callback) |
| `rerender-lazy-state-init` | MEDIUM | Direct value → factory function in useState |
| `rerender-move-effect-to-event` | MEDIUM | Validation effect → event handler logic |
| `js-tosorted-immutable` | MEDIUM-HIGH | Mutating sort() → immutable toSorted() |
