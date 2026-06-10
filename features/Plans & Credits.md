---
title: Plans & Credits
tags: [domain/features, status/implemented]
status: implemented
sources: ["lib/credits/types.ts", "lib/credits/validate.ts", "app/api/plans/route.ts", "app/api/plans/[id]/route.ts", "app/api/members/[uid]/credits/route.ts", "app/api/member/credits/route.ts", "app/(dashboard)/plans/page.tsx", "app/(dashboard)/users/page.tsx", "docs/design/member-role-and-plans.md"]
updated: 2026-06-11
---

> **Status:** ✅ Implemented & tested (real Firestore via Admin SDK; no deduction engine yet)

# Plans & Credits

A **real, admin-only** feature for selling and granting bundles of class credits. All writes are server-side via the Firebase Admin SDK and gated by `requireAdmin` (see [[auth/Roles & Claims]]).

**Important scope limit:** there is **no credit-deduction / booking engine** this phase. Joining a class does not consume credit yet (member reservations are in-memory — see [[features/Member Area]]). Credits can only be *granted* and *displayed*.

## Two concepts

| Concept | Firestore | What it is |
|---------|-----------|------------|
| **Bundle template** | `creditBundleTemplates/{id}` | A gym-wide catalog entry (a sellable product). |
| **Credit batch (grant)** | `members/{uid}/creditBatches/{id}` | A snapshot of a template (or ad-hoc bundle) granted to one member. |

A denormalized counter `members/{uid}.creditsAvailable` is kept in sync via `FieldValue.increment`.

## Types — `lib/credits/types.ts`

```ts
type ValidityUnit = 'day' | 'month';
interface Validity { unit: ValidityUnit; value: number }

interface BundleTemplate {
  id: string; name: string; credits: number;
  validity: Validity; description?: string; active: boolean;
}

interface CreditBatch {
  id: string; amount: number; remaining: number;
  grantedAt: string; expiresAt: string;
  templateId: string | null; templateName: string | null;
  grantedBy: string;
}
```

Validation and expiry math live in `lib/credits/validate.ts`:
- `validateTemplateInput(body)` — name required, `credits` positive int, `validity` `{ unit ∈ {day,month}, value: positive int }`, optional description, `active` defaults `true`.
- `computeExpiry(from, validity)` — snapshots a concrete expiry `Date` from a grant time + validity window (`setDate`/`setMonth`).

## Bundle Template CRUD — admin only

| Method | Route | File | Result |
|--------|-------|------|--------|
| GET | `/api/plans` | `app/api/plans/route.ts` | List all templates |
| POST | `/api/plans` | `app/api/plans/route.ts` | Create → `201 { id }` (stores `createdBy`, `createdAt`) |
| PATCH | `/api/plans/[id]` | `app/api/plans/[id]/route.ts` | Partial edit / (de)activate |
| DELETE | `/api/plans/[id]` | `app/api/plans/[id]/route.ts` | Remove template |

All four call `requireAdmin` first → `401` unauthenticated, `403` non-admin. Invalid bodies → `400`. PATCH builds a validated partial update from only the allowed fields present.

### Admin UI — `/plans`

`app/(dashboard)/plans/page.tsx` is a new dashboard page (new **"Plans"** entry in the admin [[ui/Layouts & Navigation|Sidebar]], `Ticket` icon, after the existing nav items). It lists templates and offers a create/edit modal (`PlanModal`) plus activate/delete actions.

## Granting credits to a member — admin only

**Route:** `POST /api/members/[uid]/credits` — `app/api/members/[uid]/credits/route.ts`

Accepts **either**:
- `{ templateId }` — snapshots the template's `credits`/`validity`/`name` into a new batch (404 if the template is missing), **or**
- `{ credits, validity, label? }` — an ad-hoc batch (validated; `label` becomes `templateName`, `templateId` is null).

It writes `members/{uid}/creditBatches/{id}` with `amount`, `remaining` (= amount), `expiresAt` (from `computeExpiry`), `templateId`, `templateName`, `grantedBy`, `grantedAt`, then increments `members/{uid}.creditsAvailable`. Returns `201 { id }`.

Snapshotting means later edits to a template never alter credits already granted.

### Admin UI — grant dialog on the Users page

The [[features/Users]] page renders a `GrantCreditsModal` per member. It lets the admin pick a plan (template) from `GET /api/plans` or enter an ad-hoc credits/validity/label, then POSTs to the grant route. See [[features/Users]].

## Member-facing credit read

**Route:** `GET /api/member/credits` — `app/api/member/credits/route.ts`. Returns the **signed-in member's own** `creditBatches`. Used by the member home screen. Member-scoped via `getSessionUser` (any authenticated user reads their own batches). See [[features/Member Area]].

## Maturity

| Piece | Status |
|-------|--------|
| `creditBundleTemplates` CRUD | ✅ Real (Admin SDK) |
| Member grants + `creditsAvailable` counter | ✅ Real (Admin SDK) |
| Member credit read | ✅ Real |
| Credit **deduction** on booking | 🔵 Deferred (no engine yet) |
| Payments / self-purchase | 🔵 Deferred |
| Firestore security rules | 🔵 Deferred to mobile phase |

## Related pages

- [[auth/Roles & Claims]] — `requireAdmin` guard and the role model
- [[features/Users]] — member list + grant dialog
- [[features/Member Area]] — where members see their credits
- [[overview/Domain Model]] — BundleTemplate & CreditBatch entities
- [[auth/Auth API Routes]] — sibling API conventions
