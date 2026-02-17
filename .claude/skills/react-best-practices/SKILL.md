---
name: react-best-practices
description:
  React and Next.js performance optimization guidelines authored by Vercel Engineering. This skill should be used when the user asks to "optimize React performance", "reduce bundle size", "fix re-renders", "improve Next.js performance", "review React code", "refactor components", "optimize data fetching", "eliminate waterfalls", "improve SSR performance", or when writing, reviewing, or refactoring React/Next.js code involving components, pages, data fetching, bundle optimization, or rendering performance.
---

# Vercel React Best Practices

Comprehensive performance optimization guide for React and Next.js applications, authored by Vercel Engineering. Contains 57 rules across 8 categories, prioritized by impact to guide refactoring and code generation.

## When to Apply

Apply these guidelines when:
- Writing new React components or Next.js pages
- Implementing data fetching (client or server-side)
- Reviewing or refactoring React/Next.js code for performance
- Optimizing bundle size, load times, or rendering
- Debugging re-render issues or waterfall requests

## Rule Categories

Rules are grouped by domain. Each rule has its own impact level from the original source. Always prioritize CRITICAL and HIGH impact rules first.

## Quick Reference

### 1. Eliminating Waterfalls

| Rule | Impact | Description |
|------|--------|-------------|
| `async-defer-await` | HIGH | Move await into branches where actually used |
| `async-parallel` | CRITICAL | Use Promise.all() for independent operations |
| `async-dependencies` | CRITICAL | Use better-all for partial dependencies |
| `async-api-routes` | CRITICAL | Start promises early, await late in API routes |
| `async-suspense-boundaries` | HIGH | Use Suspense to stream content |

### 2. Bundle Size Optimization

| Rule | Impact | Description |
|------|--------|-------------|
| `bundle-barrel-imports` | CRITICAL | Import directly, avoid barrel files |
| `bundle-dynamic-imports` | CRITICAL | Use next/dynamic for heavy components |
| `bundle-defer-third-party` | MEDIUM | Load analytics/logging after hydration |
| `bundle-conditional` | HIGH | Load modules only when feature is activated |
| `bundle-preload` | MEDIUM | Preload on hover/focus for perceived speed |

### 3. Server-Side Performance

| Rule | Impact | Description |
|------|--------|-------------|
| `server-auth-actions` | CRITICAL | Authenticate server actions like API routes |
| `server-parallel-fetching` | CRITICAL | Restructure components to parallelize fetches |
| `server-cache-lru` | HIGH | Use LRU cache for cross-request caching |
| `server-serialization` | HIGH | Minimize data passed to client components |
| `server-cache-react` | MEDIUM | Use React.cache() for per-request deduplication |
| `server-after-nonblocking` | MEDIUM | Use after() for non-blocking operations |
| `server-dedup-props` | LOW | Avoid duplicate serialization in RSC props |

### 4. Client-Side Data Fetching

| Rule | Impact | Description |
|------|--------|-------------|
| `client-swr-dedup` | MEDIUM-HIGH | Use SWR for automatic request deduplication |
| `client-passive-event-listeners` | MEDIUM | Use passive listeners for scroll |
| `client-localstorage-schema` | MEDIUM | Version and minimize localStorage data |
| `client-event-listeners` | LOW | Deduplicate global event listeners |

### 5. Re-render Optimization

| Rule | Impact | Description |
|------|--------|-------------|
| `rerender-defer-reads` | MEDIUM | Avoid subscribing to state only used in callbacks |
| `rerender-memo` | MEDIUM | Extract expensive work into memoized components |
| `rerender-memo-with-default-value` | MEDIUM | Hoist default non-primitive props |
| `rerender-derived-state` | MEDIUM | Subscribe to derived booleans, not raw values |
| `rerender-derived-state-no-effect` | MEDIUM | Derive state during render, not effects |
| `rerender-functional-setstate` | MEDIUM | Use functional setState for stable callbacks |
| `rerender-lazy-state-init` | MEDIUM | Pass function to useState for expensive values |
| `rerender-move-effect-to-event` | MEDIUM | Put interaction logic in event handlers |
| `rerender-transitions` | MEDIUM | Use startTransition for non-urgent updates |
| `rerender-use-ref-transient-values` | MEDIUM | Use refs for transient frequent values |
| `rerender-simple-expression-in-memo` | LOW-MEDIUM | Avoid memo for simple primitives |
| `rerender-dependencies` | LOW | Use primitive dependencies in effects |

### 6. Rendering Performance

| Rule | Impact | Description |
|------|--------|-------------|
| `rendering-content-visibility` | HIGH | Use content-visibility for long lists |
| `rendering-activity` | MEDIUM | Use Activity component for show/hide |
| `rendering-hydration-no-flicker` | MEDIUM | Use inline script for client-only data |
| `rendering-hydration-suppress-warning` | LOW-MEDIUM | Suppress expected mismatches |
| `rendering-animate-svg-wrapper` | LOW | Animate div wrapper, not SVG element |
| `rendering-hoist-jsx` | LOW | Extract static JSX outside components |
| `rendering-svg-precision` | LOW | Reduce SVG coordinate precision |
| `rendering-conditional-render` | LOW | Use ternary when condition can be falsy non-boolean |
| `rendering-usetransition-loading` | LOW | Prefer useTransition for loading state |

### 7. JavaScript Performance

| Rule | Impact | Description |
|------|--------|-------------|
| `js-length-check-first` | MEDIUM-HIGH | Check array length before expensive comparison |
| `js-tosorted-immutable` | MEDIUM-HIGH | Use toSorted() to prevent mutation bugs in React state |
| `js-batch-dom-css` | MEDIUM | Avoid layout thrashing by batching style writes |
| `js-cache-function-results` | MEDIUM | Cache function results in module-level Map |
| `js-index-maps` | LOW-MEDIUM | Build Map for repeated lookups |
| `js-cache-property-access` | LOW-MEDIUM | Cache object properties in loops |
| `js-cache-storage` | LOW-MEDIUM | Cache localStorage/sessionStorage reads |
| `js-combine-iterations` | LOW-MEDIUM | Combine multiple filter/map into one loop |
| `js-early-exit` | LOW-MEDIUM | Return early from functions |
| `js-hoist-regexp` | LOW-MEDIUM | Hoist RegExp creation outside loops |
| `js-set-map-lookups` | LOW-MEDIUM | Use Set/Map for O(1) lookups |
| `js-min-max-loop` | LOW | Use loop for min/max instead of sort |

### 8. Advanced Patterns

| Rule | Impact | Description |
|------|--------|-------------|
| `advanced-init-once` | LOW-MEDIUM | Initialize app once per app load |
| `advanced-event-handler-refs` | LOW | Store event handlers in refs |
| `advanced-use-latest` | LOW | useEffectEvent for stable callback refs |

## How to Use

Read individual rule files in `references/` for detailed explanations and code examples. Each rule file contains:
- Brief explanation of why the pattern matters
- Incorrect code example with explanation
- Correct code example with explanation
- Additional context and trade-offs

To look up a specific rule:

```
references/<prefix>-<rule-name>.md
```

For example: `references/async-parallel.md`, `references/bundle-barrel-imports.md`

When reviewing code, start with CRITICAL rules (async-*, bundle-*) and work down the priority list.

## Examples

Complete before/after scenarios combining multiple rules in real-world contexts:

- **`examples/nextjs-dashboard-page.md`** - Dashboard page: waterfall elimination + parallel fetching + bundle optimization + SSR
- **`examples/rerender-optimization-form.md`** - Form component: derived state + functional setState + lazy init + event handlers
- **`examples/bundle-optimization-landing.md`** - Landing page: dynamic imports + deferred third-party + conditional loading + preload
