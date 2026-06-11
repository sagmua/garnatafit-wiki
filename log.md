# Activity Log

Append-only chronological record of all wiki operations (ingests, queries, lint passes).

---

## 2026-06-01 — Initial creation

- Created full vault structure from scratch based on codebase exploration of `garnatafit-webapp` at commit `34b01e9` (main) / `52e88f8` (feature/auth-foundation).
- Wrote 28 pages covering: project overview, architecture, tech stack, domain model, auth system (7 pages — deepest), all 6 dashboard features, UI components, styling, mock data layer, and reference docs.
- Auth pages marked `implemented`; all feature pages marked `mock` (backed by `lib/data/mock_*.ts` with inline Firestore migration notes).
- Sources: all files under `/home/scardenete/Documents/garnatafit-webapp` explored via codebase agents.

## 2026-06-11 — Ingest: feature/member-onboarding-fixes (commit 3fc24c3)

Three behaviour-changing fixes ingested from branch `feature/member-onboarding-fixes`.

- **`auth/Invite & Join Flow.md`** — Phase 1: noted that `lib/email/templates/invite.ts` now uses generic copy ("You've been invited to join GarnataFit"), removing the old admin-specific "dashboard" reference, making the template suitable for both roles. Phase 4: corrected `complete-profile` Firestore write to be role-aware (`members` vs `admins` collection); added `{ status: 'ok', role }` return shape; updated client redirect to `/member` for members and `/` for admins (was always `/`). Added `lib/email/templates/invite.ts` to `sources:`.
- **`auth/Auth API Routes.md`** — `POST /api/auth/complete-profile`: step 1 now captures `role` from the session cookie; step 5 documents role-aware collection selection; step 7 added (`Returns { status: 'ok', role }`); 200 status row updated to reflect `role` in response body. `POST /api/auth/invite`: `sendInviteEmail` note updated to reference generic email copy. Added `lib/email/templates/invite.ts` to `sources:`.
- **`auth/Profile & Onboarding.md`** — `/welcome` submit description updated: now redirects to `/member` for members, `/` for admins (reads `data.role`). `complete-profile` API step 5 corrected to role-aware collection; step 7 response updated to `{ status: 'ok', role }`.

## 2026-06-11 — Ingest: feature/member-role-and-plans

Ingested the dual-audience + roles + Plans feature set (commits 1128eb7, a7498e7, cedfdbd, 49ff07b, 2da0707, 9f52a21). Verified against code; design summary in `docs/design/member-role-and-plans.md`.

- **Created** `auth/Roles & Claims.md` — `role: 'admin'|'member'` custom claim, minted in `setup-profile` (defensive in `login`), profile collections `admins/{uid}` vs `members/{uid}`, `pendingInvites.role`, `getIdToken(true)` claim refresh, legacy-no-claim→admin back-compat, shared `lib/auth/session.ts` guard (`getSessionUser`, `requireAdmin`), extended `AdminProfile`/`/api/auth/me`.
- **Created** `features/Plans & Credits.md` — `creditBundleTemplates` CRUD (`/api/plans` + `/api/plans/[id]`), grants (`/api/members/[uid]/credits` → `creditBatches` + `creditsAvailable` increment), `lib/credits/{types,validate}.ts`, member read (`/api/member/credits`), `/plans` admin UI. Noted: no deduction engine yet.
- **Created** `features/Member Area.md` — `app/(member)/` route group + `MemberLayout`; real profile (`PATCH /api/member/profile`) + password; real credits; mock classes/reservations (in-memory `reservations-context`)/chat.
- **Updated** `overview/Project Overview.md` — dual-audience reframing (members no longer mobile-only; repo owns shared member backend).
- **Updated** `overview/Architecture.md` — added `(member)` route group, plans/members/member API routes, `lib/auth/session.ts`, role-aware request lifecycle.
- **Updated** `overview/Domain Model.md` — Member now real (`members/{uid}`); added BundleTemplate & CreditBatch; `pendingInvites.role`; in-memory Reservation; refreshed maturity table; status badge.
- **Updated** `auth/Route Protection.md` — role-gated proxy logic, `/member/*` confinement, claim-driven landing, legacy back-compat.
- **Updated** `auth/Authentication Overview.md`, `auth/Auth API Routes.md`, `auth/Invite & Join Flow.md`, `auth/Login & Logout.md`, `auth/Profile & Onboarding.md` — role/claim/collection details, invite `role` body, `me` extensions, member self-service profile.
- **Updated** `auth/Firebase Setup.md` — added members/creditBatches/creditBundleTemplates collections; documented that real `firestore.rules` are NOT authored yet (intended model recorded).
- **Updated** `features/Users.md` — flipped to `implemented`: real member list (`/api/members`), member+admin invite tabs, grant-credits dialog.
- **Updated** `ui/Layouts & Navigation.md` — MemberLayout, Sidebar `items` prop, new admin "Plans" nav entry, member nav.
- **Updated** `data/Mock Data Layer.md` — in-memory reservations context note.
- **Updated** `reference/Testing.md` — jest ignores `__tests__/e2e`; 115 tests / 24 suites; new suites; stable `useAdmin` mock note.
- **Updated** `index.md` — added the 3 new pages and refreshed feature statuses.

## 2026-06-11 — Query: invite link TTL

- Checked `app/api/auth/invite/route.ts` — `actionCodeSettings` passes no custom expiration; Firebase platform default of **24 hours** applies.
- Updated `auth/Invite & Join Flow.md` Phase 2 error-case note to state the concrete 24-hour TTL.

## 2026-06-11 — Ingest: feature/e2e-playwright

Added full Playwright e2e test infrastructure (17 tests, 5 spec files) running against Firebase emulators. Branch: `feature/e2e-playwright`.

- **Updated `reference/Testing.md`** — rewrote the Playwright section from "configured but empty" to a full e2e chapter: infrastructure (`playwright.config.ts`, `firebase.json`, `scripts/test-e2e.sh`), global setup seed logic (admin + member fixture users, credit batch, session cookie handshake), all 5 spec files with per-test assertion tables, and the CI workflow details. Updated `sources:` to include all new files. Status badge updated: 17 Playwright tests now passing alongside 115 Jest tests.
- **Updated `reference/Configuration.md`** — replaced the old stale playwright.config.ts entry (was Chromium/Firefox/WebKit, commented-out webServer); documented actual config (Chromium-only, port 3001, globalSetup). Added `firebase.json` and `scripts/test-e2e.sh` entries. Replaced "no CI pipeline" note with two-workflow description: `claude.yml` (unchanged) and new `e2e.yml` (PR-on-main trigger, Java 21, emulator caching). Added new files to `sources:`. Bumped `updated:` to 2026-06-11.
- **Updated `reference/Project Setup.md`** — added Firebase CLI / Java 21 / system Chrome to prerequisites. Added `pnpm test:e2e` to the commands table with a brief explanation. Added `scripts/test-e2e.sh` and `firebase.json` to `sources:`. Bumped `updated:` to 2026-06-11.
- **Updated `auth/Firebase Setup.md`** — documented emulator mode for Admin SDK (skips `cert()`, uses `initializeApp({ projectId })`), emulator mode for Client SDK (`connectAuthEmulator` + `connectFirestoreEmulator` guarded by window + try/catch), and Resend early-return under `FIREBASE_AUTH_EMULATOR_HOST`. Added `lib/email/resend.ts`, `firebase.json`, `scripts/test-e2e.sh` to `sources:`. Added cross-ref to Testing in Related pages. Bumped `updated:` to 2026-06-11.
- **Updated `reference/Environment Variables.md`** — added "E2e / Emulator" section documenting `FIREBASE_AUTH_EMULATOR_HOST`, `FIRESTORE_EMULATOR_HOST`, `FIREBASE_ADMIN_PROJECT_ID` (demo), and `NEXT_PUBLIC_USE_EMULATORS`, all set by `scripts/test-e2e.sh` at test-time only. Added `scripts/test-e2e.sh` and `playwright.config.ts` to `sources:`. Bumped `updated:` to 2026-06-11.
- **Updated `index.md`** — refreshed Testing summary line to reflect 115 unit + 17 e2e tests.

## 2026-06-11 — Ingest: feature/member-onboarding-fixes (commit 49dda2b)

Password-change UX simplified to email-link reset in both member and admin surfaces.

- **`features/Member Area.md`** — Profile screen password description updated: removed `reauthenticateWithCredential` + `updatePassword` two-step flow; now documents single "Send reset link" button (`sendPasswordResetEmail`). Maturity table row updated to "Password reset (email link) — Real (`sendPasswordResetEmail`)". Added `app/(member)/member/profile/page.test.tsx` to `sources:`.
- **`auth/Profile & Onboarding.md`** — Settings Profile tab password entry: removed "non-functional / no `onClick`" note and reference to [[reference/Conventions & Gotchas]]; now documents the reset-link button and inline feedback. Member self-service password entry: removed `reauthenticateWithCredential` + `updatePassword` description; now matches admin: single `sendPasswordResetEmail` button with no re-auth flow. `updated:` was already `2026-06-11`.
- **`features/Settings.md`** — Profile tab password bullet: removed `showPasswordForm` state, "Update password is non-functional", and Gotchas cross-ref; replaced with reset-link button description. `updated:` bumped to `2026-06-11`.
- **`reference/Conventions & Gotchas.md`** — Removed the "Update password button does nothing" entry (the button no longer exists — replaced by the working reset-link button). `updated:` bumped to `2026-06-11`.

## 2026-06-11 — Ingest: feature/e2e-playwright PR-review fixes (merged to main)

Post-review fixes on the e2e testing branch, merged to main after CI validation.

- **`reference/Testing.md`** — Added `__tests__/e2e/constants.ts` to `sources:`. Documented the new constants module (centralizes `ADMIN_STATE`/`MEMBER_STATE` paths and `ADMIN`/`MEMBER` credential objects; spec files import from here to avoid pulling `firebase-admin` into Playwright browser workers). Added `lib/firebase/client.ts`, `lib/email/resend.ts`, `next.config.ts` to `sources:`. Updated `global.setup.ts` notes: imports from `constants.ts`; `clearEmulators()` now throws with descriptive errors on failure (no silent swallow). Updated emulator hooks: `lib/firebase/client.ts` reads host/port from `NEXT_PUBLIC_FIREBASE_AUTH_EMULATOR_HOST` / `NEXT_PUBLIC_FIRESTORE_EMULATOR_HOST` env vars; `lib/email/resend.ts` uses lazy `new Resend()` inside `sendInviteEmail()` to prevent CI crash when `RESEND_API_KEY` is absent. Added `expect: { timeout: 10_000 }` to `playwright.config.ts` entry. Updated `scripts/test-e2e.sh` entry: now exports `NEXT_TEST=1`, `NEXT_PUBLIC_FIREBASE_AUTH_EMULATOR_HOST`, `NEXT_PUBLIC_FIRESTORE_EMULATOR_HOST`. Updated CI workflow steps: `playwright install chrome` (was `chromium`); Firebase CLI no longer globally installed (is a devDependency); emulator cache key uses `package.json` hash. Added test-isolation note for `member-profile.spec.ts` (edit mutates name to 'Updated'; `users.spec.ts` asserts on email for stability).
- **`reference/Configuration.md`** — Updated `next.config.ts` section: documented `distDir` conditional (`NEXT_TEST=1` → `.next-test/`). Updated `playwright.config.ts` section: `expect: { timeout: 10_000 }`, webServer command now `NEXT_TEST=1 pnpm dev --port 3001`. Updated `scripts/test-e2e.sh` section: documents all newly exported vars. Updated `e2e.yml` section: noted `chrome` vs `chromium` fix and Firebase CLI devDependency change; updated emulator cache key description. Added `__tests__/e2e/constants.ts` to `sources:`.
- **`reference/Environment Variables.md`** — Added `NEXT_PUBLIC_FIREBASE_AUTH_EMULATOR_HOST`, `NEXT_PUBLIC_FIRESTORE_EMULATOR_HOST`, and `NEXT_TEST` to the E2e/Emulator table. Added `next.config.ts` to `sources:`. Clarified that `FIREBASE_AUTH_EMULATOR_HOST` is Admin SDK only (not client SDK — client uses the `NEXT_PUBLIC_` variants).
- **`reference/Project Setup.md`** — Corrected e2e prerequisites: removed `npm install -g firebase-tools` instruction; Firebase CLI is now a `devDependency` installed by `pnpm install`.
