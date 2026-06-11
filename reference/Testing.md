---
title: Testing
tags: [domain/reference, status/implemented]
status: implemented
sources: ["jest.config.ts", "jest.setup.ts", "__tests__/middleware.test.ts", "app/api/auth/", "app/api/plans/", "app/api/members/", "app/api/member/", "lib/credits/", "lib/member/", "app/(member)/", "playwright.config.ts", "firebase.json", "scripts/test-e2e.sh", ".github/workflows/e2e.yml", "__tests__/e2e/global.setup.ts", "__tests__/e2e/auth.spec.ts", "__tests__/e2e/users.spec.ts", "__tests__/e2e/member-home.spec.ts", "__tests__/e2e/member-classes.spec.ts", "__tests__/e2e/member-profile.spec.ts"]
updated: 2026-06-11
---

> **Status:** ✅ 115 Jest unit tests passing across 24 suites; 17 Playwright e2e tests passing across 5 spec files

# Testing

## Test Stack

| Tool | Version | Purpose |
|------|---------|---------|
| Jest | `^30.4.2` | Unit/integration test runner; configured via `next/jest` |
| React Testing Library | `^16.3.2` | Component rendering and DOM queries |
| `@testing-library/user-event` | `^14.6.1` | Simulated user interactions |
| `@testing-library/jest-dom` | `^6.9.1` | Custom matchers (`.toBeInTheDocument()`, etc.) |
| Playwright | `^1.60.0` | E2e browser tests; runs against Firebase emulators |

---

## Unit Tests (Jest)

### Configuration

**`jest.config.ts`:**
- Uses `next/jest.js` (configures Next.js module resolution, path aliases, SWC transforms)
- Default `testEnvironment: 'jsdom'` (for component tests)
- `setupFilesAfterEnv: ['./jest.setup.ts']`
- `moduleNameMapper`: `^@/(.*)$ → <rootDir>/$1` mirrors the tsconfig `@/*` path alias
- **Ignores `__tests__/e2e`** (Playwright specs) so they do not run under Jest

**`jest.setup.ts`:** Imports `@testing-library/jest-dom` matchers. Polyfills `global.ResizeObserver` (no-op) needed by Recharts and calendar components in jsdom.

**Run command:** `pnpm test` (Jest only; e2e excluded)

### Test Environments

Two environments are used:

| Environment | Used for | How to set |
|-------------|---------|------------|
| `jsdom` (default) | Component/page tests | No annotation needed |
| `node` | API route tests and proxy test | `/** @jest-environment node */` at top of file |

Node environment is required for tests that use `Request`, `NextRequest`, and Firebase Admin SDK mocks.

### Test File Locations

Tests are **colocated** with their source files as `*.test.ts(x)`. Two exceptions live in `__tests__/`:

| File | Tests |
|------|-------|
| `__tests__/middleware.test.ts` | proxy.ts route protection (9 tests, node env) |
| `app/(auth)/login/page.test.tsx` | Login page UI |
| `app/(auth)/forgot-password/page.test.tsx` | Forgot-password page UI |
| `app/(auth)/invite/complete/page.test.tsx` | Invite complete page UI |
| `app/(dashboard)/users/page.test.tsx` | Users page / AddUserModal |
| `components/header/Header.test.tsx` | Header component |
| `app/api/auth/logout/route.test.ts` | Logout API route |
| `app/api/auth/login/route.test.ts` | Login API route |
| `app/api/auth/invite/route.test.ts` | Invite API route |
| `app/api/auth/setup-profile/route.test.ts` | Setup-profile API route |
| `app/api/auth/complete-profile/route.test.ts` | Complete-profile API route |

The `feature/member-role-and-plans` phase added suites for the credits library, the plans/members/member API routes, the member pages, and the reservations context (`lib/credits/validate.test.ts`, `app/api/plans/*`, `app/api/members/*`, `app/api/member/*`, `app/(member)/member/classes/page.test.tsx`, `app/(member)/member/profile/page.test.tsx`, `lib/member/reservations-context.test.tsx`, etc.). The member profile test pins a **stable `useAdmin` mock** — an unstable one made `useEffect([admin])` re-render forever (the real `AdminProvider` is stable).

**Total: 24 Jest test suites, 115 tests** (Playwright e2e excluded).

### REQ-NN Labels

Tests are labeled with requirement IDs matching `docs/specs/auth-requirements.md`. This creates a traceable link between requirements and assertions. Example:

```ts
// REQ-01: no cookie on a protected route → redirect to /login
test('REQ-01: no session cookie on dashboard route redirects to /login', ...
```

### API Route Test Pattern

API route tests (node env):
1. Mock `@/lib/firebase/admin` with `jest.mock()`.
2. Construct a real `Request` or `NextRequest` object with `new Request(url, { method, headers, body })`.
3. Call the exported route handler directly: `await POST(req)`.
4. Assert `res.status` and `await res.json()`.
5. Assert mock calls: `expect(mockFn).toHaveBeenCalledWith(...)`.

**Cookie parsing in tests:** Tests that require session auth set `cookie: 'session=valid-session'` in the headers. The route parses this with `.substring('session='.length)`.

### Mocking Firebase

Firebase is always mocked in tests — never real SDK calls:

```ts
jest.mock('@/lib/firebase/admin', () => ({
  adminAuth: { verifySessionCookie: jest.fn(), /* ... */ },
  adminDb: { collection: jest.fn(() => ({ doc: jest.fn(() => ({ get: jest.fn(), set: jest.fn() })) })) },
}));
```

Then `jest.requireMock('@/lib/firebase/admin')` is used to access and configure mock return values per test.

---

## E2e Tests (Playwright)

### Overview

17 Playwright tests run against the real Next.js app wired to **Firebase emulators** (no production Firebase touched). The `global.setup.ts` seeds deterministic fixture users before any spec runs.

**Run command:** `pnpm test:e2e` (wraps `scripts/test-e2e.sh`)

### Infrastructure

**`playwright.config.ts`:**
- `testDir: './__tests__/e2e'`
- `fullyParallel: true`, Chromium-only (system Chrome via `channel: 'chrome'`)
- `baseURL: 'http://localhost:3001'` (dev server on port 3001, not 3000, to avoid colliding with a running dev session)
- `globalSetup: './__tests__/e2e/global.setup.ts'`
- `webServer`: starts `pnpm dev --port 3001`; `reuseExistingServer: !process.env.CI`
- CI: `forbidOnly`, `retries: 2`, `workers: 1`

**`firebase.json`:**
- Firebase Auth emulator on port 9099
- Firestore emulator on port 8080
- Emulator UI disabled

**`scripts/test-e2e.sh`:**
- Exports emulator env vars (`FIREBASE_AUTH_EMULATOR_HOST`, `FIRESTORE_EMULATOR_HOST`, `FIREBASE_ADMIN_PROJECT_ID=demo-garnatafit`, `NEXT_PUBLIC_USE_EMULATORS=true`, and stub client SDK vars so Next.js starts without real Firebase config)
- Runs: `firebase emulators:exec --only auth,firestore --project demo-garnatafit -- "pnpm exec playwright test ..."`

**Emulator hooks in source code:**
- `lib/firebase/admin.ts` — skips `cert()` when `FIREBASE_AUTH_EMULATOR_HOST` is set; uses `initializeApp({ projectId })` instead (credential-free)
- `lib/firebase/client.ts` — calls `connectAuthEmulator` + `connectFirestoreEmulator` when `typeof window !== 'undefined' && NEXT_PUBLIC_USE_EMULATORS === 'true'`; wrapped in try/catch for HMR safety
- `lib/email/resend.ts` — early-returns (skips real email sends) when `FIREBASE_AUTH_EMULATOR_HOST` is set

### Global Setup (`__tests__/e2e/global.setup.ts`)

Runs once before all specs:
1. **Clears emulators** via REST DELETE on both Auth and Firestore emulator endpoints
2. **Seeds admin user** (`uid: e2e-admin-001`, `admin-e2e@test.com`) — `role: admin` custom claim + `admins` Firestore doc (`profileComplete: true`)
3. **Seeds member user** (`uid: e2e-member-001`, `member-e2e@test.com`) — `role: member` custom claim + `members` Firestore doc (`creditsAvailable: 8`, phone/address fields) + a `creditBatches` sub-document (10 granted / 8 remaining, name: "Starter Pack")
4. **Signs each user in** via Auth emulator REST API (`identitytoolkit.googleapis.com/v1/accounts:signInWithPassword`), exchanges the ID token for a session cookie via `POST /api/auth/login`, saves Playwright `storageState` to `__tests__/e2e/.auth/admin.json` and `__tests__/e2e/.auth/member.json`

The `.auth/` directory is gitignored (`.gitignore` entry: `/__tests__/e2e/.auth/`).

### Spec Files

#### `auth.spec.ts` (7 tests)

| Test | Assertion |
|------|-----------|
| Unauthenticated `/` | Redirects to `/login` |
| Unauthenticated `/member` | Redirects to `/login` |
| Login page renders | Heading, email field, password field, Sign In button visible |
| Admin login | Signs in, lands on `/`, dashboard heading visible |
| Member login | Signs in, lands on `/member`, "Welcome back" text visible |
| Cross-role: admin on `/member` | Redirected to `/` |
| Cross-role: member on `/` | Redirected to `/member` |

#### `users.spec.ts` (3 tests, admin auth)

| Test | Assertion |
|------|-----------|
| Members list | Seeded member email (`member-e2e@test.com`) appears in the list |
| Invite flow | Opens Add User dialog → fills email → Send invite → "Invite sent!" toast/message |
| Grant ad-hoc credits | Opens grant credits dialog → submits → dialog closes |

#### `member-home.spec.ts` (2 tests, member auth)

| Test | Assertion |
|------|-----------|
| Greeting + credits | "Welcome back" greeting, 8 credits shown, "Starter Pack" visible |
| Quick links | "Browse classes", "My reservations", "Message the gym" links present |

#### `member-classes.spec.ts` (2 tests, member auth)

| Test | Assertion |
|------|-----------|
| Join/unjoin a class | Joins a class on the classes page; same page; unjoins it |
| Empty reservations | Reservations page shows empty state |

#### `member-profile.spec.ts` (3 tests, member auth)

| Test | Assertion |
|------|-----------|
| Form pre-filled | Name, Surname, Phone, City fields contain seeded values |
| Edit name and save | Changes name, saves, "Saved" feedback appears |
| Password section | 3 password fields visible; "Email me a reset link" button + "Update password" button present |

### CI Workflow (`.github/workflows/e2e.yml`)

Triggers on every **pull request targeting `main`**. Job: `ubuntu-latest`, 15-minute timeout.

Steps:
1. Checkout
2. pnpm 10 + Node 22 setup (pnpm store cached)
3. Java 21 (Temurin) — required for Firebase emulator JARs
4. `pnpm install --frozen-lockfile`
5. Cache Playwright browsers (`~/.cache/ms-playwright`, keyed on `pnpm-lock.yaml`)
6. `pnpm exec playwright install chromium --with-deps`
7. Firebase CLI installed globally via `npm install -g firebase-tools`
8. Cache Firebase emulator JARs (`~/.cache/firebase/emulators`)
9. `pnpm test:e2e`
10. On failure: upload `playwright-report/` as artifact (7-day retention)

Note: the existing `claude.yml` workflow (Claude Code bot) is unchanged. This is a second, independent workflow file.

---

## Related pages

- [[reference/Configuration]] — jest.config.ts and playwright.config.ts details
- [[reference/Project Setup]] — running tests locally, emulator prerequisites
- [[reference/Environment Variables]] — emulator env vars
- [[auth/Firebase Setup]] — emulator hooks in admin.ts and client.ts
- [[auth/Auth API Routes]] — what each API route test covers
- [[auth/Route Protection]] — what the middleware test covers
