---
title: Project Overview
tags: [domain/overview, status/implemented]
status: implemented
sources: ["CLAUDE.md", "README.md"]
updated: 2026-06-01
---

> **Status:** ✅ Project context documented

# GarnataFit — Project Overview

GarnataFit is a **cross-platform gym management ecosystem** with two applications targeting different user roles.

## Applications

| App | Repo | Platform | Status |
|-----|------|----------|--------|
| **Admin WebApp** | `garnatafit-webapp` | Next.js dashboard (this wiki) | Active development |
| **Mobile App** | Separate repo | React Native (iOS + Android) | Separate codebase |

The two apps share the same Firebase project (auth + Firestore) but are developed independently.

## User Roles

### Admins (gym staff — this webapp)
- User management: full CRUD on member profiles; approve new sign-ups before members access services
- Class scheduling: create and manage the class calendar
- Content management: upload and publish training routines/videos
- Communication: reply to member messages, send broadcast notifications to all members

### Members (gym members — mobile app)
- Book or cancel scheduled classes based on real-time availability
- View training content (video workouts) uploaded by admins
- Send direct messages to gym administration

## Scope of the Admin WebApp

The admin webapp at `garnatafit-webapp` is the control center for gym operations. Everything documented in this wiki is about this webapp only.

**What is in scope:** Auth system, dashboard pages, member management, class scheduling, messaging, analytics, settings.

**What is out of scope (for this wiki):** The React Native mobile app; Firebase Custom Claims enforcement; multi-tenant gym support (single tenant assumed); member-facing auth.

## Current State

The project has a fully implemented **authentication and admin onboarding system** backed by real Firebase + Firestore, and a set of **dashboard feature pages** that are UI-complete but backed by mock data pending a Firestore migration. See [[auth/Authentication Overview]] and [[data/Mock Data Layer]] for details.

## Related pages

- [[overview/Architecture]] — request flow, route groups, and how components fit together
- [[overview/Tech Stack]] — all libraries and versions
- [[overview/Domain Model]] — the entities the app manages (Gym, Admin, Member, Class, etc.)
- [[reference/Project Setup]] — how to run the app locally
