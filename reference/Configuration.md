---
title: Configuration
tags: [domain/reference, status/implemented]
status: implemented
sources: ["next.config.ts", "tsconfig.json", "jest.config.ts", "eslint.config.mjs", "postcss.config.mjs", "tailwind.config.ts", "pnpm-workspace.yaml", ".github/workflows/claude.yml", ".github/workflows/e2e.yml", "playwright.config.ts", "firebase.json", "scripts/test-e2e.sh", "__tests__/e2e/constants.ts"]
updated: 2026-06-11
---

> **Status:** ✅ Documented

# Configuration

## next.config.ts

Two customizations:

```ts
serverExternalPackages: ["firebase-admin"]
```
Marks `firebase-admin` as an external package so it's not bundled by webpack — required for the Admin SDK to access the native Node.js environment correctly.

```ts
distDir: process.env.NEXT_TEST ? '.next-test' : '.next'
```
When `NEXT_TEST=1` is set (exported by `scripts/test-e2e.sh` before running Playwright), the build output goes into `.next-test/` instead of `.next/`. This isolates the test dev server from a concurrently running normal dev session, preventing the two `next dev` processes from corrupting each other's shared build manifests.

## tsconfig.json

- `target: "ES2017"`, `strict: true`
- `module: "esnext"`, `moduleResolution: "bundler"`
- `jsx: "react-jsx"`, `noEmit: true`, `resolveJsonModule: true`
- Next.js TypeScript plugin: `"plugins": [{ "name": "next" }]`
- **Path alias:** `"@/*": ["./*"]` — enables `@/lib/...`, `@/components/...`, etc.
- Includes: `.ts`, `.tsx`, `.mts`, `.next/types`

## jest.config.ts

```ts
const nextJest = require('next/jest.js');
const createJestConfig = nextJest({ dir: './' });
module.exports = createJestConfig({
  setupFilesAfterFramework: ['<rootDir>/jest.setup.ts'],
  testEnvironment: 'jsdom',
  passWithNoTests: true,
  moduleNameMapper: { '^@/(.*)$': '<rootDir>/$1' },
});
```

`next/jest` handles: SWC transform, CSS module mocking, Next.js-specific module resolution. The `@/` mapper mirrors the tsconfig path alias.

## jest.setup.ts

```ts
import '@testing-library/jest-dom';
global.ResizeObserver = class ResizeObserver { observe() {} unobserve() {} disconnect() {} };
```

The `ResizeObserver` polyfill is needed because Recharts and the react-calendar library reference it in jsdom where it's not available.

## eslint.config.mjs

Flat config (ESLint v9). Composes `eslint-config-next/core-web-vitals` + `eslint-config-next/typescript`. Global ignores: `.next/`, `out/`, `build/`, `next-env.d.ts`.

Currently 0 errors, ~16 warnings (mostly `@next/next/no-img-element` for `<img>` usage and unused vars in some components).

## postcss.config.mjs

Single plugin: `@tailwindcss/postcss`. This is the Tailwind v4 PostCSS pipeline — different from the v3 `tailwindcss` plugin.

## tailwind.config.ts

Content globs: `./pages/**`, `./components/**`, `./app/**`. Theme extends: CSS-var `background`/`foreground` + custom `gym` palette (same hex values as `app/globals.css @theme`). No plugins. Gradient backgrounds defined: `gradient-radial`, `gradient-conic`.

**Note:** This is a v3-style config coexisting with the v4 `@theme` in `globals.css`. The v4 `@theme` takes precedence for the build pipeline.

## pnpm-workspace.yaml

Not a monorepo. Only declares allowed build scripts for native deps:
```yaml
allowBuilds:
  - sharp
  - unrs-resolver
```

## playwright.config.ts

- `testDir: './__tests__/e2e'`
- `fullyParallel: true`, Chromium-only (`channel: 'chrome'` — system Chrome)
- `globalSetup: './__tests__/e2e/global.setup.ts'`
- `baseURL: 'http://localhost:3001'` (port 3001, not 3000, to avoid colliding with a running dev session)
- `expect: { timeout: 10_000 }` — global assertion timeout of 10 seconds
- `reporter: 'html'`
- CI: `forbidOnly`, `retries: 2`, `workers: 1`
- `webServer`: `NEXT_TEST=1 pnpm dev --port 3001`; `reuseExistingServer: !process.env.CI`

## firebase.json

Declares the Firebase emulator configuration used by `scripts/test-e2e.sh` and CI:
- Auth emulator: port 9099
- Firestore emulator: port 8080
- Emulator UI disabled

## scripts/test-e2e.sh

Shell wrapper that sets all required env vars before invoking Playwright via `firebase emulators:exec`. Key exports:

- `NEXT_TEST=1` — causes `next.config.ts` to use `.next-test/` as `distDir`, isolating the test build
- `NEXT_PUBLIC_FIREBASE_AUTH_EMULATOR_HOST=127.0.0.1:9099` — read by `lib/firebase/client.ts` for the Auth emulator connection
- `NEXT_PUBLIC_FIRESTORE_EMULATOR_HOST=127.0.0.1:8080` — read by `lib/firebase/client.ts` for the Firestore emulator connection
- `FIREBASE_AUTH_EMULATOR_HOST`, `FIRESTORE_EMULATOR_HOST` — server-side (Admin SDK) emulator routing
- `FIREBASE_ADMIN_PROJECT_ID=demo-garnatafit`, `NEXT_PUBLIC_USE_EMULATORS=true`, and stub `NEXT_PUBLIC_FIREBASE_*` client vars so Next.js can start without real Firebase credentials

## GitHub Actions Workflows

Two CI workflow files exist under `.github/workflows/`:

### `claude.yml`

Runs the Claude Code bot on:
- `issue_comment` (created)
- `pull_request_review_comment` (created)
- `issues` (opened, assigned)
- `pull_request_review` (submitted)

Triggers only when the event body/title contains `@claude`. Job: `ubuntu-latest`, `anthropics/claude-code-action@v1`, authenticated via `CLAUDE_CODE_OAUTH_TOKEN` secret.

### `e2e.yml`

Runs e2e tests on every **pull request targeting `main`**. Job: `ubuntu-latest`, 15-minute timeout. Requires Java 21 (Temurin) for Firebase emulator JARs. Caches: pnpm store, Playwright browsers (`~/.cache/ms-playwright`), Firebase emulator JARs (`~/.cache/firebase/emulators`, keyed on `runner.os` + `package.json` hash). Runs `pnpm test:e2e`. On failure, uploads `playwright-report/` as a 7-day artifact.

Key corrections made during PR review:
- Playwright browser install uses `chrome` (not `chromium`) to match `channel: 'chrome'` in `playwright.config.ts`
- Firebase CLI is no longer installed via `npm install -g firebase-tools`; it is a `devDependency` and arrives via `pnpm install`

See [[reference/Testing]] for the full e2e setup details.

## Related pages

- [[overview/Tech Stack]] — library versions
- [[reference/Testing]] — jest.config.ts in test context
- [[reference/Environment Variables]] — env vars consumed at runtime
