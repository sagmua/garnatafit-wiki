---
title: Settings
tags: [domain/features, status/mock]
status: mock
sources: ["app/(dashboard)/settings/page.tsx", "lib/data/mock_settings.ts"]
updated: 2026-06-11
---

> **Status:** 🟡 Profile tab is real; other tabs are mock/non-functional

# Settings

**Route:** `/settings`  
**File:** `app/(dashboard)/settings/page.tsx`  
Client component. Four-tab settings panel.

## Tabs

### Gym Info Tab — 🟡 Mock / non-functional

Three sections:
- **Branding:** Logo upload button (opens file picker, no upload handler). Gym name input (edits `gymSettings.name` in local state).
- **Contact Info:** Phone, email, address inputs (edits `gymSettings.contact` in local state).
- **Class Defaults:** Default capacity input.

All "Save changes" buttons are styled but have no `onClick` handler — **non-functional**. State lives only in React, not persisted. Data seeded from `lib/data/mock_settings.ts` (`gymSettings`).

### Profile Tab — ✅ Real

Uses `useAdmin()` context. See [[auth/Profile & Onboarding]] for full detail.

Key behaviors:
- `useEffect` rehydrates `name`/`surname` fields when async profile data loads.
- Avatar upload → client-side compression → `POST /api/auth/complete-profile`.
- `res.ok` check before showing "Saved!" — shows error message on failure.
- Calls `refresh()` after save to update the header avatar/name.
- Email field is disabled (read-only).
- Password section: a single "Send reset link" button calls `sendPasswordResetEmail` (Firebase client SDK) for the signed-in admin's email. Inline ok/error feedback. No form fields, no re-auth flow.

### Notifications Tab — 🟡 Mock / non-functional

Three toggle rows (new sign-ups, class bookings, member messages). Toggles update local `adminSettings` state. Not persisted. Seeded from `lib/data/mock_settings.ts` (`adminSettings.notifications`).

### Appearance Tab — 🟡 Mock / non-functional

Language selector (Español / English). Updates `adminSettings.appearance.language` in local state. Not persisted. "Save changes" button is non-functional.

## Shared UI Patterns

- `Toggle` — reusable accessible toggle switch (uses `role="switch"`, `aria-checked`). Defined inline in `settings/page.tsx`.
- `INPUT_CLASS`, `LABEL_CLASS`, `SAVE_BTN_CLASS` — shared CSS constant strings for consistent form styling.

## Real Data Path

When Firestore is wired up:
- Gym Info → `Firestore /gyms/{gymId}` (with admin-SDK write)
- Profile → already real (see above)
- Notifications/Appearance → `Firestore /admins/{uid}/settings`

## Related pages

- [[auth/Profile & Onboarding]] — Profile tab implementation
- [[data/Mock Data Layer]] — GymSettings and AdminSettings types
- [[reference/Conventions & Gotchas]] — non-functional buttons
