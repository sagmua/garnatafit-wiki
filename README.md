# GarnataFit Wiki

A compiled, interlinked knowledge base about the **GarnataFit admin webapp** — maintained by an LLM, browsable by humans in Obsidian.

## What this is

This is an [LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f): a knowledge base where an AI agent reads the source code and synthesizes it into structured, cross-linked pages. It is not raw documentation scraped from the code — it is a curated understanding of the system, with every claim traceable back to a specific source file in the webapp repo at `/home/scardenete/Documents/garnatafit-webapp`.

## How to navigate

- **[[index]]** — full catalog of pages by category
- **[[overview/Project Overview]]** — start here if you're new to the project
- **[[auth/Authentication Overview]]** — deepest area; fully implemented
- **[[features/Dashboard (Home)]]** — dashboard pages (most backed by mock data)
- **[[ui/Component Library]]** — UI component inventory
- **[[reference/Conventions & Gotchas]]** — quick reference for known quirks

## Maturity conventions

Pages use a status badge on the first line:

| Badge | Meaning |
|-------|---------|
| ✅ **Implemented & tested** | Real code, API routes, tests passing |
| 🟡 **Mock data** | Feature UI exists but reads from `lib/data/mock_*.ts`; Firestore migration planned |
| 🔵 **Planned** | Designed but not yet built |

## For LLM agents

See **[[CLAUDE]]** for the operating manual: conventions, ingest/query/lint procedures, and the vault map.
