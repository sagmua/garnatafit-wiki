# Wiki Index

Complete catalog of all pages, organized by category. One-line summary per page.

---

## Overview

| Page | Summary |
|------|---------|
| [[overview/Project Overview]] | GarnataFit ecosystem: admin webapp + React Native mobile app, roles, scope |
| [[overview/Architecture]] | Next.js 16 App Router structure, route groups, request lifecycle, data flow |
| [[overview/Tech Stack]] | All frameworks, libraries, and tools with versions and rationale |
| [[overview/Domain Model]] | Core entities (Gym, Admin, Member, Class, Thread, Analytics) and their relationships |

## Auth System ✅

| Page | Summary |
|------|---------|
| [[auth/Authentication Overview]] | Session-cookie model, Firebase, the two-cookie gate, maturity summary |
| [[auth/Roles & Claims]] | `role` custom claim (admin/member), profile collections, `lib/auth/session.ts` guard, legacy back-compat |
| [[auth/Invite & Join Flow]] | 4-phase onboarding (admin or member): invite+role → password → claim mint → onboarding |
| [[auth/Login & Logout]] | Login API, session cookie creation, logout cleanup |
| [[auth/Profile & Onboarding]] | /welcome page, complete-profile API, avatar fallback chain, AdminContext |
| [[auth/Route Protection]] | proxy.ts logic: public vs gated routes, profile-complete gate, cookie verification |
| [[auth/Auth API Routes]] | Every /api/auth/* endpoint: inputs, outputs, status codes, error cases |
| [[auth/Firebase Setup]] | Admin SDK vs client SDK initialization, env vars, server-side-writes posture |

## Features

| Page | Summary |
|------|---------|
| [[features/Plans & Credits]] ✅ | Bundle templates (gym catalog) + member credit grants; Admin SDK; no deduction engine yet |
| [[features/Member Area]] 🟡 | `/member/*` route group: real profile/credits; mock classes/reservations/chat |
| [[features/Dashboard (Home)]] 🟡 | Home dashboard: 3 StatCards, RevenueChart (Weekly Attendance), UpcomingClasses |
| [[features/Users]] ✅ | Member management: real member list, member/admin invite tabs, grant-credits dialog |
| [[features/Classes]] 🟡 | Classes page: ClassGrid + ClassCalendar (Strip/Month views) |
| [[features/Messages]] 🟡 | Two-panel chat UI: direct threads, broadcast threads, in-memory state |
| [[features/Analytics]] 🟡 | Analytics page: 4 KPI cards, 4 Recharts visualizations, period toggle |
| [[features/Settings]] 🟡 | Settings tabs: Gym Info, Profile, Notifications, Appearance |

## UI

| Page | Summary |
|------|---------|
| [[ui/Component Library]] | Inventory of all shared components with props, behaviors, and usage |
| [[ui/Styling & Theme]] | Color palette, glass/glass-card utilities, Tailwind v4 @theme definitions |
| [[ui/Layouts & Navigation]] | Root layout, DashboardLayout, Sidebar nav map, Header |

## Data

| Page | Summary |
|------|---------|
| [[data/Mock Data Layer]] | lib/data/* type shapes, mock values, and the planned Firestore migration |

## Reference

| Page | Summary |
|------|---------|
| [[reference/Project Setup]] | pnpm commands, dev server, first-admin bootstrap, env file setup |
| [[reference/Testing]] | Jest + RTL conventions, node vs jsdom environments, REQ-NN labels, Playwright |
| [[reference/Configuration]] | next.config, tsconfig, jest.config, eslint, postcss, tailwind, CI workflow |
| [[reference/Environment Variables]] | All env var names with purpose (never values) |
| [[reference/Conventions & Gotchas]] | Known quirks, naming oddities, and minor bugs in the current codebase |
