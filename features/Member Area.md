---
title: Member Area
tags: [domain/features, status/mock]
status: mock
sources: ["app/(member)/layout.tsx", "app/(member)/member/page.tsx", "app/(member)/member/classes/page.tsx", "app/(member)/member/reservations/page.tsx", "app/(member)/member/messages/page.tsx", "app/(member)/member/profile/page.tsx", "app/(member)/member/profile/page.test.tsx", "components/MemberLayout.tsx", "lib/member/reservations-context.tsx", "app/api/member/profile/route.ts", "app/api/member/credits/route.ts", "docs/design/member-role-and-plans.md"]
updated: 2026-06-11
---

> **Status:** 🟡 Mostly mock/UI this phase — profile + credits are real; classes, reservations, and chat are not persisted

# Member Area

The member-facing half of the dual-audience webapp, served from the new `app/(member)/` route group at `/member/*`. Access is gated by the `role: 'member'` claim — `proxy.ts` confines members here and blocks admins. See [[auth/Roles & Claims]] and [[auth/Route Protection]].

## Layout — `MemberLayout`

`components/MemberLayout.tsx` mirrors `DashboardLayout` but reuses the shared `Sidebar` via its new `items` prop (member nav) and wraps content in both `AdminProvider` (profile context) and `ReservationsProvider`. Member nav items:

```
Home            /member               LayoutDashboard
Classes         /member/classes       Calendar
My Reservations /member/reservations  CalendarCheck
Messages        /member/messages      MessageSquare
Profile         /member/profile       User
```

`app/(member)/layout.tsx` wraps the group in `MemberLayout`. See [[ui/Layouts & Navigation]].

## Screens

| Screen | Route | Status | Backing |
|--------|-------|--------|---------|
| Home | `/member` | 🟡 partial | Shows the member's **real** credits via `GET /api/member/credits`; rest is UI |
| Classes (browse) | `/member/classes` | 🟡 mock | Browse classes; Join/Cancel via in-memory context |
| My Reservations | `/member/reservations` | 🟡 mock | Reads the in-memory reservations context |
| Messages | `/member/messages` | 🟡 mock | Chat with the gym — mock, not persisted |
| Profile | `/member/profile` | ✅ real | Self-edit details + password (see below) |

### Reservations — in-memory only

`lib/member/reservations-context.tsx` (`ReservationsProvider` / `useReservations`) holds reserved class IDs in client `useState`. `join`/`cancel`/`isReserved` operate on this array. **No persistence** — state resets on reload, by design this phase. There is **no credit deduction** on join (see [[features/Plans & Credits]]).

### Profile — real self-service

`app/(member)/member/profile/page.tsx` has two real cards:

- **Personal details** → `PATCH /api/member/profile` (`app/api/member/profile/route.ts`): member-only (`getSessionUser` + `role === 'member'`, else `403`). Edits `name`, `surname`, `phone`, and a structured `address` (`line1`, `city`, `postalCode`, `country`), merged into `members/{uid}`. **Email and role are intentionally not editable here**; name/surname are required (`400` otherwise). Calls `refresh()` on the shared context after save.
- **Password** → single "Send reset link" button (`sendPasswordResetEmail` via Firebase client SDK). Sends a reset email to the member's address. No in-app re-auth or `updatePassword` flow exists.

Member onboarding still collects only name/surname/avatar at `/welcome`; phone/address are added later here in the profile screen.

## Credit read

`GET /api/member/credits` returns the signed-in member's own `creditBatches`. Credits are granted by admins — see [[features/Plans & Credits]].

## Maturity

| Piece | Status |
|-------|--------|
| Route group + layout + nav | ✅ Real |
| Member profile self-edit | ✅ Real (`members/{uid}`) |
| Password reset (email link) | ✅ Real (`sendPasswordResetEmail`) |
| Member credits display | ✅ Real (`GET /api/member/credits`) |
| Classes browse | 🟡 Mock |
| Reservations (Join/Cancel) | 🟡 In-memory, not persisted |
| Chat with gym | 🟡 Mock |

## Related pages

- [[auth/Roles & Claims]] — the `member` role and member-scoped guards
- [[auth/Route Protection]] — how members are confined to `/member/*`
- [[features/Plans & Credits]] — where the credits shown here come from
- [[ui/Layouts & Navigation]] — MemberLayout and the shared Sidebar `items` prop
- [[features/Users]] — the admin side that invites members and grants credits
