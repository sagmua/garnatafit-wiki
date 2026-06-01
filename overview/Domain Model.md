---
title: Domain Model
tags: [domain/overview, status/mock]
status: mock
sources: ["lib/data/mock_data.ts", "lib/data/mock_messages.ts", "lib/data/mock_analytics.ts", "lib/data/mock_settings.ts", "lib/admin-context.tsx", "app/api/auth/me/route.ts"]
updated: 2026-06-01
---

> **Status:** 🟡 Auth/Admin entity is real; all other entities are mock data pending Firestore migration

# Domain Model

The core entities the GarnataFit system manages, their fields, and their relationships.

## Entities

### Admin
*Source: `lib/admin-context.tsx` (`AdminProfile`), `lib/data/mock_settings.ts` (`AdminSettings`)*

The gym staff member who operates the admin webapp. This entity is **real** (Firebase Auth + Firestore `admins/{uid}`).

```ts
// Real — Firestore admins/{uid} + Firebase Auth
AdminProfile {
  uid: string
  name: string | null
  surname: string | null
  email: string | null
  photoURL: string | null          // base64 JPEG or UI Avatars URL
  fallbackAvatar: string           // always '/default-avatar.svg'
  profileComplete: boolean
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
*Source: `lib/data/mock_messages.ts` (`ThreadMember`), analytics KPIs*

Gym customers. They appear as thread counterparties and as aggregate counts (Total Members ~1,240, New Sign-ups). Not yet a standalone Firestore collection. Future: `Firestore /gyms/{gymId}/members/{uid}`.

```ts
ThreadMember {
  id: string            // "u1"–"u5" in mock
  name: string
  avatarUrl: string     // Unsplash URL in mock
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

An authorization token collection — **real**, in Firestore. Written when an admin sends an invite; consumed (then deleted) when the invitee completes setup. Prevents unauthorized users from self-creating admin records.

```
Firestore pendingInvites/{email} { invitedBy: uid, createdAt: Timestamp }
```

## Relationships

```
Gym (1) ──── (*) Admin
Gym (1) ──── (*) Member
Gym (1) ──── (*) Class
Class (*) ── (1) Instructor  [denormalized string — no Instructor entity]
Member (*) ─ (*) Class       [implicit; modeled only as enrolled counts]
Admin (1) ── (*) Thread
Thread (direct, 1) ─── (1) Member
Thread (*) ── (*) ChatMessage
Analytics ── aggregates over Members + Classes per TimePeriod
```

## Data Maturity

| Entity | Storage | Status |
|--------|---------|--------|
| Admin | Firebase Auth + Firestore `admins/{uid}` | ✅ Real |
| pendingInvites | Firestore `pendingInvites/{email}` | ✅ Real |
| Gym settings | `lib/data/mock_settings.ts` | 🟡 Mock |
| Member | `lib/data/mock_messages.ts` + KPI counts | 🟡 Mock |
| Class | `lib/data/mock_data.ts` | 🟡 Mock |
| Thread / ChatMessage | `lib/data/mock_messages.ts` | 🟡 Mock |
| Analytics | `lib/data/mock_analytics.ts` | 🟡 Mock |

## Related pages

- [[auth/Authentication Overview]] — Admin auth model
- [[data/Mock Data Layer]] — full type definitions from mock files
- [[features/Messages]] — Thread entity in action
- [[features/Analytics]] — AnalyticsSnapshot in action
