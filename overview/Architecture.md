---
title: Architecture
tags: [domain/overview, status/implemented]
status: implemented
sources: ["app/", "proxy.ts", "components/DashboardLayout.tsx", "components/MemberLayout.tsx", "app/(dashboard)/layout.tsx", "app/(member)/layout.tsx", "lib/auth/session.ts"]
updated: 2026-06-11
---

> **Status:** ✅ Implemented — App Router in use

# Architecture

The admin webapp is a **Next.js 16 App Router** application with TypeScript. All routing is file-based under `app/`.

## Route Groups

Three route groups handle layout separation:

```
app/
├── layout.tsx               ← root: html/body, Inter font, global CSS
├── (dashboard)/             ← admin area (role: admin)
│   ├── layout.tsx           ← wraps children in DashboardLayout (sidebar + header)
│   ├── page.tsx             ← /
│   ├── users/page.tsx       ← /users
│   ├── classes/page.tsx     ← /classes
│   ├── plans/page.tsx       ← /plans
│   ├── messages/page.tsx    ← /messages
│   ├── analytics/page.tsx   ← /analytics
│   └── settings/page.tsx    ← /settings
├── (member)/                ← member area (role: member)
│   ├── layout.tsx           ← wraps children in MemberLayout (member sidebar + header)
│   └── member/
│       ├── page.tsx         ← /member (home; shows real credits)
│       ├── classes/page.tsx ← /member/classes
│       ├── reservations/page.tsx ← /member/reservations
│       ├── messages/page.tsx     ← /member/messages
│       └── profile/page.tsx      ← /member/profile (real self-edit)
├── (auth)/                  ← no layout.tsx; inherits root layout only
│   ├── login/page.tsx       ← /login
│   ├── forgot-password/page.tsx
│   ├── invite/complete/page.tsx  ← /invite/complete
│   └── welcome/page.tsx     ← /welcome
└── api/
    ├── auth/   login, logout, invite, me, setup-profile, complete-profile
    ├── plans/  route.ts (GET/POST) + [id]/route.ts (PATCH/DELETE)
    ├── members/ route.ts (GET) + [uid]/credits/route.ts (POST)
    └── member/ profile/route.ts (PATCH) + credits/route.ts (GET)
```

**Key point:** Auth pages (`(auth)/`) render inside the root layout only — no chrome. Admin pages (`(dashboard)/`) render inside `DashboardLayout`; member pages (`(member)/`) render inside `MemberLayout`. Both layouts provide the shared `Sidebar` (member layout passes a member nav via the `items` prop), `Header`, and `AdminProvider`. The two areas are kept apart by the `role` claim in `proxy.ts` — see [[auth/Roles & Claims]] and [[ui/Layouts & Navigation]].

## Request Lifecycle

```
Browser request
  → proxy.ts (runs on every non-static, non-API request)
      ↓ reads session + profile_complete cookies
      ↓ verifies session with Firebase Admin SDK (reads the `role` claim)
      ↓ redirects to /login, /welcome, /member, /, or passes through
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

API routes live under `app/api/` — `auth/` (session/onboarding), `plans/` and `members/` (admin-only, via `requireAdmin`), and `member/` (member-scoped). They are **server-only** and **excluded from the proxy.ts matcher** (the matcher regex `/((?!_next/static|_next/image|favicon.ico|api/).*)`). They use the Firebase Admin SDK and write to Firestore server-side — the browser never touches Firestore directly. A shared guard `lib/auth/session.ts` (`getSessionUser`, `requireAdmin`) centralizes cookie parsing and role checks. See [[auth/Auth API Routes]], [[auth/Roles & Claims]], and [[features/Plans & Credits]].

## Related pages

- [[auth/Route Protection]] — proxy.ts logic in detail
- [[ui/Layouts & Navigation]] — DashboardLayout, Sidebar, Header
- [[overview/Tech Stack]] — framework versions
