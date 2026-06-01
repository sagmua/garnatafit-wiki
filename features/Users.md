---
title: Users
tags: [domain/features, status/mock]
status: mock
sources: ["app/(dashboard)/users/page.tsx"]
updated: 2026-06-01
---

> **Status:** 🟡 Mock data — member list is hardcoded inline; only the invite flow is real

# Users

**Route:** `/users`  
**File:** `app/(dashboard)/users/page.tsx`  
Client component. Member management page.

## Layout

```
Header: "Users" / "Manage gym members and staff"
Search bar + "Add user" button
Member table (5 hardcoded mock members)
```

## Member Table

5 hardcoded members (inline `users` array, not from `lib/data/`):

| Name | Status | Membership |
|------|--------|-----------|
| Alex Johnson | Active | Pro Plan |
| Maria Garcia | Active | Standard |
| James Wilson | Inactive | Basic |
| Sarah Connor | Active | Pro Plan |
| Mike Ross | Pending | Standard |

Columns: Avatar, Name/Email, Status badge, Membership tier, Last check-in, Actions (MoreHorizontal icon — no-op).

Status badge colors: Active → gym-green, Inactive → red-500, Pending → yellow-500.

## Add User Modal

`AddUserModal` is an inline component in `users/page.tsx`. Triggered by the "Add user" button.

### Role tabs

Two tabs: **Member** and **Admin**.

- **Member tab:** Non-functional placeholder ("Member invite coming soon" or similar static text).
- **Admin tab:** Email input + "Send invite" button → `POST /api/auth/invite`. This is the **real** invite flow entry point. See [[auth/Invite & Join Flow]].

### Error handling

On a failed invite response, reads `res.json()` and displays the API error message directly. This means:
- 409 duplicate → shows "This email is already registered"
- Generic failure → shows "Failed to send invite."

### Success state

After a successful invite, the modal shows a confirmation ("Invite sent!") and no longer renders the form.

## Search

Client-side filter over the hardcoded `users` array. Filters by name.

## Real Data Path

When Firestore is wired up, the member list would read from `Firestore /gyms/{gymId}/members`, and the `Pending` status would map to unapproved sign-ups requiring admin action.

## Related pages

- [[auth/Invite & Join Flow]] — what happens after "Send invite"
- [[data/Mock Data Layer]] — mock member data shapes
- [[overview/Domain Model]] — Member entity
