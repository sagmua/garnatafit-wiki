---
title: Route Protection
tags: [domain/auth, status/implemented]
status: implemented
sources: ["proxy.ts", "__tests__/middleware.test.ts"]
updated: 2026-06-01
---

> **Status:** ✅ Implemented & tested (9 tests in `__tests__/middleware.test.ts`)

# Route Protection

Route protection is enforced by `proxy.ts` at the project root. This is the **Next.js 16** convention (`proxy.ts` replaces the deprecated `middleware.ts`). Next.js renames `proxy.js` to `middleware.js` internally during build.

## Matcher

```ts
export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico|api/).*)'],
};
```

**Excluded from the proxy:** static files (`_next/static`), optimized images (`_next/image`), favicon, and all API routes (`api/`). All page routes are matched.

## Decision Logic

The proxy reads two cookies on every matched request:

```ts
const session = req.cookies.get('session')?.value;
const profileComplete = req.cookies.get('profile_complete')?.value === '1';
```

Then applies this logic:

```
Is it a public page? (/login, /forgot-password, /invite/*)
├── No session → pass through (NextResponse.next())
└── Valid session → redirect to profileComplete ? '/' : '/welcome'

Is it /welcome?
├── No session → redirect to /login
├── Valid session + profileComplete → redirect to /
└── Valid session, not complete → pass through (onboarding)

Is it any other (dashboard) route?
├── No session → redirect to /login
├── Valid session, not profileComplete → redirect to /welcome
└── Valid session + profileComplete → pass through ✓

Cookie verification fails (verifySessionCookie throws):
→ treat as if no session (redirect to /login or pass through public page)
```

## Session Verification

`adminAuth.verifySessionCookie(session, true)` — the `true` argument forces Firebase to check if the session has been revoked. This is called on every request to a protected route, adding a small Firebase round-trip cost.

## Public Pages

```ts
const isPublicPage = pathname === '/login'
    || pathname === '/forgot-password'
    || pathname.startsWith('/invite/');
```

`/invite/complete` (the invite acceptance page) must be public — invitees are unauthenticated when they click the link.

## Test Coverage

`__tests__/middleware.test.ts` covers 9 scenarios (imports `{ proxy as middleware }` from `../proxy`):

- REQ-01: No session on dashboard → 307 to `/login`
- REQ-02: Valid session + `profile_complete` → passes through
- REQ-03: Expired/invalid session → 307 to `/login`
- REQ-04: Authenticated user visits `/login` (complete profile) → 307 to `/`
- Unauthenticated visits `/invite/complete` → passes through
- Unauthenticated visits `/forgot-password` → passes through
- Valid session without `profile_complete` → 307 to `/welcome`
- Valid session without `profile_complete` on `/welcome` → passes through
- Valid session with `profile_complete` on `/welcome` → 307 to `/`
- Authenticated (incomplete) visits `/login` → 307 to `/welcome`

## Related pages

- [[auth/Authentication Overview]] — the two-cookie model
- [[auth/Login & Logout]] — how the session cookie is set
- [[auth/Profile & Onboarding]] — how the profile_complete cookie is set
- [[reference/Testing]] — how the proxy is tested in Jest
