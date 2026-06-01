---
title: Firebase Setup
tags: [domain/auth, status/implemented]
status: implemented
sources: ["lib/firebase/admin.ts", "lib/firebase/client.ts", "next.config.ts"]
updated: 2026-06-01
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

The same Firebase project serves both apps. Firebase Auth is the identity provider for admins. Firestore is used for:
- `admins/{uid}` — admin profile docs
- `pendingInvites/{email}` — invite authorization tokens

**Security rules:** Deny all client-side reads/writes. All mutations go through the Admin SDK (server-side).

## Firestore Collections

| Collection | Key | Purpose | Written by |
|------------|-----|---------|-----------|
| `admins` | `{uid}` | Admin profile | Admin SDK (setup-profile, login, complete-profile routes) |
| `pendingInvites` | `{email}` | Invite authorization tokens | Admin SDK (invite route) |

## Related pages

- [[auth/Auth API Routes]] — which API routes use which Firebase methods
- [[reference/Environment Variables]] — full env var list with purposes
- [[overview/Domain Model]] — Firestore collections in context
