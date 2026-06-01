---
title: Component Library
tags: [domain/ui, status/implemented]
status: implemented
sources: ["components/"]
updated: 2026-06-01
---

> **Status:** ✅ Implemented

# Component Library

Inventory of all shared components under `components/`. Pages under `app/` may also define inline components (e.g. `Toggle` in settings, `MessageBubble` in messages).

## Shell Components

### DashboardLayout
`components/DashboardLayout.tsx` — Client component. The composition root for every dashboard page.  
**Props:** `children`, `initialCollapsed?: boolean` (default `false`)  
Wraps everything in `<AdminProvider>`. Manages `isCollapsed` and `isMobileOpen` state. Locks body scroll when mobile menu is open. Main content padding shifts: `md:pl-20` (collapsed) vs `md:pl-64` (expanded).  
**Used by:** `app/(dashboard)/layout.tsx`

### Sidebar
`components/Sidebar.tsx` — Client component. Primary navigation.  
**Props:** `isCollapsed`, `toggleSidebar`, `isMobileOpen`, `onMobileClose`  
Two variants rendered simultaneously: mobile drawer (with backdrop, backdrop click closes) and desktop fixed sidebar. Nav items: Dashboard `/`, Users `/users`, Classes `/classes`, Messages `/messages`, Analytics `/analytics`, Settings `/settings` — each with a Lucide icon. Active link detected via `usePathname()` — styled `text-gym-green` with glow shadow. `useEffect` auto-closes mobile drawer on route change.

### Header
`components/header/Header.tsx` — Client component.  
**Props:** `onMenuClick: () => void`  
Sticky top bar. Left: hamburger button (mobile only). Right: admin name (desktop only, from `useAdmin()`), notification bell with green dot, avatar `DropdownMenu`.  
Avatar fallback chain: `admin.photoURL → admin.fallbackAvatar → '/default-avatar.svg'` (with `onError` handler). Logout calls `signOut(clientAuth)` + `POST /api/auth/logout` + redirect to `/login`.

### DropdownMenu
`components/DropdownMenu.tsx` — No `"use client"` (server-compatible).  
**Props:** `children` (the `MenuButton` trigger), `options: DropdownOption[]`, `importantOptions?: DropdownOption[]`  
Built on Headless UI `Menu`/`MenuButton`/`MenuItems`. `MenuItems` uses `anchor="bottom end"`. Normal options render as `<Link href={item.path}>`. Important options render as `<button onClick>` styled `text-red-400`. A divider separates the two groups.  
⚠️ **Gotcha:** Normal options always render as `<Link>` even when only `onClick` is supplied — `path` can be `undefined`. See [[reference/Conventions & Gotchas]].

## Dashboard Widgets

### StatCard
`components/StatCard.tsx` — Server-compatible.  
**Props:** `title`, `value`, `icon: LucideIcon`, `trend?`, `trendUp?`, `trendLabel?`, `color?: "green" | "blue" | "purple"`  
Metric card with a colored icon badge. Color map: green=`text-gym-green bg-gym-green/10`, blue=`text-gym-blue bg-gym-blue/10`, purple=`text-gym-purple bg-gym-purple/10`. Hover lift `hover:-translate-y-1`. Uses `glass-card`.  
**Used by:** Dashboard home page and Analytics page.

### RevenueChart
`components/RevenueChart.tsx` — Client component.  
**Note:** UI heading reads **"Weekly Attendance"** despite the filename. See [[reference/Conventions & Gotchas]].  
No props. Hardcoded Mon–Sun `data` array. Recharts `AreaChart` with `#ccff00` gradient fill. Custom `Tooltip`. Non-functional week selector.

### UpcomingClasses
`components/UpcomingClasses.tsx` — Server-compatible.  
No props. Hardcoded 4-class inline array (Unsplash avatars). "View All" button does nothing.

## Calendar Components

### ClassCalendar
`components/calendar/ClassCalendar.tsx` — Client component.  
Toggles between `StripCalendar` (collapsed) and `MonthCalendar` (expanded) via a `ChevronDown` button.

### StripCalendar
`components/calendar/StripCalendar.tsx` — Client component.  
Fully static. Mon–Sun, dates 20–26, "December 2025" header. Wed highlighted gym-green. No props.

### MonthCalendar
`components/calendar/MonthCalendar.tsx` — Client component.  
Wraps `react-calendar` v6. Local `value` state (defaults `new Date()`). Styled by `components/styles/FullCalendar.css`.

## Class Components

### ClassGrid
`components/classes/ClassGrid.tsx` — Client component.  
Renders cards from `lib/data/mock_data.ts`. Each card has: colored accent bar (`cls.color`), status pill, instructor, time/duration, enrollment progress bar, `DropdownMenu` (Edit/Delete — no-op).  
⚠️ **Gotcha:** Declares `children?: ReactNode` prop that is never used. See [[reference/Conventions & Gotchas]].

## Related pages

- [[ui/Layouts & Navigation]] — how shell components fit together
- [[ui/Styling & Theme]] — glass-card, color palette
- [[reference/Conventions & Gotchas]] — known quirks in components
