---
title: Invite & Join Flow
tags: [domain/auth, status/implemented]
status: implemented
sources: ["app/api/auth/invite/route.ts", "app/(auth)/invite/complete/page.tsx", "app/api/auth/setup-profile/route.ts", "app/api/auth/login/route.ts", "app/(auth)/welcome/page.tsx", "app/api/auth/complete-profile/route.ts", "docs/design/auth-invite-flow.md"]
updated: 2026-06-11
---

> **Status:** ✅ Implemented & tested

# Invite & Join Flow

The only way to add a new account — **admin or member** — is through an explicit invite from an existing admin; the invite carries the **role**. The flow spans four phases and crosses multiple pages, API routes, Firebase services, and Firestore collections. The role determines which custom claim is minted and which profile collection is used. See [[auth/Roles & Claims]].

A full **Mermaid sequence diagram** lives in the webapp repo at `docs/design/auth-invite-flow.md`.

## Phase 1 — Send Invite

**Trigger:** Existing admin opens Users page → "Add user" modal → picks **Member** or **Admin** → enters email → submits.

**Route:** `POST /api/auth/invite` — body `{ email, role? }`

1. Verify the caller's `session` cookie (`adminAuth.verifySessionCookie`). Capture `inviterUid`.
2. Validate email format (regex).
3. Resolve role: `role ?? 'admin'`; reject if not `'admin' | 'member'` (**400**).
4. `adminAuth.getUserByEmail(email)` — if user exists, return **409** (duplicate check).
5. `adminAuth.generateSignInWithEmailLink(email, settings)` — generates a Firebase OOB sign-in link. URL embeds the invitee's email as a query param: `/invite/complete?email=<encoded>`.
6. `sendInviteEmail(email, link)` — sends a branded HTML email via **Resend** (see `lib/email/resend.ts` and `lib/email/templates/invite.ts`).
7. `Firestore pendingInvites/{email}.set({ invitedBy: inviterUid, role, createdAt })` — authorization token (now carrying the role) for phase 3.

**Error cases:** 401 (no/invalid session), 400 (bad email or invalid role), 409 (already registered), 500 (Firebase/Resend/Firestore failure).

## Phase 2 — Accept Invite & Set Password

**Page:** `/invite/complete?email=<email>&oobCode=...`  
**File:** `app/(auth)/invite/complete/page.tsx`

This is a public page (no session required — listed in the proxy's public-pages allowlist).

1. `isSignInWithEmailLink(clientAuth, window.location.href)` — verifies the link is valid. If not, shows error; form is not rendered.
2. Admin sets a password (min 8 chars) + confirm.
3. On submit:
   - `signInWithEmailLink(clientAuth, email, url)` — email read from URL `?email=` query param (not localStorage — cross-device safe). Creates the Firebase Auth account and signs in.
   - `updatePassword(user, password)` — sets the chosen password on the account.
   - `user.getIdToken()` — gets an ID token for `setup-profile`.
4. After `setup-profile` mints the claim, the page calls **`getIdToken(true)`** (force refresh) so the next token carries the new `role` claim, then calls `login`. Without this refresh the session cookie would lack the role. See [[auth/Roles & Claims]].

**Error case:** Invalid/expired link shows a static error message. Link is a Firebase OOB code with a limited TTL.

## Phase 3 — Setup Profile & Create Session

**Routes:** `POST /api/auth/setup-profile` then `POST /api/auth/login`

### setup-profile

1. `adminAuth.verifyIdToken(idToken)` — get `uid`, `email`, token `role`.
2. Resolve role from `pendingInvites/{email}.role` (first setup) or the token claim (idempotent repeats); pick collection `members` vs `admins`.
3. If the profile doc already exists, return 200 idempotently (handles retries; deletes the invite if still present).
4. If no pending invite (and no doc), return **403** (prevents uninvited Firebase users from creating records).
5. `adminAuth.setCustomUserClaims(uid, { role })` — mint the role claim.
6. `Firestore <collection>/{uid}.set({ email, invitedBy, role, profileComplete: false, joinedAt })`.
7. `Firestore pendingInvites/{email}.delete()` — consume the authorization token.

### login

1. `adminAuth.verifyIdToken(idToken)` (reads the now-refreshed `role` claim) + `adminAuth.createSessionCookie(idToken, 5 days)`.
2. Profile doc lookup uses the claim's collection. If the doc doesn't exist (race / crash recovery), re-source role from `pendingInvites`, `setCustomUserClaims`, and create the doc defensively in the matching collection.
3. Returns `{ status: 'ok', profileComplete: false }` + sets `session` cookie.
4. **Does not set `profile_complete` cookie** because `profileComplete === false`.

**Client:** On 200, reads `data.profileComplete = false` → redirects to `/welcome`. After onboarding, the proxy sends admins to `/` and members to `/member`.

## Phase 4 — Onboarding (Complete Profile)

**Page:** `/welcome`  
**File:** `app/(auth)/welcome/page.tsx`

The proxy lets through any request to `/welcome` with a valid session, regardless of `profile_complete`. This page is mandatory before dashboard access.

1. Admin fills in `name`, `surname` (required), and optionally uploads a profile photo.
2. Photo upload: client-side compression via `compressImage()` (`lib/compress-image.ts`) → resizes to 200×200 JPEG at 0.7 quality.
3. `POST /api/auth/complete-profile { name, surname, avatar? }`:
   - Verify session cookie.
   - If `avatar` is a `data:image/jpeg;base64,...` string, validate it's under ~100 KB.
   - If no avatar, generate a UI Avatars URL (`https://ui-avatars.com/api/?name=...`).
   - `Firestore admins/{uid}.set({ name, surname, photoURL, fallbackAvatar: '/default-avatar.svg', profileComplete: true }, { merge: true })`.
   - Sets `profile_complete=1` cookie (1-year TTL).
4. Client redirects to `/` (dashboard). The proxy now allows through — both cookies present.

## Avatar Fallback Chain

```
photoURL (base64 JPEG or UI Avatars URL)
  → if img error: fallbackAvatar ('/default-avatar.svg')
  → /welcome page before name typed: UserRound Lucide icon (no img at all)
```

## Error & Edge Cases

| Scenario | Handling |
|----------|----------|
| Email already registered | Phase 1 returns 409 before generating link |
| Invalid/expired invite link | Phase 2 shows static error, no form rendered |
| Server crash between Auth creation and Firestore write | Phase 3 login route defensively recreates admin doc (checks pendingInvites first) |
| Non-invited user calls setup-profile | Phase 3 returns 403 |
| pendingInvites doc already deleted (double submit) | setup-profile returns 403 on retry; login's defensive path also returns 403 |

## Related pages

- [[auth/Authentication Overview]] — big picture
- [[auth/Roles & Claims]] — role minting, claim refresh, profile collections
- [[auth/Auth API Routes]] — input/output specs for each API route
- [[auth/Profile & Onboarding]] — AdminContext, header avatar, settings profile tab
- [[features/Users]] — AddUserModal (Member/Admin tabs) that triggers Phase 1
