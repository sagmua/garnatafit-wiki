---
title: Configuration
tags: [domain/reference, status/implemented]
status: implemented
sources: ["next.config.ts", "tsconfig.json", "jest.config.ts", "eslint.config.mjs", "postcss.config.mjs", "tailwind.config.ts", "pnpm-workspace.yaml", ".github/workflows/claude.yml", "playwright.config.ts"]
updated: 2026-06-01
---

> **Status:** ✅ Documented

# Configuration

## next.config.ts

Minimal. Only one customization:
```ts
serverExternalPackages: ["firebase-admin"]
```
Marks `firebase-admin` as an external package so it's not bundled by webpack — required for the Admin SDK to access the native Node.js environment correctly.

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
- `fullyParallel: true`
- `reporter: 'html'`
- CI: `forbidOnly`, `retries: 2`, `workers: 1`
- Projects: chromium, firefox, webkit
- `webServer` block is commented out (requires manual server start)
- `baseURL: 'http://127.0.0.1:3000'` (commented out)

## GitHub Actions — .github/workflows/claude.yml

The only CI workflow. Runs the Claude Code bot on:
- `issue_comment` (created)
- `pull_request_review_comment` (created)
- `issues` (opened, assigned)
- `pull_request_review` (submitted)

Triggers only when the event body/title contains `@claude`. Job: `ubuntu-latest`, `anthropics/claude-code-action@v1`, authenticated via `CLAUDE_CODE_OAUTH_TOKEN` secret.

**No build/test/lint CI pipeline exists** — tests are run locally only.

## Related pages

- [[overview/Tech Stack]] — library versions
- [[reference/Testing]] — jest.config.ts in test context
- [[reference/Environment Variables]] — env vars consumed at runtime
