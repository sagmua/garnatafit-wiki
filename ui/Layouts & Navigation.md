---
title: Layouts & Navigation
tags: [domain/ui, status/implemented]
status: implemented
sources: ["app/layout.tsx", "app/(dashboard)/layout.tsx", "components/DashboardLayout.tsx", "components/Sidebar.tsx", "components/header/Header.tsx"]
updated: 2026-06-01
---

> **Status:** ✅ Implemented

# Layouts & Navigation

## Root Layout

**File:** `app/layout.tsx`  
Sets `<html lang="en">`, applies Inter Google font to `<body>`, imports `globals.css`. Metadata: `title: "GarnataFit"`, `description: "Premium Gym Management System"`. Renders `{children}` only — no chrome.

## Dashboard Layout

**File:** `app/(dashboard)/layout.tsx`  
One-liner: `<DashboardLayout initialCollapsed={false}>{children}</DashboardLayout>`

### DashboardLayout Component

**File:** `components/DashboardLayout.tsx`  
The real implementation. Provides:
- `<AdminProvider>` wrapping (profile context for all dashboard pages)
- `<Sidebar>` with collapse/mobile state
- Left content padding that shifts based on `isCollapsed`: `md:pl-20` (icons-only) vs `md:pl-64` (full)
- `<Header>` with `onMenuClick` callback
- `<main>` with `p-4 md:p-6`

Body scroll is locked via `overflow-hidden` class when the mobile drawer is open.

## Auth Layout

`app/(auth)/` has **no layout file**. Auth pages inherit only the root layout — no sidebar, no header. This gives auth pages their full-screen centered design.

## Sidebar

**File:** `components/Sidebar.tsx`  
Client component. Two simultaneously rendered variants:
- **Mobile drawer** — fixed `inset-0 z-50`, dark backdrop. Slides in; backdrop click or route change closes it.
- **Desktop sidebar** — fixed left, `w-20` (collapsed) or `w-64` (expanded), hidden on mobile.

### Navigation Items

```
Dashboard   /         LayoutDashboard icon
Users       /users    Users icon
Classes     /classes  Calendar icon
Messages    /messages MessageSquare icon
Analytics   /analytics BarChart3 icon
Settings    /settings Settings icon
```

Active route detected via `usePathname()`. Active style: `text-gym-green bg-gym-green/5 shadow-[0_0_20px_rgba(204,255,0,0.1)]`. Inactive: `text-gray-400 hover:text-white hover:bg-white/5`.

### Logo Block

`Dumbbell` icon + "Garnata" + `<span class="text-gym-green">Fit</span>`. In collapsed mode (desktop), only the Dumbbell icon is shown.

### Collapse Toggle

A `ChevronLeft`/`ChevronRight` button at the bottom of the desktop sidebar toggles `isCollapsed`. Not shown on mobile.

## Header

**File:** `components/header/Header.tsx`  
Sticky (`sticky top-0 z-50`), 64px tall. Backdrop blur (`backdrop-blur-md`).

### Left side (mobile only)
- Hamburger `Menu` icon button → calls `onMenuClick` → opens mobile drawer

### Right side
- Admin name (hidden on mobile, `hidden md:block`)
- Notification bell with green dot badge
- Avatar `DropdownMenu`:
  - Trigger: `MenuButton` wrapping the admin avatar `<img>`
  - Options: Settings → `/settings`
  - Important options: Logout (red, calls `handleLogout`)

### Avatar Display

```
admin?.photoURL ?? admin?.fallbackAvatar ?? DEFAULT_AVATAR
```
`onError` sets `imgError` state → falls back to `/default-avatar.svg`. The avatar is `p-1` padded inside a rounded-full container (`h-11 w-11`).

## Related pages

- [[ui/Component Library]] — DashboardLayout, Sidebar, Header, DropdownMenu detail
- [[auth/Authentication Overview]] — AdminProvider provides data to Header
- [[overview/Architecture]] — route group structure
