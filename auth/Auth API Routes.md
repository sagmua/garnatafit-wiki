---
title: Auth API Routes
tags: [domain/auth, status/implemented]
status: implemented
sources: ["app/api/auth/login/route.ts", "app/api/auth/logout/route.ts", "app/api/auth/invite/route.ts", "app/api/auth/setup-profile/route.ts", "app/api/auth/complete-profile/route.ts", "app/api/auth/me/route.ts", "lib/auth/session.ts", "lib/email/templates/invite.ts"]
updated: 2026-06-11
---

> **Status:** ✅ Implemented & tested — all routes have test files

# Auth API Routes

All routes under `app/api/auth/`. Excluded from `proxy.ts` matcher (API routes are never redirected). All use the Firebase Admin SDK; all Firestore writes are server-side only.

> This page documents the `/api/auth/*` routes. The **Plans/credits** and **members/member** routes (`/api/plans*`, `/api/members*`, `/api/member/*`) are documented in [[features/Plans & Credits]] and [[features/Member Area]]; they share the `lib/auth/session.ts` guard described in [[auth/Roles & Claims]].

**Cookie parsing note:** Session cookies are parsed with `.substring('session='.length)` (not `split('=')[1]`), which preserves base64 padding characters (`=` / `==`) at the end of Firebase session tokens. This same logic is now centralized in `lib/auth/session.ts` (`readSessionCookie`).

---

## POST /api/auth/login

**File:** `app/api/auth/login/route.ts`  
**Body:** `{ idToken: string }`

1. `adminAuth.verifyIdToken(idToken)` — also reads the `role` claim (default `admin`)
2. `adminAuth.createSessionCookie(idToken, { expiresIn: 5 days })`
3. Check/create the profile in `members/{uid}` or `admins/{uid}` (by claim). Defensive first-time creation re-sources the role from `pendingInvites`, mints the claim via `setCustomUserClaims`, and writes the matching collection.
4. Returns `{ status: 'ok', profileComplete: boolean }` + `session` cookie (+ `profile_complete` if complete)

| Status | Condition |
|--------|-----------|
| 200 | Valid token; session cookie set |
| 401 | Invalid token, no email on token, or no pending invite for new admin |

---

## POST /api/auth/logout

**File:** `app/api/auth/logout/route.ts`  
**Body:** none

Clears both `session` and `profile_complete` cookies (`maxAge: 0`).

| Status | Condition |
|--------|-----------|
| 200 | Always |

---

## POST /api/auth/invite

**File:** `app/api/auth/invite/route.ts`  
**Auth:** Requires valid `session` cookie  
**Body:** `{ email: string, role?: 'admin' | 'member' }`

1. Verify session cookie → capture `inviterUid`
2. Validate email format (regex `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`)
3. Resolve role: `rawRole === undefined ? 'admin' : rawRole`; reject anything not in `['admin','member']` with 400
4. `adminAuth.getUserByEmail(email)` — 409 if user exists
5. `adminAuth.generateSignInWithEmailLink(email, settings)`
6. `sendInviteEmail(email, link)` via Resend — email copy is generic ("You've been invited to join GarnataFit"), suitable for both admin and member invites (`lib/email/templates/invite.ts`)
7. `Firestore pendingInvites/{email}.set({ invitedBy: inviterUid, role, createdAt })`

| Status | Condition |
|--------|-----------|
| 200 | Invite sent, pending invite doc written |
| 400 | Invalid email format, or invalid `role` |
| 401 | Missing/invalid session |
| 409 | Email already registered in Firebase Auth |
| 500 | Firebase / Resend / Firestore failure |

---

## POST /api/auth/setup-profile

**File:** `app/api/auth/setup-profile/route.ts`  
**Body:** `{ idToken: string }`  

Creates the initial (incomplete) profile doc. Idempotent — returns 200 if doc already exists. **Mints the `role` custom claim** and writes into the role-appropriate collection (`members/{uid}` or `admins/{uid}`).

1. `adminAuth.verifyIdToken(idToken)` → `uid`, `email`, token `role`
2. Resolve role: from `pendingInvites/{email}.role` if the invite exists; otherwise from the already-minted token claim (idempotent repeats). Default `admin`. Pick `collection` accordingly.
3. If the profile doc already exists → delete the invite (if present) and return 200 (no-op)
4. If no invite exists (and no doc) → 403
5. `adminAuth.setCustomUserClaims(uid, { role })`
6. `Firestore <collection>/{uid}.set({ email, invitedBy, role, profileComplete: false, joinedAt })`
7. `Firestore pendingInvites/{email}.delete()`

| Status | Condition |
|--------|-----------|
| 200 | Profile created (or already existed) |
| 400 | Missing idToken |
| 401 | Invalid idToken |
| 403 | No pending invite found for this email |

---

## POST /api/auth/complete-profile

**File:** `app/api/auth/complete-profile/route.ts`  
**Auth:** Requires valid `session` cookie  
**Body:** `{ name: string, surname: string, avatar?: string }`

1. Verify session cookie → `uid`, `role` (decoded from the session cookie claim)
2. Validate name + surname (required, trimmed)
3. Validate avatar size if provided (max ~100 KB)
4. Generate UI Avatars URL if no avatar
5. `Firestore <collection>/{uid}.set({ name, surname, photoURL, fallbackAvatar, profileComplete: true }, { merge: true })` — collection is `members` for role `member`, `admins` otherwise
6. Sets `profile_complete=1` cookie (1-year TTL)
7. Returns `{ status: 'ok', role }`

| Status | Condition |
|--------|-----------|
| 200 | Profile saved, `profile_complete` cookie set; body contains `{ status: 'ok', role }` |
| 400 | Missing name/surname, or avatar too large |
| 401 | Missing/invalid session |

---

## GET /api/auth/me

**File:** `app/api/auth/me/route.ts`  
**Auth:** Requires valid `session` cookie

Returns the signed-in user's profile from Firestore. Reads from `members/{uid}` or `admins/{uid}` depending on the `role` claim. Now includes `role`, `phone`, and `address`.

```ts
Response: {
  uid: string
  role: 'admin' | 'member'
  name: string | null
  surname: string | null
  email: string | null
  photoURL: string | null
  fallbackAvatar: string
  profileComplete: boolean
  phone: string | null
  address: { line1?, city?, postalCode?, country? } | null
}
```

| Status | Condition |
|--------|-----------|
| 200 | Profile returned |
| 401 | Missing/invalid session |
| 404 | Firestore doc not found |

---

## Related pages

- [[auth/Roles & Claims]] — role claim, profile collections, `lib/auth/session.ts` guard
- [[auth/Invite & Join Flow]] — how these routes chain together
- [[auth/Login & Logout]] — login and logout routes in detail
- [[auth/Profile & Onboarding]] — complete-profile and me routes
- [[features/Plans & Credits]] — `/api/plans*` and `/api/members/[uid]/credits` routes
- [[features/Member Area]] — `/api/member/profile` and `/api/member/credits` routes
- [[reference/Testing]] — how each route is tested
