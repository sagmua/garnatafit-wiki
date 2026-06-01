---
title: Conventions & Gotchas
tags: [domain/reference, status/implemented]
status: implemented
sources: ["components/RevenueChart.tsx", "components/DropdownMenu.tsx", "components/classes/ClassGrid.tsx", "components/calendar/StripCalendar.tsx", "components/styles/FullCalendar.css", "app/(dashboard)/settings/page.tsx", "package.json"]
updated: 2026-06-01
---

> **Status:** ✅ Documented — known quirks and intentional conventions

# Conventions & Gotchas

A collection of naming oddities, non-functional UI elements, and minor bugs in the current codebase. Useful context before diving into the code.

---

## Naming Mismatches

### `RevenueChart.tsx` renders "Weekly Attendance"
**File:** `components/RevenueChart.tsx`  
The component is named `RevenueChart` and the file is `RevenueChart.tsx`, but the UI heading it renders is **"Weekly Attendance"**. The chart data is weekly attendance counts (Mon–Sun), not revenue. Revenue data appears in the Analytics page (`AnalyticsPage`). Do not confuse the two.

### `FullCalendar.css` overrides `react-calendar`
**File:** `components/styles/FullCalendar.css`  
Despite the name, this CSS file overrides `.react-calendar` classes from the `react-calendar` npm package. It has no relation to the FullCalendar.io calendar library. The naming is misleading.

### `StripCalendar` is December 2025
**File:** `components/calendar/StripCalendar.tsx`  
The week strip shows "December 2025" as the month header, with hardcoded dates 20–26 and Wednesday always highlighted. It is entirely static — not driven by `new Date()`. This is clearly unfinished; a future implementation would be date-aware.

---

## Non-Functional UI Elements

### "Update password" button does nothing
**File:** `app/(dashboard)/settings/page.tsx` (ProfileTab, password section)  
The "Update password" `<button>` in the Settings Profile tab has no `onClick` handler. Clicking it does nothing. The form is visible and inputs accept text, but no password change is triggered.  
*Issue: tracked in [[features/Settings]]; [[reference/Project Setup#First Admin]] notes this is pending implementation.*

### "Save changes" buttons in Gym Info, Notifications, Appearance tabs
**File:** `app/(dashboard)/settings/page.tsx`  
All "Save changes" buttons in non-Profile tabs have no `onClick` handler. Settings edits are in-memory only and reset on page refresh.

### "Schedule Class" button
**File:** `app/(dashboard)/classes/page.tsx`  
The "Schedule Class" button renders but has no handler. No class creation modal exists.

### "View All" in UpcomingClasses
**File:** `components/UpcomingClasses.tsx`  
The "View All" button has no navigation target or handler.

### DropdownMenu Edit/Delete in ClassGrid
**File:** `components/classes/ClassGrid.tsx`  
The per-row `DropdownMenu` has Edit and Delete options that have no `onClick` handlers.

---

## Structural Quirks

### `DropdownMenu` always renders `options` as `<Link>`
**File:** `components/DropdownMenu.tsx`  
The `DropdownOption` type allows `{ onClick?: () => void; path?: string }` but normal options always render as `<Link href={item.path}>`. If an option has only `onClick` and no `path`, the `href` will be `undefined`. This can cause unexpected navigation behavior. Important options (the red group) correctly use `<button onClick>`.

### `ClassGrid` declares unused `children` prop
**File:** `components/classes/ClassGrid.tsx`  
`ClassGridProps` declares `children?: ReactNode`, but `children` is never rendered anywhere in the component body. The prop has no effect.

### `package-lock.json` alongside `pnpm-lock.yaml`
The repo has both a `package-lock.json` (from an earlier npm install) and a `pnpm-lock.yaml`. Only `pnpm-lock.yaml` is used. The `package-lock.json` is stale and should be deleted. **Always use pnpm** — see [[reference/Project Setup]].

### `garnatafit-firebase-adminsdk.json` at repo root
A Firebase service account JSON file exists at the root. It is gitignored and **not used** by the code — the Admin SDK reads from env vars (`FIREBASE_ADMIN_*`). It should be deleted or moved out of the repo directory.

---

## Intentional Conventions (not bugs)

### `"use client"` only where needed
Server-compatible components (`StatCard`, `UpcomingClasses`, `DropdownMenu`) deliberately omit `"use client"` even though they receive callbacks. This is intentional — they don't use hooks themselves.

### `adminAuth.verifySessionCookie(session, true)` on every request
The `true` argument forces Firebase to check session revocation on every protected request. This is a deliberate security choice at the cost of a Firebase round-trip.

### Forgot-password shows same message for unknown emails
`sendPasswordResetEmail` errors are swallowed and the success message is always shown. This is intentional user enumeration prevention (REQ-13 in `docs/specs/auth-requirements.md`).

### Tests use `/** @jest-environment node */` instead of config
API route tests and the proxy test declare their environment inline rather than in `jest.config.ts`. This is the Next.js/Jest convention for per-file environment overrides.

---

## Related pages

- [[ui/Component Library]] — component props and usage
- [[features/Settings]] — Settings page non-functional buttons
- [[reference/Testing]] — test conventions and REQ-NN labels
