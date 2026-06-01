---
title: Authentication Overview
tags: [domain/auth, status/implemented]
status: implemented
sources: ["proxy.ts", "app/api/auth/", "lib/firebase/admin.ts", "lib/firebase/client.ts"]
updated: 2026-06-01
---

> **Status:** ✅ Implemented & tested — 49 tests passing

# Authentication Overview

The GarnataFit admin webapp uses **Firebase Authentication** with **server-side session cookies** managed via the Firebase Admin SDK. All protected routes are gated by a `proxy.ts` middleware that runs before every non-static, non-API page request.

## The Two-Cookie Model

Every logged-in session is controlled by two HTTP-only cookies:

| Cookie | Set by | Value | Purpose |
|--------|--------|-------|---------|
| `session` | `POST /api/auth/login` | Firebase session cookie (base64url) | Proves identity. Verified server-side on every request. 5-day TTL. |
| `profile_complete` | `POST /api/auth/complete-profile` | `"1"` | Flags that the admin has completed the onboarding form. Permanent (1-year TTL). |

Both cookies are `httpOnly: true`, `sameSite: strict`, and `secure: true` in production.

## Route Protection Rules (proxy.ts)

```
Request to a page route
├── Public page (/login, /forgot-password, /invite/*)
│   ├── No session → pass through
│   └── Valid session → redirect to / or /welcome (keep them out of auth pages)
│
├── /welcome (requires session, not profile completion)
│   ├── No session → redirect to /login
│   ├── Session + profile_complete=1 → redirect to / (already done)
│   └── Session, no profile_complete → pass through (onboarding)
│
└── All other routes (dashboard)
    ├── No session → redirect to /login
    ├── Session, no profile_complete → redirect to /welcome
    └── Session + profile_complete → pass through ✓
```

Full details in [[auth/Route Protection]].

## Admin Onboarding: 4-Phase Flow

New admins are added exclusively via an invite sent by an existing admin. The flow has four phases:

1. **Send invite** — existing admin sends email via `POST /api/auth/invite`; a `pendingInvites/{email}` Firestore doc is written for authorization.
2. **Accept & set password** — invitee clicks link, lands on `/invite/complete`, sets a password, Firebase Auth account is created; `POST /api/auth/setup-profile` verifies the pending invite and creates the `admins/{uid}` Firestore doc.
3. **Session** — `POST /api/auth/login` verifies identity, creates the session cookie; `profile_complete` is NOT set yet.
4. **Onboarding** — invitee is redirected to `/welcome`, fills in name/surname/photo; `POST /api/auth/complete-profile` saves the profile and sets the `profile_complete` cookie.

Full Mermaid sequence diagram in the webapp repo at `docs/design/auth-invite-flow.md`. See [[auth/Invite & Join Flow]].

## First Admin (Bootstrap)

The first admin account is created manually in the Firebase Console. All subsequent admins are added via the invite flow.

## Auth Maturity

All auth pages, API routes, and the proxy have **unit tests** under `app/api/auth/` and `__tests__/middleware.test.ts`. The test suite has 49 passing tests. See [[reference/Testing]].

## Security Principles

- `verifySessionCookie(session, true)` — the `true` forces revocation checks on every request.
- All Firestore writes go through the Admin SDK (server-side only). Firestore security rules: **deny all** client access.
- `pendingInvites` authorization prevents any Firebase Auth user from self-creating an admin record.
- Error messages never leak implementation details to the client (e.g. "Failed to send invite" not "Firestore write timed out").
- Forgot-password flow deliberately swallows errors to prevent user enumeration.

## Related pages

- [[auth/Invite & Join Flow]] — detailed 4-phase onboarding sequence
- [[auth/Login & Logout]] — login API, session creation, logout
- [[auth/Route Protection]] — proxy.ts decision tree
- [[auth/Auth API Routes]] — all /api/auth/* endpoints
- [[auth/Firebase Setup]] — SDK initialization and env vars
