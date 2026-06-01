---
title: Dashboard (Home)
tags: [domain/features, status/mock]
status: mock
sources: ["app/(dashboard)/page.tsx", "components/StatCard.tsx", "components/RevenueChart.tsx", "components/UpcomingClasses.tsx"]
updated: 2026-06-01
---

> **Status:** 🟡 Mock data — all values are hardcoded inline; no Firestore reads

# Dashboard (Home)

**Route:** `/`  
**File:** `app/(dashboard)/page.tsx`  
Server component. The first page an admin sees after logging in.

## Layout

```
Header: "Dashboard" / "Welcome back, Admin"
│
├── StatCards (3, full-width row)
│   ├── Total Members
│   ├── Monthly Revenue
│   └── Active Classes
│
└── Two-column grid
    ├── RevenueChart (2/3 width) — "Weekly Attendance"
    └── UpcomingClasses (1/3 width)
```

## StatCards

Three `StatCard` instances with **hardcoded string values** directly in `page.tsx` (not from mock data files):

| Card | Value | Trend | Color |
|------|-------|-------|-------|
| Total Members | "1,240" | +12%, up | green |
| Monthly Revenue | "$45,231" | +8%, up | blue |
| Active Classes | "24" | -2%, down | purple |

See [[ui/Component Library#StatCard]] for the component API.

## RevenueChart (Weekly Attendance)

**File:** `components/RevenueChart.tsx`  
**Note:** Despite the filename, the UI heading reads **"Weekly Attendance"**. See [[reference/Conventions & Gotchas]].

- Recharts `AreaChart` with a gradient fill (`#ccff00` at opacity 0.3 → transparent).
- Hardcoded `data` array: Mon–Sun with `attendance` values.
- A non-functional This Week/Last Week `<select>` dropdown.
- No props; data is entirely internal to the component.

## UpcomingClasses

**File:** `components/UpcomingClasses.tsx`  
**Note:** Uses its own **inline `classes` array** (not `lib/data/mock_data.ts`). 4 entries with Unsplash avatar URLs.

- CrossFit WOD (Kratos, 6:00 AM, 45 min)
- Morning Yoga (Elena, 8:00 AM, 60 min)
- HIIT Blast (Marcus, 12:00 PM, 30 min)
- Power Lifting (Sarah, 5:00 PM, 75 min)

"View All" button is styled but does nothing.

## Planned Real Data Sources

When Firestore is wired up, the expected data sources would be:
- StatCards → aggregate queries on `members`, `classes` collections
- RevenueChart → `Firestore /gyms/{gymId}/revenue` time-series
- UpcomingClasses → `Firestore /gyms/{gymId}/classes` filtered by time

## Related pages

- [[ui/Component Library]] — StatCard, RevenueChart, UpcomingClasses components
- [[data/Mock Data Layer]] — where mock data lives
- [[overview/Domain Model]] — Member, Class entities
