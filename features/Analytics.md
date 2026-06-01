---
title: Analytics
tags: [domain/features, status/mock]
status: mock
sources: ["app/(dashboard)/analytics/page.tsx", "lib/data/mock_analytics.ts"]
updated: 2026-06-01
---

> **Status:** 🟡 Mock data — all values from `lib/data/mock_analytics.ts`

# Analytics

**Route:** `/analytics`  
**File:** `app/(dashboard)/analytics/page.tsx`  
Client component. Gym performance dashboard with KPI cards and four Recharts visualizations.

## Period Toggle

Four buttons at the top: **7d / 30d / 90d / 1y**. Selecting a period switches `analyticsData[period]` snapshot. All data for all four periods is pre-loaded in memory.

## KPI Cards (4 StatCards)

Each reads from the active `AnalyticsSnapshot`:

| Card | Field | Trend field | Color |
|------|-------|-------------|-------|
| Total Members | `totalMembers` | `membersTrend` | green |
| Revenue | `periodRevenue` (formatted as `$Xk`) | `revenueTrend` | blue |
| Avg Attendance | `avgAttendanceRate` (formatted as `X%`) | `attendanceTrend` | purple |
| New Sign-ups | `newSignUps` | `signUpsTrend` | green |

Trend > 0 → `trendUp: true` (green up arrow); trend ≤ 0 → red down arrow.

## Charts (4 Recharts)

### 1. Member Growth — AreaChart

- Data: `snapshot.memberGrowth: TimeSeriesPoint[]` (label, value)
- Green fill gradient (`#ccff00`, opacity 0.3 → transparent)
- Same visual style as the Dashboard's RevenueChart

### 2. Revenue — BarChart

- Data: `snapshot.revenueByPeriod: RevenuePoint[]` (label, revenue)
- Green fill (`#ccff00`), gym-gray background grid, custom `Tooltip`

### 3. Class Types — PieChart (Donut)

- Data: `snapshot.classTypeBreakdown: ClassTypeBreakdown[]` (name, value, color)
- 4 slices: HIIT (`#ccff00`), Yoga (`#00f3ff`), CrossFit (`#9d00ff`), Strength (`#f59e0b`)
- Custom legend rendered below the donut
- `innerRadius: 60, outerRadius: 100` — donut shape

### 4. Class Utilization — Custom progress bars

- Data: `snapshot.classAttendance: ClassAttendance[]` (name, attended, capacity)
- Sorted by `attended/capacity` ratio descending
- Progress bar width = `(attended / capacity) * 100%`
- Color thresholds: `>= 90%` → red-500, `>= 70%` → yellow-500, else gym-green
- Not a Recharts component — plain HTML/CSS

## Real Data Path

When Firestore is wired up, analytics would be derived from live `members` and `classes` collections, potentially via Cloud Functions that aggregate into pre-computed snapshots for performance.

## Related pages

- [[data/Mock Data Layer]] — AnalyticsSnapshot type definitions
- [[ui/Component Library]] — StatCard component
- [[overview/Domain Model]] — Analytics entity
