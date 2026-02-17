# Example: Landing Page Bundle Optimization

A landing page applying bundle size and loading optimization rules together.

## Before (4 issues)

```tsx
// app/page.tsx
import { Hero, Features, Pricing, FAQ, Footer } from '@/components'  // barrel import
import { Analytics } from '@segment/analytics-next'                    // loaded immediately
import { motion } from 'framer-motion'                                 // full library loaded
import Confetti from 'react-confetti'                                  // loaded even if not used

export default function LandingPage() {
  const [showConfetti, setShowConfetti] = useState(false)

  return (
    <div>
      <Analytics writeKey="..." />
      {showConfetti && <Confetti />}
      <Hero />
      <Features />
      <Pricing onPurchase={() => setShowConfetti(true)} />
      <FAQ />
      <Footer />
    </div>
  )
}
```

## After (all 4 issues fixed)

```tsx
// app/page.tsx
import Hero from '@/components/hero/Hero'               // direct imports (bundle-barrel-imports)
import Features from '@/components/features/Features'
import Pricing from '@/components/pricing/Pricing'
import dynamic from 'next/dynamic'
import { Suspense } from 'react'

// Heavy components loaded on demand (bundle-dynamic-imports)
const FAQ = dynamic(() => import('@/components/faq/FAQ'))
const Footer = dynamic(() => import('@/components/footer/Footer'))

// Conditional module: only loaded when triggered (bundle-conditional)
const Confetti = dynamic(() => import('react-confetti'), { ssr: false })

export default function LandingPage() {
  const [showConfetti, setShowConfetti] = useState(false)

  return (
    <div>
      {/* Analytics deferred after hydration (bundle-defer-third-party) */}
      <DeferredAnalytics />
      {showConfetti && <Confetti />}
      <Hero />
      <Features />
      {/* Preload below-fold sections on interaction (bundle-preload) */}
      <div onMouseEnter={() => {
        import('@/components/pricing/Pricing')
      }}>
        <Pricing onPurchase={() => setShowConfetti(true)} />
      </div>
      <Suspense fallback={null}>
        <FAQ />
      </Suspense>
      <Suspense fallback={null}>
        <Footer />
      </Suspense>
    </div>
  )
}

// Defer non-critical third-party (bundle-defer-third-party)
function DeferredAnalytics() {
  useEffect(() => {
    import('@segment/analytics-next').then(({ Analytics }) => {
      const analytics = new Analytics({ writeKey: '...' })
      analytics.page()
    })
  }, [])
  return null
}
```

## Applied Rules

| Rule | Impact | What Changed |
|------|--------|-------------|
| `bundle-barrel-imports` | CRITICAL | Barrel re-export → direct file imports |
| `bundle-dynamic-imports` | CRITICAL | Static imports → next/dynamic for FAQ, Footer |
| `bundle-conditional` | HIGH | Confetti always bundled → loaded only when triggered |
| `bundle-defer-third-party` | MEDIUM | Analytics loaded immediately → deferred after hydration |
| `bundle-preload` | MEDIUM | No preloading → preload on hover interaction |
