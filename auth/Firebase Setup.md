---
title: Firebase Setup
tags: [domain/auth, status/implemented]
status: implemented
sources: ["lib/firebase/admin.ts", "lib/firebase/client.ts", "next.config.ts", "docs/design/member-role-and-plans.md"]
updated: 2026-06-11
---

> **Status:** ✅ Implemented

# Firebase Setup

The app uses two Firebase SDK instances: the **Admin SDK** (server-only) and the **Client SDK** (browser). They must never be mixed.

## Admin SDK

**File:** `lib/firebase/admin.ts`

Used exclusively in API routes and `proxy.ts`. Handles session cookie creation/verification, ID token verification, sign-in link generation, and all Firestore writes.

**Initialization (singleton pattern):**
```ts
function getAdminApp() {
  if (getApps().length > 0) return getApps()[0];
  return initializeApp({
    credential: cert({ projectId, clientEmail, privateKey }),
  });
}
```

`privateKey` is read from `process.env.FIREBASE_ADMIN_PRIVATE_KEY` with `.replace(/\\n/g, '\n')` to un-escape newlines that get collapsed in env strings.

**Exports:** `adminAuth` (`getAuth(adminApp)`), `adminDb` (`getFirestore(adminApp)`).

**Required env vars:**
- `FIREBASE_ADMIN_PROJECT_ID`
- `FIREBASE_ADMIN_CLIENT_EMAIL`
- `FIREBASE_ADMIN_PRIVATE_KEY`

**next.config.ts** sets `serverExternalPackages: ["firebase-admin"]` so the Admin SDK is treated as an external Node package and is not bundled by webpack.

**Note:** `garnatafit-firebase-adminsdk.json` exists at the repo root and is gitignored. It is not used by the code — the admin SDK reads from env vars, not the JSON file.

## Client SDK

**File:** `lib/firebase/client.ts`

Used in client components and auth pages (`"use client"`). Handles: `signInWithEmailAndPassword`, `signInWithEmailLink`, `isSignInWithEmailLink`, `updatePassword`, `signOut`, `sendPasswordResetEmail`.

**Initialization (singleton pattern):**
```ts
const app = !getApps().length ? initializeApp(firebaseConfig) : getApps()[0];
```

**Exports:** `clientAuth` (`getAuth(app)`), `clientDb` (`getFirestore(app)`).

**Required env vars** (all `NEXT_PUBLIC_` — exposed to the browser):
- `NEXT_PUBLIC_FIREBASE_API_KEY`
- `NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN`
- `NEXT_PUBLIC_FIREBASE_PROJECT_ID`
- `NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET`
- `NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID`
- `NEXT_PUBLIC_FIREBASE_APP_ID`

## Firebase Project Roles

The same Firebase project serves both audiences (admins and members) and will serve the future mobile app. Firebase Auth is the identity provider, with a **`role` custom claim** (`admin | member`) minted server-side — see [[auth/Roles & Claims]].

**Security rules status:** Real `firestore.rules` are **NOT authored yet** — deferred to the mobile-integration phase. Today, the browser never touches Firestore directly; every read/write goes through `/api/*` + the Admin SDK, and admin-only routes are guarded by `requireAdmin` (`lib/auth/session.ts`). The **intended** rules model is documented in `docs/design/member-role-and-plans.md`:

```
members/{uid}              — member reads/writes own doc (limited fields); admins full
members/{uid}/creditBatches/* — member read-only own; writes server-only
creditBundleTemplates/*    — admin write, authenticated read
pendingInvites/*           — server-only
```

## Firestore Collections

| Collection | Key | Purpose | Written by |
|------------|-----|---------|-----------|
| `admins` | `{uid}` | Admin profile | Admin SDK (setup-profile, login, complete-profile) |
| `members` | `{uid}` | Member profile + `creditsAvailable` | Admin SDK (setup-profile, login; member profile route) |
| `members/{uid}/creditBatches` | `{id}` | Granted credit batches | Admin SDK (grant route) |
| `creditBundleTemplates` | `{id}` | Gym-wide plan catalog | Admin SDK (plans routes) |
| `pendingInvites` | `{email}` | Invite authorization tokens (+`role`) | Admin SDK (invite route) |

## Related pages

- [[auth/Roles & Claims]] — custom claims and the shared session guard
- [[auth/Auth API Routes]] — which API routes use which Firebase methods
- [[features/Plans & Credits]] — the credit collections in context
- [[reference/Environment Variables]] — full env var list with purposes
- [[overview/Domain Model]] — Firestore collections in context
