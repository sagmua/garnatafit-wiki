---
title: Project Setup
tags: [domain/reference, status/implemented]
status: implemented
sources: ["CLAUDE.md", "package.json", "lib/firebase/admin.ts", "lib/firebase/client.ts", "scripts/test-e2e.sh", "firebase.json"]
updated: 2026-06-11
---

> **Status:** ✅ Documented

# Project Setup

## Prerequisites

- Node.js (version compatible with Next.js 16)
- pnpm (never npm)
- A Firebase project with Auth and Firestore enabled
- A Resend account (for invite emails)
- ngrok or similar tunnel for testing invite links remotely (Firebase requires HTTPS for sign-in links)
- **For e2e tests only:** Java 21 (for emulator JARs), system Chrome. Firebase CLI is a `devDependency` — `pnpm install` provides it, no global install required

## Commands

```bash
pnpm dev         # Start dev server at http://localhost:3000
pnpm build       # Production build
pnpm start       # Run production server
pnpm lint        # Run ESLint
pnpm test        # Run Jest unit tests (115 tests, 24 suites)
pnpm test:watch  # Jest watch mode
pnpm test:e2e    # Run Playwright e2e tests against Firebase emulators (17 tests)
```

`pnpm test:e2e` calls `scripts/test-e2e.sh`, which spins up Firebase emulators (Auth + Firestore), seeds fixture users, and runs Playwright tests on port 3001. The dev server on 3001 is started automatically by Playwright's `webServer` config if not already running. See [[reference/Testing]] for the full e2e architecture.

## Environment Variables

Create `.env.local` at the repo root with the following. See [[reference/Environment Variables]] for the full list with descriptions.

All Firebase Admin vars (`FIREBASE_ADMIN_*`) come from your Firebase service account JSON. The `NEXT_PUBLIC_*` vars come from your Firebase project's web app config.

## First Admin (Bootstrap)

New admin accounts can only be created via the invite flow, which requires an existing admin. To bootstrap the first admin:

1. Go to the **Firebase Console → Authentication → Users** and create a user manually (email + password).
2. In **Firestore**, create the document `admins/{uid}` with:
   ```json
   {
     "email": "admin@example.com",
     "invitedBy": null,
     "profileComplete": false,
     "joinedAt": <Firestore timestamp>
   }
   ```
   Set `profileComplete: false` so the admin is directed to `/welcome` on first login.
3. Alternatively: set `profileComplete: true` and add `name`/`surname`/`photoURL`/`fallbackAvatar` fields to skip the onboarding step.

All subsequent admins are added via the invite flow from the Users page. See [[auth/Invite & Join Flow]].

## Firestore Rules

Set Firestore security rules to **deny all** client reads/writes — the app never writes to Firestore from the browser; all mutations go through the Admin SDK on the server.

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

## Firebase Authorized Domains

For the invite link to work, your app's domain must be in Firebase Console → Authentication → Settings → Authorized Domains. Add:
- `localhost` (already there by default)
- Your ngrok domain (for remote testing)
- Your production domain

## Testing Invites Remotely (ngrok)

The invite email contains a link to `NEXT_PUBLIC_APP_URL`. For the Firebase OOB code to validate on a different device, that URL must be an authorized domain. Static ngrok domain used during development: `ungruesome-jimmie-blousier.ngrok-free.dev`.

Set `NEXT_PUBLIC_APP_URL=https://your-ngrok-domain` in `.env.local`, add the domain to Firebase authorized domains, and run an ngrok tunnel pointing at port 3000.

## Related pages

- [[reference/Environment Variables]] — all env var names with purposes
- [[auth/Firebase Setup]] — SDK initialization details
- [[auth/Invite & Join Flow]] — the full onboarding sequence
