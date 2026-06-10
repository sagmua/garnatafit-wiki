---
title: Users
tags: [domain/features, status/implemented]
status: implemented
sources: ["app/(dashboard)/users/page.tsx", "app/api/members/route.ts", "app/api/members/[uid]/credits/route.ts", "app/api/auth/invite/route.ts"]
updated: 2026-06-11
---

> **Status:** ✅ Real member list + invite + credit grants (Firestore via Admin SDK)

# Users

**Route:** `/users`  
**File:** `app/(dashboard)/users/page.tsx`  
Admin-only client component for member management.

## Layout

```
Header: "Users" / "Manage your gym members"
Search bar + "Add user" button
Member table (REAL — fetched from /api/members)
```

## Member Table — now real

The page fetches members from `GET /api/members` (`app/api/members/route.ts`, admin-only via `requireAdmin`), which lists `members/{uid}` docs with `uid`, `name`, `surname`, `email`, `photoURL`, `profileComplete`, and the denormalized `creditsAvailable`. (Previously this table was hardcoded mock data.)

Each row shows the member's identity, credit balance (e.g. "8 credits"), and an action to **grant credits**.

## Add User Modal — both roles functional

`AddUserModal` (inline in `users/page.tsx`) has two role tabs, **Gym member** and **Admin** — both functional now:

- Pick a role, enter an email, submit → `POST /api/auth/invite` with `{ email, role }` (`role: 'member'` or `'admin'`).
- This is the real invite entry point for either audience. See [[auth/Invite & Join Flow]].

### Error / success handling

- On failure, reads `res.json()` and shows the API error (e.g. 409 "This email is already registered").
- On success, the modal shows a confirmation and stops rendering the form.

## Grant Credits Dialog

`GrantCreditsModal` (per member) lets an admin grant a credit batch:

- Pick a **plan** (bundle template, fetched from `GET /api/plans`) — snapshots the template, **or**
- Enter an ad-hoc **credits + validity (value/unit) + optional label**.
- Submits to `POST /api/members/[uid]/credits`, which writes a `creditBatches` doc and bumps `creditsAvailable`.

There is **no credit deduction** anywhere yet — grants only add credit. Full mechanics in [[features/Plans & Credits]].

## Search

Client-side filter over the fetched members list.

## Related pages

- [[auth/Invite & Join Flow]] — what happens after "Send invite" (admin or member)
- [[auth/Roles & Claims]] — roles and the `requireAdmin` guard on `/api/members`
- [[features/Plans & Credits]] — bundle templates and the grant route
- [[features/Member Area]] — what an invited member lands in
- [[overview/Domain Model]] — Member entity (now real)
