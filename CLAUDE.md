# GarnataFit Wiki — LLM Operating Manual (Schema)

This file defines the three-layer architecture, conventions, and operations for this wiki. Any LLM agent maintaining the wiki should read this file first.

---

## Three-Layer Architecture

| Layer | Location | Description |
|-------|----------|-------------|
| **Raw Source** | `/home/scardenete/Documents/garnatafit-webapp` | The webapp codebase. Read-only. The source of truth all wiki claims must trace back to. Never modify it from wiki-maintenance operations. |
| **The Wiki** | This vault (`/home/scardenete/Apps/obsidian/garnatafit-wiki`) | LLM-authored, interlinked markdown pages. You own this layer entirely. |
| **The Schema** | `CLAUDE.md` (this file) | Defines how the wiki is structured and how to maintain it. |

---

## Vault Map

```
garnatafit-wiki/
├── CLAUDE.md               ← this file
├── README.md               ← human-facing entry point
├── index.md                ← content catalog (all pages, one-line summaries)
├── log.md                  ← append-only activity log
│
├── overview/               ← project-wide context
│   ├── Project Overview.md
│   ├── Architecture.md
│   ├── Tech Stack.md
│   └── Domain Model.md
│
├── auth/                   ← fully implemented & tested (deepest pages)
│   ├── Authentication Overview.md
│   ├── Invite & Join Flow.md
│   ├── Login & Logout.md
│   ├── Profile & Onboarding.md
│   ├── Route Protection.md
│   ├── Auth API Routes.md
│   └── Firebase Setup.md
│
├── features/               ← mostly mock-data backed
│   ├── Dashboard (Home).md
│   ├── Users.md
│   ├── Classes.md
│   ├── Messages.md
│   ├── Analytics.md
│   └── Settings.md
│
├── ui/
│   ├── Component Library.md
│   ├── Styling & Theme.md
│   └── Layouts & Navigation.md
│
├── data/
│   └── Mock Data Layer.md
│
└── reference/
    ├── Project Setup.md
    ├── Testing.md
    ├── Configuration.md
    ├── Environment Variables.md
    └── Conventions & Gotchas.md
```

---

## Page Conventions

### Frontmatter (required on every topic page)

```yaml
---
title: <Page Title>
tags: [domain/<area>, status/<implemented|mock|planned>]
status: implemented | mock | planned
sources: ["relative/path/in/garnatafit-webapp"]
updated: YYYY-MM-DD
---
```

### Status badge (first body line after frontmatter)

```
> **Status:** ✅ Implemented & tested | 🟡 Mock data (Firestore migration planned) | 🔵 Planned
```

### Wikilinks

Use `[[Page Name]]` for all cross-references. Page Name = the exact file stem. Link liberally — orphaned links are acceptable and signal pages worth creating later.

### Source citations

Cite exact repo-relative paths in prose, e.g. `app/api/auth/invite/route.ts`. This traces claims back to the raw source layer. Do not quote large code blocks in the wiki; reference the file and line instead.

---

## Operations

### Ingest — when the codebase changes

1. Read the diff or changed files in the raw source.
2. Identify which wiki pages are affected (use `sources:` frontmatter to find them).
3. Update the affected pages: revise prose, update `sources:` and `updated:` fields.
4. If a feature transitions from `mock` to `implemented`, update `status:` and the badge.
5. Append an entry to `log.md`.

### Query — when answering a question about the project

1. Search wiki pages first. Read `index.md` for orientation.
2. Synthesize an answer with wikilinks and source file references.
3. If the answer is durable and reusable, promote it into a new wiki page or update an existing one.
4. Append a query entry to `log.md` if it produced lasting knowledge.

### Lint — periodic health-check

Run a lint pass to find:
- **Stale claims**: check that `sources:` paths still exist in the raw source repo.
- **Maturity drift**: pages marked `mock` whose source code is now real (check `lib/data/` references vs live routes).
- **Orphaned pages**: pages with no inbound `[[links]]` from other pages.
- **Missing entries**: pages not listed in `index.md`.
- **Contradictions**: conflicting claims across pages (e.g. color hex values, route paths).

After a lint pass, append an entry to `log.md`.

---

## Log Format

Append entries to `log.md` in this format:

```
## YYYY-MM-DD — <operation>
- <bullet summarizing what changed / what was found>
```
