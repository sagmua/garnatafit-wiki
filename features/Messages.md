---
title: Messages
tags: [domain/features, status/mock]
status: mock
sources: ["app/(dashboard)/messages/page.tsx", "lib/data/mock_messages.ts"]
updated: 2026-06-01
---

> **Status:** 🟡 Mock data — in-memory only; no Firestore persistence

# Messages

**Route:** `/messages`  
**File:** `app/(dashboard)/messages/page.tsx`  
Client component. Two-panel chat UI for admin↔member communication.

## Layout

```
Left panel: Thread list
├── Search bar (filters by member name)
├── Thread rows (member avatar + name, last message preview, timestamp, unread count)
└── "Compose" button

Right panel: Active conversation
├── Thread header (member name or "Broadcast")
├── Message bubbles (admin = right-aligned gym-green, member = left-aligned gray)
├── Auto-scroll to latest
└── Message input + send
```

Mobile: panels toggle — list hides when a conversation is open.

## Thread Types

### Direct Thread
Targets one `ThreadMember`. Shows the member's avatar (Unsplash URL in mock). Clicking marks `unreadCount → 0`.

### Broadcast Thread
A message to all members. Renders a `Megaphone` icon instead of an avatar. Replies from the admin side are visible but in a real implementation would fan out to all members.

## Data

Seeded from `lib/data/mock_messages.ts`. `MessagesPage` initialises `useState(mockThreads)`. All interactions (send, mark read, sort) are purely in-memory — refresh resets everything.

5 mock threads: Alex Johnson (direct, 2 unread), Maria Garcia (direct, 0), Broadcast: Weekly Schedule (broadcast, 1 unread), James Wilson (direct, 0), Broadcast: Maintenance (broadcast, 0).

## Key Behaviors

- **Search:** Filters `DirectThread` entries by `thread.member.name`. Broadcasts are always shown.
- **Send message:** Appends a `ChatMessage` with `sender: "admin"` and `timestamp: new Date().toISOString()`. Re-sorts threads by last message timestamp (most recent first).
- **Relative timestamps:** A `formatTimestamp()` helper converts ISO strings to relative labels ("Just now", "X min ago", "X hours ago", "Yesterday", or a locale date string).
- **ComposeModal:** Opens on "Compose" button. Lets admin select Direct or Broadcast and pick a member from `mockMembers`. Creates a new thread or appends to an existing one (finds by member id). All in-memory.

## Inner Components

Both are defined inline in `messages/page.tsx`:
- `MessageBubble` — renders a single chat message with correct alignment and timestamp.
- `ComposeModal` — thread creation dialog.

## Real Data Path

When Firestore is wired up, threads would live at `Firestore /gyms/{gymId}/threads/{threadId}` with a `messages` sub-collection. Real-time sync via Firestore listeners would power the live chat. Broadcast threads would fan out to member devices via FCM push notifications.

## Related pages

- [[data/Mock Data Layer]] — Thread, ChatMessage, ThreadMember types
- [[overview/Domain Model]] — Thread/Message entities and relationships
