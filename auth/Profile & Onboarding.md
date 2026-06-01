---
title: Profile & Onboarding
tags: [domain/auth, status/implemented]
status: implemented
sources: ["app/(auth)/welcome/page.tsx", "app/api/auth/complete-profile/route.ts", "app/api/auth/me/route.ts", "lib/admin-context.tsx", "lib/compress-image.ts", "app/(dashboard)/settings/page.tsx", "components/header/Header.tsx"]
updated: 2026-06-01
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

On submit, `POST /api/auth/complete-profile`. On success, redirects to `/`.

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
5. `Firestore admins/{uid}.set({ name, surname, photoURL, fallbackAvatar: '/default-avatar.svg', profileComplete: true }, { merge: true })`.
6. Sets `profile_complete=1` cookie (1-year TTL).
7. Returns 200 `{ status: 'ok' }`.

## me API — `GET /api/auth/me`

**File:** `app/api/auth/me/route.ts`

Reads the admin's Firestore doc and returns the `AdminProfile` shape. Used by `AdminProvider` on every mount.

```ts
Response: {
  uid, name, surname, email, photoURL, fallbackAvatar, profileComplete
}
```

Returns 404 if the Firestore doc doesn't exist, 401 if the session is missing/invalid.

## AdminContext

**File:** `lib/admin-context.tsx`

React context that makes admin profile data available throughout the dashboard without prop drilling.

- `AdminProvider` — provided at the `DashboardLayout` level. Fetches `/api/auth/me` on mount using `useReducer` (avoids `react-hooks/set-state-in-effect` lint error). A shared `cancelledRef` guards against state updates after unmount (in both the `useEffect` initial fetch and the `refresh()` function).
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
- Password section: form fields exist, Change button shows the form — but the "Update password" button has no `onClick` handler (non-functional; tracked in [[reference/Conventions & Gotchas]]).

## Related pages

- [[auth/Invite & Join Flow]] — where /welcome is reached in the flow
- [[auth/Authentication Overview]] — the two-cookie model and profile_complete gate
- [[ui/Layouts & Navigation]] — Header avatar display
