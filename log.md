# Activity Log

Append-only chronological record of all wiki operations (ingests, queries, lint passes).

---

## 2026-06-01 — Initial creation

- Created full vault structure from scratch based on codebase exploration of `garnatafit-webapp` at commit `34b01e9` (main) / `52e88f8` (feature/auth-foundation).
- Wrote 28 pages covering: project overview, architecture, tech stack, domain model, auth system (7 pages — deepest), all 6 dashboard features, UI components, styling, mock data layer, and reference docs.
- Auth pages marked `implemented`; all feature pages marked `mock` (backed by `lib/data/mock_*.ts` with inline Firestore migration notes).
- Sources: all files under `/home/scardenete/Documents/garnatafit-webapp` explored via codebase agents.

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
