---
title: Auth API Routes
tags: [domain/auth, status/implemented]
status: implemented
sources: ["app/api/auth/login/route.ts", "app/api/auth/logout/route.ts", "app/api/auth/invite/route.ts", "app/api/auth/setup-profile/route.ts", "app/api/auth/complete-profile/route.ts", "app/api/auth/me/route.ts"]
updated: 2026-06-01
---

> **Status:** ✅ Implemented & tested — all routes have test files

# Auth API Routes

All routes under `app/api/auth/`. Excluded from `proxy.ts` matcher (API routes are never redirected). All use the Firebase Admin SDK; all Firestore writes are server-side only.

**Cookie parsing note:** Session cookies are parsed with `.substring('session='.length)` (not `split('=')[1]`), which preserves base64 padding characters (`=` / `==`) at the end of Firebase session tokens.

---

## POST /api/auth/login

**File:** `app/api/auth/login/route.ts`  
**Body:** `{ idToken: string }`

1. `adminAuth.verifyIdToken(idToken)`
2. `adminAuth.createSessionCookie(idToken, { expiresIn: 5 days })`
3. Check/create `Firestore admins/{uid}` (with `pendingInvites` authorization for new docs)
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
**Body:** `{ email: string }`

1. Verify session cookie → capture `inviterUid`
2. Validate email format (regex `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`)
3. `adminAuth.getUserByEmail(email)` — 409 if user exists
4. `adminAuth.generateSignInWithEmailLink(email, settings)`
5. `sendInviteEmail(email, link)` via Resend
6. `Firestore pendingInvites/{email}.set({ invitedBy: inviterUid, createdAt })`

| Status | Condition |
|--------|-----------|
| 200 | Invite sent, pending invite doc written |
| 400 | Invalid email format |
| 401 | Missing/invalid session |
| 409 | Email already registered in Firebase Auth |
| 500 | Firebase / Resend / Firestore failure |

---

## POST /api/auth/setup-profile

**File:** `app/api/auth/setup-profile/route.ts`  
**Body:** `{ idToken: string }`  

Creates the initial (incomplete) `admins/{uid}` Firestore doc. Idempotent — returns 200 if doc already exists.

1. `adminAuth.verifyIdToken(idToken)` → `uid`, `email`
2. If `admins/{uid}` exists → 200 (no-op)
3. Check `pendingInvites/{email}` → 403 if not found
4. `Firestore admins/{uid}.set({ email, invitedBy, profileComplete: false, joinedAt })`
5. `Firestore pendingInvites/{email}.delete()`

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

1. Verify session cookie → `uid`
2. Validate name + surname (required, trimmed)
3. Validate avatar size if provided (max ~100 KB)
4. Generate UI Avatars URL if no avatar
5. `Firestore admins/{uid}.set({ name, surname, photoURL, fallbackAvatar, profileComplete: true }, { merge: true })`
6. Sets `profile_complete=1` cookie (1-year TTL)

| Status | Condition |
|--------|-----------|
| 200 | Profile saved, `profile_complete` cookie set |
| 400 | Missing name/surname, or avatar too large |
| 401 | Missing/invalid session |

---

## GET /api/auth/me

**File:** `app/api/auth/me/route.ts`  
**Auth:** Requires valid `session` cookie

Returns the admin's profile from Firestore.

```ts
Response: {
  uid: string
  name: string | null
  surname: string | null
  email: string | null
  photoURL: string | null
  fallbackAvatar: string
  profileComplete: boolean
}
```

| Status | Condition |
|--------|-----------|
| 200 | Profile returned |
| 401 | Missing/invalid session |
| 404 | Firestore doc not found |

---

## Related pages

- [[auth/Invite & Join Flow]] — how these routes chain together
- [[auth/Login & Logout]] — login and logout routes in detail
- [[auth/Profile & Onboarding]] — complete-profile and me routes
- [[reference/Testing]] — how each route is tested
