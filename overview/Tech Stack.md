---
title: Tech Stack
tags: [domain/overview, status/implemented]
status: implemented
sources: ["package.json"]
updated: 2026-06-01
---

> **Status:** ✅ Documented from package.json

# Tech Stack

Internal package name: `gym-dashboard` v0.1.0. Package manager: **pnpm** (use pnpm, never npm).

## Core Framework

| Package | Version | Purpose |
|---------|---------|---------|
| `next` | `16.0.7` | Framework. App Router, server components, API routes, proxy.ts middleware |
| `react` | `19.2.0` | UI runtime |
| `react-dom` | `19.2.0` | DOM renderer |
| `typescript` | `^5` | Type safety across the codebase |

## Auth & Backend

| Package | Version | Purpose |
|---------|---------|---------|
| `firebase` | `^12.13.0` | Client SDK — `signInWithEmailLink`, `signOut`, `updatePassword`, `clientAuth` |
| `firebase-admin` | `^13.10.0` | Admin SDK — `verifySessionCookie`, `generateSignInWithEmailLink`, `verifyIdToken`, `createSessionCookie`, Firestore writes |
| `resend` | `^6.12.3` | Transactional email — sends branded invite emails |

## UI

| Package | Version | Purpose |
|---------|---------|---------|
| `@headlessui/react` | `2.2.9` | Accessible unstyled primitives — `Menu`/`MenuButton`/`MenuItem` used in `DropdownMenu` |
| `lucide-react` | `^0.555.0` | Icon component library |
| `recharts` | `^3.5.1` | Charts — AreaChart, BarChart, PieChart used on Analytics and Dashboard pages |
| `react-calendar` | `^6.0.0` | Month calendar widget used in `MonthCalendar.tsx` |

## Styling

| Package | Version | Purpose |
|---------|---------|---------|
| `tailwindcss` | `^4` | Utility CSS. v4 uses `@import "tailwindcss"` + `@theme` in globals.css |
| `@tailwindcss/postcss` | `^4` | Tailwind v4 PostCSS integration |
| `clsx` | `^2.1.1` | Conditional class name construction |
| `tailwind-merge` | `^3.4.0` | Deduplicate conflicting Tailwind classes |

The `cn()` utility in `lib/utils.ts` wraps both: `twMerge(clsx(inputs))`. Used pervasively.

See [[ui/Styling & Theme]] for the full color palette and custom utilities.

## Testing

| Package | Version | Purpose |
|---------|---------|---------|
| `jest` | `^30.4.2` | Test runner |
| `jest-environment-jsdom` | `^30.4.1` | jsdom env for component tests |
| `@testing-library/react` | `^16.3.2` | Component rendering + queries |
| `@testing-library/user-event` | `^14.6.1` | Simulated user interactions |
| `@testing-library/jest-dom` | `^6.9.1` | Custom DOM matchers (`.toBeInTheDocument()`, etc.) |
| `@playwright/test` | `^1.60.0` | E2e test runner (configured, minimal tests so far) |

See [[reference/Testing]] for conventions and test file locations.

## Build & CI

- No CI pipeline for lint/test/build — only `claude.yml` GitHub Action (Claude Code bot for PR reviews)
- Deployment target: Vercel (`.vercel` is gitignored; no deploy workflow committed)

## Related pages

- [[reference/Configuration]] — how each config file is wired together
- [[reference/Environment Variables]] — env vars consumed by firebase and resend
- [[ui/Styling & Theme]] — Tailwind v4 theme definitions
