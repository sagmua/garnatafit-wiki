---
title: Architecture
tags: [domain/overview, status/implemented]
status: implemented
sources: ["app/", "proxy.ts", "components/DashboardLayout.tsx", "app/(dashboard)/layout.tsx"]
updated: 2026-06-01
---

> **Status:** ✅ Implemented — App Router in use

# Architecture

The admin webapp is a **Next.js 16 App Router** application with TypeScript. All routing is file-based under `app/`.

## Route Groups

Two route groups handle layout separation:

```
app/
├── layout.tsx               ← root: html/body, Inter font, global CSS
├── (dashboard)/
│   ├── layout.tsx           ← wraps children in DashboardLayout (sidebar + header)
│   ├── page.tsx             ← /
│   ├── users/page.tsx       ← /users
│   ├── classes/page.tsx     ← /classes
│   ├── messages/page.tsx    ← /messages
│   ├── analytics/page.tsx   ← /analytics
│   └── settings/page.tsx    ← /settings
├── (auth)/                  ← no layout.tsx; inherits root layout only
│   ├── login/page.tsx       ← /login
│   ├── forgot-password/page.tsx
│   ├── invite/complete/page.tsx  ← /invite/complete
│   └── welcome/page.tsx     ← /welcome
└── api/auth/
    ├── login/route.ts
    ├── logout/route.ts
    ├── invite/route.ts
    ├── me/route.ts
    ├── setup-profile/route.ts
    └── complete-profile/route.ts
```

**Key point:** Auth pages (`(auth)/`) render inside the root layout only — no sidebar/header. Dashboard pages (`(dashboard)/`) render inside `DashboardLayout`, which provides the `Sidebar`, `Header`, and an `AdminProvider` context wrapping all dashboard content.

## Request Lifecycle

```
Browser request
  → proxy.ts (runs on every non-static, non-API request)
      ↓ reads session + profile_complete cookies
      ↓ verifies session with Firebase Admin SDK
      ↓ redirects to /login, /welcome, or passes through
  → Next.js routing
  → Layout(s) → Page component
  → API routes (server-only, bypass proxy.ts matcher)
```

See [[auth/Route Protection]] for the full proxy.ts decision logic.

## Component Layers

```
DashboardLayout (client)
├── AdminProvider (context — fetches /api/auth/me on mount)
├── Sidebar (client — nav, collapse state)
├── Header (client — uses useAdmin() for avatar/name)
└── <main>
    └── Page content (varies by route)
```

`DashboardLayout` is the composition root for every dashboard page. It is referenced from `app/(dashboard)/layout.tsx`.

## Server vs Client Components

- **Server components** (default, no `"use client"`): most page files, `StatCard`, `UpcomingClasses`, `DropdownMenu`, layout files.
- **Client components** (`"use client"`): anything with state, hooks, or browser APIs — `Sidebar`, `Header`, `DashboardLayout`, `RevenueChart`, all calendar components, `ClassGrid`, `AdminProvider`, and all auth/form pages.

## API Routes

All API routes live under `app/api/auth/`. They are **server-only** and **excluded from the proxy.ts matcher** (the matcher regex `/((?!_next/static|_next/image|favicon.ico|api/).*)`). They use the Firebase Admin SDK and write to Firestore server-side. See [[auth/Auth API Routes]].

## Related pages

- [[auth/Route Protection]] — proxy.ts logic in detail
- [[ui/Layouts & Navigation]] — DashboardLayout, Sidebar, Header
- [[overview/Tech Stack]] — framework versions
