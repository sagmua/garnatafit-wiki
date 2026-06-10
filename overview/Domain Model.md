---
title: Domain Model
tags: [domain/overview, status/mock]
status: mock
sources: ["lib/data/mock_data.ts", "lib/data/mock_messages.ts", "lib/data/mock_analytics.ts", "lib/data/mock_settings.ts", "lib/admin-context.tsx", "lib/credits/types.ts", "app/api/auth/me/route.ts", "app/api/members/route.ts", "docs/design/member-role-and-plans.md"]
updated: 2026-06-11
---

> **Status:** 🟡 Admin, Member, BundleTemplate, CreditBatch & pendingInvites are real; Classes / Threads / Analytics / Gym settings remain mock

# Domain Model

The core entities the GarnataFit system manages, their fields, and their relationships.

## Entities

### Admin
*Source: `lib/admin-context.tsx` (`AdminProfile`), `lib/data/mock_settings.ts` (`AdminSettings`)*

The gym staff member who operates the dashboard. **Real** (Firebase Auth + Firestore `admins/{uid}`). Identified by the `role: 'admin'` custom claim (or no claim → admin). See [[auth/Roles & Claims]].

```ts
// Real — Firestore admins/{uid} + Firebase Auth. Shared shape with Member (AdminProfile).
AdminProfile {
  uid: string
  role?: 'admin' | 'member'        // from the custom claim / me route
  name: string | null
  surname: string | null
  email: string | null
  photoURL: string | null          // base64 JPEG or UI Avatars URL
  fallbackAvatar: string           // always '/default-avatar.svg'
  profileComplete: boolean
  phone?: string | null            // members; optional for admins
  address?: { line1?, city?, postalCode?, country? } | null
}

// Settings — currently mock, future Firestore /admins/{uid}/settings
AdminSettings {
  notifications: { newSignUps, classBookings, newMessages: boolean }
  appearance: { language: "es" | "en" }
}
```

See [[auth/Profile & Onboarding]] for the onboarding flow that creates this record.

### Gym
*Source: `lib/data/mock_settings.ts` (`GymSettings`)*

The tenant organisation. Currently a single tenant ("GarnataFit Studio", Granada). Future: `Firestore /gyms/{gymId}`.

```ts
GymSettings {
  name: string
  logoUrl: string | null
  contact: { phone, email, address: string }
  classCapacityDefault: number     // default: 20
}
```

### Member
*Source: Firestore `members/{uid}`, `app/api/members/route.ts`, `app/api/member/profile/route.ts`*

Gym customers. **Now real** — a standalone Firestore collection `members/{uid}`, created via the role-aware invite flow and identified by the `role: 'member'` custom claim. Members have their own webapp area; see [[features/Member Area]] and [[auth/Roles & Claims]].

```ts
// Real — Firestore members/{uid} (shares the AdminProfile shape)
members/{uid} {
  uid, role: 'member', name, surname, email,
  photoURL, fallbackAvatar, profileComplete,
  phone, address: { line1, city, postalCode, country },
  creditsAvailable: number,         // denormalized counter (see CreditBatch)
  invitedBy, joinedAt
}
```

Sub-collection: `members/{uid}/creditBatches/*` (see CreditBatch below). The legacy `ThreadMember` (`id`, `name`, `avatarUrl`) still exists in `lib/data/mock_messages.ts` for the mock chat UI only.

### BundleTemplate (Plan)
*Source: `lib/credits/types.ts`, `app/api/plans/route.ts`*

A gym-wide sellable product: a bundle of class credits with a validity window. **Real** — `Firestore creditBundleTemplates/{id}`. See [[features/Plans & Credits]].

```ts
BundleTemplate {
  id: string
  name: string
  credits: number
  validity: { unit: 'day' | 'month', value: number }
  description?: string
  active: boolean
  // + createdBy, createdAt server-side
}
```

### CreditBatch (Grant)
*Source: `lib/credits/types.ts`, `app/api/members/[uid]/credits/route.ts`*

A snapshot of a template (or ad-hoc bundle) granted to one member. **Real** — `Firestore members/{uid}/creditBatches/{id}`. Snapshotting freezes the granted values against later template edits. Increments the denormalized `members/{uid}.creditsAvailable`.

```ts
CreditBatch {
  id: string
  amount: number; remaining: number      // remaining == amount (no deduction engine yet)
  grantedAt: string; expiresAt: string   // expiresAt = computeExpiry(grantedAt, validity)
  templateId: string | null              // null for ad-hoc grants
  templateName: string | null            // template name, or the ad-hoc label
  grantedBy: string                       // admin uid
}
```

### Class
*Source: `lib/data/mock_data.ts`*

A scheduled workout session. Future: `Firestore /gyms/{gymId}/classes/{classId}`.

```ts
Class {
  id: number
  name: string            // e.g. "Morning Yoga Flow"
  instructor: string      // denormalized name string
  time: string            // "07:00 AM - 08:00 AM"
  duration: string        // "60 min"
  capacity: number
  enrolled: number
  status: "Open" | "Almost Full" | "Full"
  color: string           // Tailwind bg class, e.g. "bg-purple-500"
}
```

5 mock classes: Morning Yoga Flow, CrossFit WOD, HIIT Blast, Power Lifting, Strength Training.

### Thread / ChatMessage
*Source: `lib/data/mock_messages.ts`*

Admin↔Member communication. Two types: `DirectThread` (one member) and `BroadcastThread` (all members). Future: `Firestore /gyms/{gymId}/threads/{threadId}/messages`.

```ts
Thread = DirectThread | BroadcastThread

DirectThread { id, type: "direct", member: ThreadMember, messages: ChatMessage[], unreadCount }
BroadcastThread { id, type: "broadcast", messages: ChatMessage[], unreadCount }

ChatMessage { id, sender: "admin" | "member", text, timestamp: string /* ISO */ }
```

### Analytics Snapshot
*Source: `lib/data/mock_analytics.ts`*

Aggregated KPI snapshots keyed by time period. Not a stored entity — derived from Members + Classes.

```ts
TimePeriod = "7d" | "30d" | "90d" | "1y"

AnalyticsSnapshot {
  totalMembers, membersTrend, periodRevenue, revenueTrend,
  avgAttendanceRate, attendanceTrend, newSignUps, signUpsTrend,
  memberGrowth: TimeSeriesPoint[]
  revenueByPeriod: RevenuePoint[]
  classAttendance: ClassAttendance[]
  classTypeBreakdown: ClassTypeBreakdown[]
}
```

### pendingInvites
*Source: `app/api/auth/invite/route.ts`, `app/api/auth/setup-profile/route.ts`*

An authorization token collection — **real**, in Firestore. Written when an admin sends an invite; consumed (then deleted) when the invitee completes setup. Prevents unauthorized users from self-creating admin/member records. Now carries a `role` field that decides the minted claim and the target profile collection.

```
Firestore pendingInvites/{email} { invitedBy: uid, role: 'admin' | 'member', createdAt: Timestamp }
```

## Relationships

```
Gym (1) ──── (*) Admin
Gym (1) ──── (*) Member
Gym (1) ──── (*) Class
Gym (1) ──── (*) BundleTemplate
Member (1) ─ (*) CreditBatch          [members/{uid}/creditBatches/*]
CreditBatch (*) ─ (0..1) BundleTemplate  [snapshot; templateId nullable]
Class (*) ── (1) Instructor  [denormalized string — no Instructor entity]
Member (*) ─ (*) Class       [reservations are in-memory only this phase]
Admin (1) ── (*) Thread
Thread (direct, 1) ─── (1) Member
Thread (*) ── (*) ChatMessage
Analytics ── aggregates over Members + Classes per TimePeriod
```

## Data Maturity

| Entity | Storage | Status |
|--------|---------|--------|
| Admin | Firebase Auth + Firestore `admins/{uid}` | ✅ Real |
| Member | Firebase Auth + Firestore `members/{uid}` | ✅ Real |
| BundleTemplate | Firestore `creditBundleTemplates/{id}` | ✅ Real |
| CreditBatch | Firestore `members/{uid}/creditBatches/{id}` | ✅ Real |
| pendingInvites | Firestore `pendingInvites/{email}` (+`role`) | ✅ Real |
| Gym settings | `lib/data/mock_settings.ts` | 🟡 Mock |
| Class | `lib/data/mock_data.ts` | 🟡 Mock |
| Reservation | in-memory (`lib/member/reservations-context.tsx`) | 🟡 Mock (not persisted) |
| Thread / ChatMessage | `lib/data/mock_messages.ts` | 🟡 Mock |
| Analytics | `lib/data/mock_analytics.ts` | 🟡 Mock |

> Firestore **security rules** for these real collections are not authored yet — see `docs/design/member-role-and-plans.md` and [[auth/Firebase Setup]].

## Related pages

- [[auth/Authentication Overview]] — Admin/Member auth model
- [[auth/Roles & Claims]] — role claim & profile collections
- [[features/Plans & Credits]] — BundleTemplate & CreditBatch in action
- [[features/Member Area]] — Member entity in action
- [[data/Mock Data Layer]] — full type definitions from mock files
- [[features/Messages]] — Thread entity in action
- [[features/Analytics]] — AnalyticsSnapshot in action
