---
title: Testing
tags: [domain/reference, status/implemented]
status: implemented
sources: ["jest.config.ts", "jest.setup.ts", "__tests__/middleware.test.ts", "app/api/auth/", "app/api/plans/", "app/api/members/", "app/api/member/", "lib/credits/", "lib/member/", "app/(member)/", "playwright.config.ts"]
updated: 2026-06-11
---

> **Status:** ✅ 115 Jest tests passing across 24 suites; Playwright configured (e2e excluded from Jest)

# Testing

## Test Stack

| Tool | Version | Purpose |
|------|---------|---------|
| Jest | `^30.4.2` | Test runner; configured via `next/jest` |
| React Testing Library | `^16.3.2` | Component rendering and DOM queries |
| `@testing-library/user-event` | `^14.6.1` | Simulated user interactions |
| `@testing-library/jest-dom` | `^6.9.1` | Custom matchers (`.toBeInTheDocument()`, etc.) |
| Playwright | `^1.60.0` | E2e tests (configured, `__tests__/e2e/`) |

## Configuration

**`jest.config.ts`:**
- Uses `next/jest.js` (configures Next.js module resolution, path aliases, SWC transforms)
- Default `testEnvironment: 'jsdom'` (for component tests)
- `setupFilesAfterEnv: ['./jest.setup.ts']`
- `moduleNameMapper`: `^@/(.*)$ → <rootDir>/$1` mirrors the tsconfig `@/*` path alias
- **Ignores `__tests__/e2e`** (Playwright specs) so they don't run under Jest

**`jest.setup.ts`:** Imports `@testing-library/jest-dom` matchers. Polyfills `global.ResizeObserver` (no-op) needed by Recharts and calendar components in jsdom.

## Test Environments

Two environments are used:

| Environment | Used for | How to set |
|-------------|---------|------------|
| `jsdom` (default) | Component/page tests | No annotation needed |
| `node` | API route tests and proxy test | `/** @jest-environment node */` at top of file |

Node environment is required for tests that use `Request`, `NextRequest`, and Firebase Admin SDK mocks.

## Test File Locations

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

## REQ-NN Labels

Tests are labeled with requirement IDs matching `docs/specs/auth-requirements.md`. This creates a traceable link between requirements and assertions. Example:

```ts
// REQ-01: no cookie on a protected route → redirect to /login
test('REQ-01: no session cookie on dashboard route redirects to /login', ...
```

## API Route Test Pattern

API route tests (node env):
1. Mock `@/lib/firebase/admin` with `jest.mock()`.
2. Construct a real `Request` or `NextRequest` object with `new Request(url, { method, headers, body })`.
3. Call the exported route handler directly: `await POST(req)`.
4. Assert `res.status` and `await res.json()`.
5. Assert mock calls: `expect(mockFn).toHaveBeenCalledWith(...)`.

**Cookie parsing in tests:** Tests that require session auth set `cookie: 'session=valid-session'` in the headers. The route parses this with `.substring('session='.length)`.

## Mocking Firebase

Firebase is always mocked in tests — never real SDK calls:

```ts
jest.mock('@/lib/firebase/admin', () => ({
  adminAuth: { verifySessionCookie: jest.fn(), /* ... */ },
  adminDb: { collection: jest.fn(() => ({ doc: jest.fn(() => ({ get: jest.fn(), set: jest.fn() })) })) },
}));
```

Then `jest.requireMock('@/lib/firebase/admin')` is used to access and configure mock return values per test.

## Playwright E2e

Config: `playwright.config.ts`. Test dir: `__tests__/e2e/`. Configured for chromium/firefox/webkit. `webServer` is commented out (manual start required). Currently contains only the Playwright-generated example spec. No pnpm script exists for running Playwright tests (not included in the `test` script).

## Related pages

- [[reference/Configuration]] — jest.config.ts details
- [[auth/Auth API Routes]] — what each API route test covers
- [[auth/Route Protection]] — what the middleware test covers
