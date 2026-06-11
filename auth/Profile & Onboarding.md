---
title: Profile & Onboarding
tags: [domain/auth, status/implemented]
status: implemented
sources: ["app/(auth)/welcome/page.tsx", "app/api/auth/complete-profile/route.ts", "app/api/auth/me/route.ts", "lib/admin-context.tsx", "lib/compress-image.ts", "app/(dashboard)/settings/page.tsx", "app/(member)/member/profile/page.tsx", "app/api/member/profile/route.ts", "components/header/Header.tsx"]
updated: 2026-06-11
---

> **Status:** ✅ Implemented & tested

# Profile & Onboarding

## The /welcome Onboarding Page

**File:** `app/(auth)/welcome/page.tsx`

Required immediately after invite acceptance, before any dashboard access. The proxy allows `/welcome` only for sessions without `profile_complete` set; it redirects completed admins to `/`.

**Fields:**
- Name (required)
- Surname (required)
- Profile photo (optional) — triggers browser file picker, compressed client-side before upload

**Avatar preview behavior:**
- No name typed + no photo uploaded → `UserRound` Lucide icon (no broken image)
- Name typed but no photo → UI Avatars URL previewed in real time
- Photo uploaded → compressed JPEG preview
- Image load error → falls back silently

**Client-side compression** (`lib/compress-image.ts`):
- `compressImage(file, maxSize = 200, quality = 0.7): Promise<string>`
- Reads via `FileReader`, scales to fit 200×200 (never upscales), draws to `<canvas>`, returns `data:image/jpeg;base64,...`

On submit, `POST /api/auth/complete-profile`. On success, reads `data.role` from the response and redirects to `/member` for members or `/` for admins.

## complete-profile API — `POST /api/auth/complete-profile`

**File:** `app/api/auth/complete-profile/route.ts`

| Input | Type | Validation |
|-------|------|-----------|
| `name` | string (body) | Required, trimmed |
| `surname` | string (body) | Required, trimmed |
| `avatar` | string? (body) | Optional `data:image/...;base64,...`; max ~100 KB decoded |

**Behavior:**
1. Verify `session` cookie.
2. Validate name + surname.
3. If `avatar` is a base64 data URI: validate size (`base64.length > MAX_AVATAR_BYTES * 1.37` check).
4. If no avatar: generate `https://ui-avatars.com/api/?name=<encoded>&background=random&size=200`.
5. Resolves the target collection from the role decoded from the session cookie: `members` for `role === 'member'`, `admins` otherwise. `Firestore <collection>/{uid}.set({ name, surname, photoURL, fallbackAvatar: '/default-avatar.svg', profileComplete: true }, { merge: true })`.
6. Sets `profile_complete=1` cookie (1-year TTL).
7. Returns 200 `{ status: 'ok', role }`.

## me API — `GET /api/auth/me`

**File:** `app/api/auth/me/route.ts`

Reads the signed-in user's Firestore doc (from `members/{uid}` or `admins/{uid}` per the `role` claim) and returns the `AdminProfile` shape. Used by `AdminProvider` on every mount, in **both** the admin and member areas.

```ts
Response: {
  uid, role, name, surname, email, photoURL,
  fallbackAvatar, profileComplete, phone, address
}
```

`role`, `phone`, and `address` were added this phase (see [[auth/Roles & Claims]]). Returns 404 if the Firestore doc doesn't exist, 401 if the session is missing/invalid.

## AdminContext

**File:** `lib/admin-context.tsx`

React context that makes profile data available throughout the dashboard **and the member area** without prop drilling (both `DashboardLayout` and `MemberLayout` wrap their content in `AdminProvider`). The `AdminProfile` type now includes optional `role`, `phone`, and `address` (`MemberAddress`: `line1`/`city`/`postalCode`/`country`).

- `AdminProvider` — Fetches `/api/auth/me` on mount using `useReducer` (avoids `react-hooks/set-state-in-effect` lint error). A shared `cancelledRef` guards against state updates after unmount (in both the `useEffect` initial fetch and the `refresh()` function).
- `useAdmin()` hook — returns `{ admin: AdminProfile | null, loading: boolean, refresh: () => void }`.
- `refresh()` — re-fetches `/api/auth/me` and dispatches `loaded` action. Called after the Settings profile save to update the header immediately.

**Consumers:**
- `Header.tsx` — reads `admin.name`, `admin.surname`, `admin.photoURL`, `admin.fallbackAvatar` for the avatar and name display.
- `settings/page.tsx` (ProfileTab) — reads profile fields for the edit form; calls `refresh()` after save.

## Avatar Fallback Chain

```
1. admin.photoURL         (Firestore: base64 JPEG or UI Avatars URL)
2. admin.fallbackAvatar   (always '/default-avatar.svg')
3. onError handler        (if the image URL fails to load, imgError state → fallback)
```

On the `/welcome` page before any name is typed: a `UserRound` icon is shown instead (no `<img>` tag at all).

## Settings — Profile Tab

**File:** `app/(dashboard)/settings/page.tsx` (`ProfileTab` component)

- Pre-populates name/surname from `useAdmin()`. A `useEffect` rehydrates the fields when the async `profile` data loads (avoiding stale empty state if the user opens Settings before `/api/auth/me` resolves).
- Calls `POST /api/auth/complete-profile` on save (same endpoint as onboarding).
- Checks `res.ok` before calling `setSaved(true)` — shows error message on failure.
- Calls `refresh()` after a successful save so the Header avatar and name update immediately.
- Email field is read-only (disabled input). "Contact support to change your login email."
- Password section: a single "Send reset link" button calls `sendPasswordResetEmail` (Firebase client SDK) for the signed-in admin's email. Displays an inline ok/error message. No in-app re-auth or `updatePassword` flow.

## Member Self-Service Profile

Members edit their own profile at `/member/profile` (`app/(member)/member/profile/page.tsx`), separate from the admin Settings tab.

- **Personal details** → `PATCH /api/member/profile` (`app/api/member/profile/route.ts`): member-only (`getSessionUser` + `role === 'member'`, else 403). Updates `name`, `surname`, `phone`, structured `address`, merged into `members/{uid}`. **Email and role are not editable here**; name/surname required (400 otherwise). Calls `refresh()` after save.
- **Password** → single "Send reset link" button (`sendPasswordResetEmail` via Firebase client SDK). Sends a reset email to the member's own address. No in-app re-auth or `updatePassword` flow.

Full member-area detail in [[features/Member Area]].

## Related pages

- [[auth/Roles & Claims]] — role claim and `/api/auth/me` extensions
- [[auth/Invite & Join Flow]] — where /welcome is reached in the flow
- [[auth/Authentication Overview]] — the two-cookie model and profile_complete gate
- [[features/Member Area]] — member self-service profile & password
- [[ui/Layouts & Navigation]] — Header avatar display
