---
title: Roles & Claims
tags: [domain/auth, status/implemented]
status: implemented
sources: ["lib/auth/session.ts", "app/api/auth/setup-profile/route.ts", "app/api/auth/login/route.ts", "app/api/auth/invite/route.ts", "app/api/auth/me/route.ts", "proxy.ts", "lib/admin-context.tsx", "docs/design/member-role-and-plans.md"]
updated: 2026-06-11
---

> **Status:** ✅ Implemented & tested

# Roles & Claims

The webapp is now **dual-audience**: it serves both gym **admins** (staff) and gym **members**. Each account has exactly **one role**, carried as a Firebase Auth **custom claim** `role: 'admin' | 'member'`, and this repo owns the shared Firebase backend for both audiences. This supersedes the earlier "members are mobile-only" framing — see [[overview/Project Overview]].

## The `role` Custom Claim

```
Firebase Auth custom claims: { role: 'admin' | 'member' }
```

- **One role per account.** No multi-role accounts.
- **Minted server-side** with `adminAuth.setCustomUserClaims(uid, { role })` during invite acceptance:
  - Primary path: `app/api/auth/setup-profile/route.ts` — reads the role from the `pendingInvites/{email}` doc on first setup; on idempotent repeat calls (invite already consumed) it falls back to the already-minted token claim.
  - Defensive path: `app/api/auth/login/route.ts` — if the profile doc is missing (race/crash recovery), it re-sources the role from the pending invite, mints the claim, and writes the profile.
- **Legacy back-compat:** accounts with **no** `role` claim are treated as **admin** everywhere (`proxy.ts`, `lib/auth/session.ts`, the auth routes). This keeps pre-existing admin accounts working without re-provisioning.

The claim is normalized defensively everywhere with the same idiom: `decoded.role === 'member' ? 'member' : 'admin'`.

## Claim Propagation: forcing the session to carry the claim

A Firebase ID token only reflects custom claims minted **before** it was issued. Because `setCustomUserClaims` runs *during* setup, the invite/complete page forces a token refresh — `getIdToken(true)` — **after** `setup-profile` returns and **before** calling `login`. This guarantees the session cookie minted by `login` carries the fresh `role` claim. See [[auth/Invite & Join Flow]].

## Profiles split by role

Profile documents now live in **one of two collections**, keyed by role:

| Role | Profile collection |
|------|--------------------|
| `admin` | `admins/{uid}` |
| `member` | `members/{uid}` |

`setup-profile` and `login` choose the collection from the role; `GET /api/auth/me` reads from `members` or `admins` depending on the verified claim. `pendingInvites/{email}` docs now carry a `role` field so the collection and claim are decided at invite time.

## Shared session guard — `lib/auth/session.ts`

A reusable guard added this phase removes repeated cookie parsing across API routes:

```ts
type Role = 'admin' | 'member';
interface SessionUser { uid: string; role: Role }

readSessionCookie(req): string | undefined        // parse the `session=` cookie value
getSessionUser(req): Promise<SessionUser | null>   // verify cookie → { uid, role } | null
requireAdmin(req): Promise<{ ok: true; user } | { ok: false; status: 401 | 403 }>
```

- `getSessionUser` verifies the session cookie (`verifySessionCookie(session, true)`); no claim ⇒ `admin`.
- `requireAdmin` returns `401` if unauthenticated, `403` if authenticated but not an admin. Used by every admin-only route (`/api/plans*`, `/api/members*`).
- Member-only routes call `getSessionUser` directly and reject `user.role !== 'member'` with `403` (e.g. `PATCH /api/member/profile`).

## AdminProfile shape (extended)

`lib/admin-context.tsx` `AdminProfile` now exposes `role`, `phone`, and `address` (a structured `MemberAddress`), and `GET /api/auth/me` returns them. The same context/provider backs both the admin dashboard and the member area. See [[auth/Profile & Onboarding]].

## Routing consequences

`proxy.ts` reads the `role` claim from the verified session cookie and confines members to `/member/*`, blocks admins from `/member/*`, and drives the post-login landing (`/member` for members, `/` for admins). Full logic in [[auth/Route Protection]].

## Security note

Firestore **security rules are not authored yet** (deferred to the mobile-integration phase). Today, all privileged access goes through `/api/*` + Admin SDK and the browser never touches Firestore directly. The intended rules model is documented in `docs/design/member-role-and-plans.md`. See [[auth/Firebase Setup]].

## Related pages

- [[auth/Authentication Overview]] — session-cookie model
- [[auth/Route Protection]] — proxy.ts role gating
- [[auth/Invite & Join Flow]] — where the claim is minted and refreshed
- [[auth/Auth API Routes]] — setup-profile / login / me / invite specs
- [[features/Member Area]] — the member-facing pages this unlocks
- [[features/Plans & Credits]] — admin-only credit feature gated by `requireAdmin`
- [[overview/Domain Model]] — Member entity is now real
