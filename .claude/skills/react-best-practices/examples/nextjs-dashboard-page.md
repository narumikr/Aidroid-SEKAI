# Example: Next.js Dashboard Page Optimization

A real-world dashboard page applying multiple optimization rules together.

## Before (6 issues)

```tsx
// app/dashboard/page.tsx
import { Chart } from '@/components/charts'           // barrel import (bundle-barrel-imports)
import { formatDate, formatCurrency } from '@/lib/utils' // barrel import

export default async function DashboardPage() {
  const user = await getCurrentUser()                   // waterfall: sequential fetches
  const stats = await fetchStats(user.id)               // (async-parallel)
  const activities = await fetchActivities(user.id)     //

  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <StatsPanel stats={stats} user={user} activities={activities} />
      <Chart data={stats.chartData} />
      <ActivityFeed activities={activities} user={user} stats={stats} />
    </div>
  )
}

// Passes full objects as props causing duplicate serialization (server-dedup-props)
function StatsPanel({ stats, user, activities }: {
  stats: Stats; user: User; activities: Activity[]
}) {
  return (
    <div>
      <p>{user.name}'s Stats</p>
      <p>Total: {stats.total}</p>
      <p>Activities: {activities.length}</p>
    </div>
  )
}
```

## After (all 6 issues fixed)

```tsx
// app/dashboard/page.tsx
import { formatDate } from '@/lib/utils/format-date'     // direct imports (bundle-barrel-imports)
import { formatCurrency } from '@/lib/utils/format-currency'
import dynamic from 'next/dynamic'
import { Suspense } from 'react'
import { cache } from 'react'

// Dynamic import for heavy chart component (bundle-dynamic-imports)
const Chart = dynamic(() => import('@/components/charts/Chart'))

// Per-request deduplication (server-cache-react)
const getCurrentUser = cache(async () => {
  const session = await auth()
  if (!session?.user?.id) return null
  return await db.user.findUnique({ where: { id: session.user.id } })
})

export default async function DashboardPage() {
  const user = await getCurrentUser()
  if (!user) redirect('/login')

  // Parallel fetching (async-parallel)
  const [stats, activities] = await Promise.all([
    fetchStats(user.id),
    fetchActivities(user.id),
  ])

  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      {/* Pass only needed primitives (server-dedup-props, server-serialization) */}
      <StatsPanel
        userName={user.name}
        total={stats.total}
        activityCount={activities.length}
      />
      <Suspense fallback={<ChartSkeleton />}>
        <Chart data={stats.chartData} />
      </Suspense>
      <Suspense fallback={<FeedSkeleton />}>
        <ActivityFeed userId={user.id} />
      </Suspense>
    </div>
  )
}

// Receives only primitives - no duplicate serialization
function StatsPanel({ userName, total, activityCount }: {
  userName: string; total: number; activityCount: number
}) {
  return (
    <div>
      <p>{userName}'s Stats</p>
      <p>Total: {total}</p>
      <p>Activities: {activityCount}</p>
    </div>
  )
}
```

## Applied Rules

| Rule | Impact | What Changed |
|------|--------|-------------|
| `async-parallel` | CRITICAL | Sequential awaits → Promise.all() |
| `bundle-barrel-imports` | CRITICAL | Barrel imports → direct file imports |
| `bundle-dynamic-imports` | CRITICAL | Static Chart import → next/dynamic |
| `server-serialization` | HIGH | Minimized data passed to client |
| `server-cache-react` | MEDIUM | getCurrentUser wrapped in React.cache() |
| `server-dedup-props` | LOW | Full objects → primitive props only |
