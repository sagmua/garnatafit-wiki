---
title: Project Overview
tags: [domain/overview, status/implemented]
status: implemented
sources: ["CLAUDE.md", "README.md", "docs/design/member-role-and-plans.md"]
updated: 2026-06-11
---

> **Status:** ✅ Project context documented

# GarnataFit — Project Overview

GarnataFit is a **cross-platform gym management ecosystem**. As of the `feature/member-role-and-plans` work, the webapp is **dual-audience** — it serves both gym **admins** (staff) and gym **members** — and this repo **owns the shared Firebase backend** that members use. A React Native mobile app is planned as a separate client over the same backend.

## Applications

| App | Repo | Platform | Status |
|-----|------|----------|--------|
| **WebApp (admin + member)** | `garnatafit-webapp` | Next.js (this wiki) | Active development |
| **Mobile App** | Separate repo | React Native (iOS + Android) | Planned; shares this backend |

> **Revised framing:** earlier docs treated members as *mobile-only* and the webapp as *admin-only*. That is no longer accurate — members can use the webapp, with a dedicated `/member/*` area gated by a Firebase Auth role claim. See [[auth/Roles & Claims]] and [[features/Member Area]].

## User Roles

### Admins (gym staff — this webapp)
- User management: full CRUD on member profiles; approve new sign-ups before members access services
- Class scheduling: create and manage the class calendar
- Content management: upload and publish training routines/videos
- Communication: reply to member messages, send broadcast notifications to all members

### Members (gym members — webapp `/member/*` area, and the future mobile app)
- Browse classes; reserve/cancel a spot (in-memory this phase — no persistence/credit deduction yet)
- See their own credit balance (granted by admins)
- Self-manage their profile (name, surname, phone, address) and password
- Message gym administration (mock this phase)

Roles are enforced via a Firebase Auth **custom claim** (`role: 'admin' | 'member'`). See [[auth/Roles & Claims]].

## Scope of the WebApp

The webapp at `garnatafit-webapp` is both the staff control center and the members' web client, and it owns the shared Firebase backend.

**In scope:** Auth + role system, admin dashboard pages, member management, **plans/credits**, the member area, messaging, analytics, settings.

**Out of scope / deferred:** The React Native mobile app; the booking/credit-**deduction** engine; the class-scheduling engine; real `firestore.rules` (intended model documented in `docs/design/member-role-and-plans.md`, authored in the mobile phase); payments/self-purchase; multi-tenant gym support (single tenant assumed).

## Current State

The project has a fully implemented **authentication, role, and onboarding system** (admins + members) backed by real Firebase + Firestore, a real **Plans/credits** feature, and a member area whose profile/credits are real but whose classes/reservations/chat are mock. The admin dashboard feature pages remain UI-complete but mostly mock pending Firestore. See [[auth/Authentication Overview]], [[auth/Roles & Claims]], [[features/Plans & Credits]], [[features/Member Area]], and [[data/Mock Data Layer]].

## Related pages

- [[overview/Architecture]] — request flow, route groups, and how components fit together
- [[overview/Tech Stack]] — all libraries and versions
- [[overview/Domain Model]] — the entities the app manages (Gym, Admin, Member, Class, etc.)
- [[reference/Project Setup]] — how to run the app locally
