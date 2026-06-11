---
title: Environment Variables
tags: [domain/reference, status/implemented]
status: implemented
sources: ["lib/firebase/admin.ts", "lib/firebase/client.ts", "lib/email/resend.ts", "app/api/auth/invite/route.ts", "scripts/test-e2e.sh", "playwright.config.ts"]
updated: 2026-06-11
---

> **Status:** ✅ Documented — names only, never values

# Environment Variables

All env vars are loaded from `.env.local` (gitignored). A `.env` file exists but is empty.

## Firebase Admin SDK (server-only)

Used in `lib/firebase/admin.ts`. Never exposed to the browser.

| Variable | Source | Used for |
|----------|--------|---------|
| `FIREBASE_ADMIN_PROJECT_ID` | Firebase service account | Admin SDK initialization |
| `FIREBASE_ADMIN_CLIENT_EMAIL` | Firebase service account | Admin SDK initialization |
| `FIREBASE_ADMIN_PRIVATE_KEY` | Firebase service account | Admin SDK initialization; newlines un-escaped with `.replace(/\\n/g, '\n')` |

These three vars replace the need for a service account JSON file. The `garnatafit-firebase-adminsdk.json` at the repo root is gitignored and unused.

## Firebase Client SDK (browser-exposed)

Used in `lib/firebase/client.ts`. All have `NEXT_PUBLIC_` prefix — they are embedded in the client bundle.

| Variable | Used for |
|----------|---------|
| `NEXT_PUBLIC_FIREBASE_API_KEY` | Firebase client config |
| `NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN` | Firebase client config (e.g. `project.firebaseapp.com`) |
| `NEXT_PUBLIC_FIREBASE_PROJECT_ID` | Firebase client config |
| `NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET` | Firebase client config |
| `NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID` | Firebase client config |
| `NEXT_PUBLIC_FIREBASE_APP_ID` | Firebase client config |

## Application

| Variable | Used in | Purpose |
|----------|---------|---------|
| `NEXT_PUBLIC_APP_URL` | `app/api/auth/invite/route.ts` | Base URL for invite links (e.g. `https://yourdomain.com`). Must match a Firebase authorized domain. |
| `RESEND_API_KEY` | `lib/email/resend.ts` | Resend API key for sending invite emails |
| `RESEND_FROM_EMAIL` | `lib/email/resend.ts` | From address for invite emails. Falls back to `'GarnataFit <onboarding@resend.dev>'` if not set. |

## Runtime / CI

| Variable | Used in | Purpose |
|----------|---------|---------|
| `NODE_ENV` | Cookie security flags in API routes | `secure: process.env.NODE_ENV === 'production'` |
| `CI` | `playwright.config.ts` | Sets `forbidOnly`, `retries: 2`, `workers: 1` in CI |

## E2e / Emulator (test-time only)

Set by `scripts/test-e2e.sh` before Playwright runs. Never added to `.env.local` for normal development.

| Variable | Value in e2e | Purpose |
|----------|-------------|---------|
| `FIREBASE_AUTH_EMULATOR_HOST` | `127.0.0.1:9099` | Tells Firebase SDKs (Admin + client) to use the Auth emulator; also suppresses real email sends in `lib/email/resend.ts` |
| `FIRESTORE_EMULATOR_HOST` | `127.0.0.1:8080` | Tells Firebase Admin SDK to use the Firestore emulator |
| `FIREBASE_ADMIN_PROJECT_ID` | `demo-garnatafit` | Project ID used by Admin SDK in emulator mode (no real project) |
| `NEXT_PUBLIC_USE_EMULATORS` | `'true'` | Triggers `connectAuthEmulator` + `connectFirestoreEmulator` calls in `lib/firebase/client.ts` |

## Related pages

- [[auth/Firebase Setup]] — how the Firebase vars are consumed
- [[reference/Project Setup]] — where to set these up
