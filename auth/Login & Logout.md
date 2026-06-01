---
title: Login & Logout
tags: [domain/auth, status/implemented]
status: implemented
sources: ["app/(auth)/login/page.tsx", "app/api/auth/login/route.ts", "app/api/auth/logout/route.ts", "components/header/Header.tsx"]
updated: 2026-06-01
---

> **Status:** ✅ Implemented & tested

# Login & Logout

## Login Flow

**Page:** `/login` — `app/(auth)/login/page.tsx`

1. Admin enters email + password.
2. Client calls `signInWithEmailAndPassword(clientAuth, email, password)` (Firebase client SDK).
3. On success: `user.getIdToken()` to get an ID token.
4. `POST /api/auth/login { idToken }`:
   - `adminAuth.verifyIdToken(idToken)` — validates the token.
   - `adminAuth.createSessionCookie(idToken, { expiresIn: 5 days })` — creates a server-managed session cookie.
   - `Firestore admins/{uid}.get()` — check if profile exists and if it's complete. If the doc is missing AND no `pendingInvites` doc exists, returns 401 (uninvited user). If missing but invite exists, creates the doc defensively.
   - Returns `{ status: 'ok', profileComplete: boolean }`.
   - Sets `session` cookie (HttpOnly, 5-day TTL).
   - If `profileComplete === true`: also sets `profile_complete=1` cookie.
5. Client reads `data.profileComplete`:
   - `true` → redirect to `/`
   - `false` → redirect to `/welcome`

**Error handling:** Firebase error codes map to a generic "Invalid email or password" message (no enumeration). Loading state disables the submit button.

### Login API — `POST /api/auth/login`

| Input | Type | Notes |
|-------|------|-------|
| `idToken` | string (body) | Firebase ID token from client SDK |

| Response | Condition |
|----------|-----------|
| 200 `{ status: 'ok', profileComplete }` + `session` cookie | Valid token |
| 401 | Invalid token, missing email, or non-invited user without admin doc |

### Session Cookie Properties

```
name: session
httpOnly: true
sameSite: strict
secure: true (production only)
maxAge: 5 days (432,000 seconds)
path: /
```

## Forgot Password

**Page:** `/forgot-password` — `app/(auth)/forgot-password/page.tsx`

Calls `sendPasswordResetEmail(clientAuth, email)` from the Firebase client SDK. The page intentionally shows the same success message whether the email exists or not (prevents user enumeration — REQ-13). Client-side email format validation prevents calls for obviously invalid strings. See REQ-12, REQ-13, REQ-14 in `docs/specs/auth-requirements.md`.

## Logout Flow

**Trigger:** Admin clicks Logout in the header dropdown (`components/header/Header.tsx`).

1. `signOut(clientAuth)` — clears the Firebase client-side auth state.
2. `POST /api/auth/logout` — server clears both cookies.
3. Client redirects to `/login`.

### Logout API — `POST /api/auth/logout`

No body required. Clears the `session` cookie and the `profile_complete` cookie by setting `maxAge: 0`. Returns 200.

```
app/api/auth/logout/route.ts
```

## Related pages

- [[auth/Authentication Overview]] — the two-cookie model
- [[auth/Route Protection]] — what happens after session is verified
- [[auth/Invite & Join Flow]] — how new admins get their accounts
