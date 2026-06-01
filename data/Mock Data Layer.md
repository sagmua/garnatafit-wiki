---
title: Mock Data Layer
tags: [domain/data, status/mock]
status: mock
sources: ["lib/data/mock_data.ts", "lib/data/mock_settings.ts", "lib/data/mock_messages.ts", "lib/data/mock_analytics.ts"]
updated: 2026-06-01
---

> **Status:** 🟡 All mock — Firestore migration planned for all non-auth entities

# Mock Data Layer

All domain data except auth/admin-profile is served from static TypeScript files under `lib/data/`. Each file contains inline `// future: Firestore /gyms/{gymId}/...` comments indicating the planned persistence path.

## lib/data/mock_data.ts — Classes

No exported TypeScript types (untyped array). Used by `ClassGrid` and `UpcomingClasses`.

```ts
// Inferred shape:
{
  id: number
  name: string          // "Morning Yoga Flow"
  instructor: string    // "Elena Fisher"
  time: string          // "07:00 AM - 08:00 AM"
  duration: string      // "60 min"
  capacity: number
  enrolled: number
  status: "Open" | "Almost Full" | "Full"
  color: string         // Tailwind bg class, e.g. "bg-purple-500"
}
```

5 entries. See [[features/Classes]] for the full table.

**Future Firestore path:** `// future: Firestore /gyms/{gymId}/classes`

## lib/data/mock_settings.ts — Gym & Admin Settings

```ts
export type Language = "es" | "en";

export const SUPPORTED_LANGUAGES: { code: Language; label: string }[] = [
  { code: "es", label: "Español" },
  { code: "en", label: "English" },
];

export interface GymSettings {
  name: string
  logoUrl: string | null
  contact: { phone: string; email: string; address: string }
  classCapacityDefault: number
}

export interface AdminSettings {
  displayName: string
  email: string
  avatarUrl: string | null
  passwordLastChangedAt: string
  notifications: {
    newSignUps: boolean
    classBookings: boolean
    newMessages: boolean
  }
  appearance: { language: Language }
}
```

`gymSettings` default: name "GarnataFit Studio", Granada contact info, `classCapacityDefault: 20`.  
`adminSettings` default: displayName "Samuel Cardenete", email "admin@garnatafit.com", language "es".

**Future Firestore paths:** `// future: Firestore /gyms/{gymId}` and `/admins/{uid}/settings`

## lib/data/mock_messages.ts — Chat Threads

```ts
export type MessageSender = "admin" | "member";
export type ThreadType = "direct" | "broadcast";

export interface ChatMessage {
  id: string
  sender: MessageSender
  text: string
  timestamp: string   // ISO string — future: Firestore Timestamp
}

export interface ThreadMember {
  id: string          // "u1"–"u5"
  name: string
  avatarUrl: string   // Unsplash URL
}

interface BaseThread {
  id: string
  messages: ChatMessage[]
  unreadCount: number
}
export interface DirectThread extends BaseThread { type: "direct"; member: ThreadMember }
export interface BroadcastThread extends BaseThread { type: "broadcast" }
export type Thread = DirectThread | BroadcastThread;
```

`mockMembers`: 5 members — Alex Johnson, Maria Garcia, James Wilson, Sarah Connor, Mike Ross.  
`mockThreads`: 5 threads (3 direct + 2 broadcast), each with pre-seeded messages.

**Future Firestore path:** `// future: Firestore /gyms/{gymId}/threads + /threads/{threadId}/messages`

## lib/data/mock_analytics.ts — Analytics Snapshots

```ts
export type TimePeriod = "7d" | "30d" | "90d" | "1y";

export interface TimeSeriesPoint { label: string; value: number }
export interface RevenuePoint    { label: string; revenue: number }
export interface ClassAttendance { name: string; attended: number; capacity: number }
export interface ClassTypeBreakdown { name: string; value: number; color: string }

export interface AnalyticsSnapshot {
  totalMembers: number;   membersTrend: number;
  periodRevenue: number;  revenueTrend: number;
  avgAttendanceRate: number; attendanceTrend: number;
  newSignUps: number;     signUpsTrend: number;
  memberGrowth: TimeSeriesPoint[];
  revenueByPeriod: RevenuePoint[];
  classAttendance: ClassAttendance[];
  classTypeBreakdown: ClassTypeBreakdown[];
}

export const analyticsData: Record<TimePeriod, AnalyticsSnapshot>
```

All 4 period snapshots are pre-populated with realistic mock values.

## Migration Strategy (planned)

All `lib/data/` files are intended to be replaced by Firestore reads under `Firestore /gyms/{gymId}/`. The admin webapp will remain single-tenant initially (one gym), so the `gymId` would be a fixed constant until multi-tenancy is needed.

The Auth/Admin layer is already real — serving as the template for how future domain entities will be migrated (server-side Admin SDK writes, no client-side Firestore rules).

## Related pages

- [[overview/Domain Model]] — entity relationships and maturity table
- [[features/Classes]], [[features/Messages]], [[features/Analytics]], [[features/Settings]] — consumers of this data
